# JETTO Vault – Hierarchical Folder Structure
## Best Practices & Technical Guide

**Version:** 1.0  
**Date:** March 2026  
**Scope:** Data model, API conventions, UX rules, and implementation guide for the customizable drawing vault folder system.

---

## 1. WHY THIS EXISTS

The original MVP spec used a flat `areas` table (e.g. "Strutturale", "Dettaglio", "Impianti") as a single-level tag on drawings. This doesn't match how architects actually think about — or organize — a real project.

A mid-size residential project might have:

```
Villa Mondello
├── Piano Terra
│   ├── Cucina
│   │   ├── Montature Mobili
│   │   └── Impianti Elettrici
│   ├── Bagno Principale
│   └── Soggiorno
├── Piano Primo
│   ├── Camera da Letto
│   └── Terrazza
├── Strutturale
└── Impianti Generali
```

Architects need to create this structure themselves, just like they would in Dropbox or a filesystem. This guide defines how that works technically and what rules the system enforces.

---

## 2. DATA MODEL DECISION: WHY ADJACENCY LIST + MATERIALIZED PATH

### Options Evaluated

| Pattern | Read Speed | Write Speed | Complexity | Notes |
|---|---|---|---|---|
| Adjacency List (`parent_id`) | Medium | Fast | Low | Simple; needs recursive CTEs for subtree queries |
| Nested Sets (left/right) | Fast | Very slow | High | Reindexes entire tree on every write; not suitable for frequent folder creation |
| Closure Table | Fast | Medium | Medium | Stores every ancestor-descendant pair; storage overhead grows O(N²) |
| **Materialized Path** (`/path/`) | Fast | Fast | Low | Natural filesystem semantics; fast `LIKE` prefix queries |
| **Hybrid (Adjacency + Path)** ✅ | Fast | Fast | Low–Medium | Best of both worlds for JETTO's use case |

### Why the Hybrid Wins for JETTO

1. **Supabase uses PostgreSQL** — which has native `WITH RECURSIVE` CTEs. This eliminates the main weakness of pure adjacency lists.
2. **Depth is shallow** — JETTO folders will rarely exceed 4 levels deep. No need for closure table overhead.
3. **Writes happen often** — Architects create and reorganize folders regularly. Nested sets would be unusable.
4. **Breadcrumbs are easy** — The `path` column lets you render the full breadcrumb trail without any DB queries.
5. **Subtree queries are a single `LIKE`** — `WHERE path LIKE '/proj-uuid/folder-uuid/%'` returns all descendants instantly.

This is the same pattern used by AWS S3 (key prefixes), Dropbox, and most modern cloud storage systems at their core.

---

## 3. DATABASE SCHEMA

### `folders` Table

```sql
CREATE TABLE folders (
  id          UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id  UUID        NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  parent_id   UUID        REFERENCES folders(id) ON DELETE CASCADE,
  -- NULL parent_id = root-level folder within the project

  name        VARCHAR(255) NOT NULL,
  path        TEXT        NOT NULL,
  -- Format: '/{project_id}/{folder_id}/' for root folders
  -- Format: '/{project_id}/{parent_id}/{folder_id}/' for nested folders
  -- Always starts and ends with '/'
  -- Example: '/abc123/def456/ghi789/'

  position    INT         NOT NULL DEFAULT 0,
  -- Manual sort order among siblings. Lower = higher in list.
  -- Allows drag-and-drop reordering without touching names/paths.

  created_by  UUID        NOT NULL REFERENCES users(id),
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  -- Prevent duplicate folder names within the same parent
  CONSTRAINT unique_name_per_parent UNIQUE (project_id, parent_id, name)
);

-- Indexes for performance
CREATE INDEX idx_folders_project    ON folders(project_id);
CREATE INDEX idx_folders_parent     ON folders(parent_id);
CREATE INDEX idx_folders_path_btree ON folders USING btree(path text_pattern_ops);
-- text_pattern_ops is required for LIKE prefix queries to use the index in PostgreSQL
```

### Updated `drawings` Table

Replace the old `area_id` column with `folder_id`:

```sql
-- Remove old flat area reference
ALTER TABLE drawings DROP COLUMN area_id;

-- Add hierarchical folder reference
ALTER TABLE drawings
  ADD COLUMN folder_id UUID REFERENCES folders(id) ON DELETE SET NULL;
  -- ON DELETE SET NULL: if folder is deleted, drawings float back to root, not deleted.

CREATE INDEX idx_drawings_folder ON drawings(folder_id);
```

### `areas` Table — DEPRECATED

The original `areas` table is replaced entirely by `folders`. Migrate any existing area records to root-level folders on deploy.

```sql
-- Migration: convert areas to root folders
INSERT INTO folders (project_id, parent_id, name, path, created_by)
SELECT
  project_id,
  NULL,
  name,
  CONCAT('/', project_id::text, '/', id::text, '/'),
  (SELECT id FROM users ORDER BY created_at ASC LIMIT 1) -- assign to first user as fallback
FROM areas;

-- Then update drawings to point to new folders
-- (match by project_id and name)
UPDATE drawings d
SET folder_id = f.id
FROM areas a
JOIN folders f ON f.name = a.name AND f.project_id = a.project_id
WHERE d.area_id = a.id;

-- Drop areas table after verification
DROP TABLE areas;
```

---

## 4. KEY QUERY PATTERNS

### 4.1 Get Root-Level Folders for a Project

```sql
SELECT * FROM folders
WHERE project_id = $1
  AND parent_id IS NULL
ORDER BY position ASC, name ASC;
```

### 4.2 Get Direct Children of a Folder

```sql
SELECT * FROM folders
WHERE parent_id = $1
ORDER BY position ASC, name ASC;
```

### 4.3 Get ALL Descendants of a Folder (Subtree)

Using materialized path — no recursion needed:

```sql
SELECT * FROM folders
WHERE path LIKE ($1 || '%')  -- $1 is the folder's own path, e.g. '/proj/folder/'
  AND id != $2               -- exclude the folder itself
ORDER BY path ASC;
```

Or using PostgreSQL recursive CTE (more explicit):

```sql
WITH RECURSIVE subtree AS (
  SELECT * FROM folders WHERE id = $1
  UNION ALL
  SELECT f.* FROM folders f
  JOIN subtree s ON f.parent_id = s.id
)
SELECT * FROM subtree WHERE id != $1;
```

### 4.4 Get Breadcrumb Trail (All Ancestors)

Using materialized path — parse the path string to get folder IDs, then fetch names:

```sql
-- Path example: '/proj-abc/folder-1/folder-2/folder-3/'
-- Extract ancestor IDs from path segments
SELECT id, name
FROM folders
WHERE ('/' || id::text || '/') LIKE ANY (
  SELECT '/' || unnest(string_to_array(trim(BOTH '/' FROM $1), '/')) || '/'
)
ORDER BY path ASC;
```

### 4.5 Get Drawing Count per Folder (Recursive)

```sql
SELECT f.id, f.name, COUNT(d.id) AS drawing_count
FROM folders f
LEFT JOIN drawings d ON d.folder_id = f.id
WHERE f.path LIKE ($1 || '%')  -- descendants of a given folder
GROUP BY f.id, f.name;
```

---

## 5. API CONVENTIONS

### Folder CRUD

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/projects/{id}/folders` | List root folders for project |
| `GET` | `/api/folders/{id}/children` | List direct children of a folder |
| `GET` | `/api/folders/{id}` | Get single folder (includes path for breadcrumb) |
| `POST` | `/api/folders` | Create new folder |
| `PATCH` | `/api/folders/{id}` | Rename or reorder folder |
| `PATCH` | `/api/folders/{id}/move` | Move folder to a new parent |
| `DELETE` | `/api/folders/{id}` | Delete folder (with rules — see below) |

### Create Folder – Request Body

```json
{
  "project_id": "uuid",
  "parent_id": "uuid | null",    // null = create at project root
  "name": "Cucina",
  "position": 0                  // optional; defaults to appending at end
}
```

**Server-side:** Compute `path` automatically on create — never accept it from the client.

```typescript
// Path computation logic
const parentPath = parent ? parent.path : '/';
const newPath = `${parentPath}${project_id}/${newFolderId}/`;
// For root: /${project_id}/${folder_id}/
// For nested: /${project_id}/${parent_id}/${folder_id}/
```

### Move Folder – Path Update

When a folder is moved, ALL its descendants' paths must be updated atomically:

```sql
-- Move folder $1 to new parent $2
-- Old path: /proj/old-parent/folder/
-- New path: /proj/new-parent/folder/

UPDATE folders
SET path = REPLACE(path, $old_path, $new_path)
WHERE path LIKE ($old_path || '%');
-- This updates the folder AND all descendants in a single query
```

### Delete Rules

| Scenario | Behavior |
|---|---|
| Delete empty folder | Hard delete — instant |
| Delete folder with subfolders | Block delete; require confirmation; recursively delete children |
| Delete folder with drawings | Drawings move to parent folder (or root if deleting a root folder) — never cascade-delete drawings |

---

## 6. DEPTH LIMITS & UX RULES

### Depth Limit: 4 Levels Maximum

```
Level 0 (root): Project vault root — no folder card, just content
Level 1: Master categories (Piano Terra, Piano Primo, Strutturale)
Level 2: Rooms / zones (Cucina, Bagno, Soggiorno)
Level 3: Disciplines (Impianti Elettrici, Pavimenti, Montature Mobili)
Level 4: Sub-disciplines (Dettaglio A-A, Sezione B-B)
```

Enforce at API level:
```typescript
const pathDepth = newPath.split('/').filter(Boolean).length;
// path '/proj/a/b/c/d/' splits to ['proj','a','b','c','d'] = depth 4 (after removing project_id)
if (pathDepth > 5) throw new Error('Maximum folder depth (4 levels) reached');
```

### Naming Rules

- Max 50 characters per folder name
- No leading/trailing spaces (trim on save)
- Allowed characters: letters, numbers, spaces, hyphens, parentheses, periods
- Duplicate names within same parent are blocked at DB level (`UNIQUE` constraint)
- Case-insensitive duplicate check recommended (e.g. "Cucina" and "cucina" should conflict)

### Display Conventions (Dropbox-Style)

1. **Folders always appear before drawings** within a view
2. **Folder cards** show: name, subfolder count, drawing count (direct + recursive), status summary badge
3. **Drawings** appear after all folders, as the existing list view
4. **Empty folders** are displayed normally (not hidden) — architects organize ahead of time
5. **Breadcrumb trail** is always visible at top of vault view
6. **Create Folder** button available at every level (visible to Architects and PMs only)
7. **Folder context menu** (long-press on mobile, right-click on desktop): Rename, Move, Delete

### Icon System

| Context | Icon | Meaning |
|---|---|---|
| Folder (default) | 📁 | Standard folder |
| Folder (open / active) | 📂 | Currently viewing this folder |
| Folder (has alerts) | 🔴📁 | Contains drawings needing attention |
| Drawing (PDF) | 📄 | PDF file |
| Drawing (DWG) | 📐 | AutoCAD file |
| Drawing (image) | 🖼️ | JPG/PNG image |

---

## 7. TYPICAL PROJECT FOLDER TEMPLATES

Architects can start from scratch or choose a template when creating a project. Recommended templates:

### Template: Residential Villa

```
📁 Architettonico
  📁 Piante
  📁 Prospetti
  📁 Sezioni
📁 Strutturale
  📁 Fondazioni
  📁 Solai
  📁 Pilastri
📁 Impianti
  📁 Elettrico
  📁 Idraulico
  📁 Riscaldamento
📁 Particolari Costruttivi
📁 Arredi & Finiture
```

### Template: Commercial Renovation

```
📁 Stato di Fatto
📁 Progetto
  📁 Planimetrie
  📁 Alzati
  📁 Dettagli
📁 Strutture
📁 Impianti
📁 Documentazione
  📁 Autorizzazioni
  📁 Collaudi
```

### Template: Infrastructure / External Works

```
📁 Planimetria Generale
📁 Sezioni Trasversali
📁 Sezioni Longitudinali
📁 Dettagli Stradali
📁 Impianti Esterni
📁 Paesaggio & Verde
```

---

## 8. PERMISSIONS

Folder operations follow the same role matrix as drawings:

| Action | Architect | PM | Builder | Owner |
|---|---|---|---|---|
| Create folder | ✅ | ✅ | ❌ | ❌ |
| Rename folder | ✅ (own) | ✅ (any) | ❌ | ❌ |
| Move folder | ✅ (own) | ✅ (any) | ❌ | ❌ |
| Delete empty folder | ✅ (own) | ✅ (any) | ❌ | ❌ |
| Delete folder with content | ❌ | ✅ (any) | ❌ | ❌ |
| View all folders | ✅ | ✅ | ✅ | ✅ |

---

## 9. STORAGE & PERFORMANCE NOTES

- **Expected scale:** Each project: 3–8 root folders, 2–4 sublevels, ~50–200 total folders max
- **Performance:** At this scale, adjacency list + materialized path is instant. No optimization needed at MVP. Indexes are sufficient.
- **Storage:** A `folders` row is ~200 bytes. 200 folders = ~40KB per project. Negligible.
- **Path string length:** Max path length at 4 levels deep with UUID keys = `/(36)/(36)/(36)/(36)/` ≈ 155 characters. Well within `TEXT` limits.

---

## 10. WHAT THIS IS NOT

- **Not a filesystem.** Folders are organizational containers, not actual OS directories. Files are stored flat in object storage (Supabase/S3); the folder hierarchy is purely a metadata layer.
- **Not a permission boundary.** Folder permissions follow the project role matrix — there is no per-folder access control in MVP. (Post-MVP: per-folder visibility for multi-phase projects.)
- **Not a version container.** Versions live on individual drawings, not folders.

---

*End of VAULT_FOLDER_STRUCTURE.md*  
*This document should be shared with the contract developer alongside MVP_SPEC.md.*
