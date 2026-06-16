# Use Cases — Phase 1: Collections Module

---

## UC-COL-001 — Create a New Collection

**Actor:** Admin, Registrar  
**Phase:** 1 — Core Framework  
**Module:** Collections

### Preconditions
- User is logged in with Admin or Registrar role.
- No collection named "Photograph Collection" exists.

### Test Data
| Field            | Value |
| :--------------- | :---- |
| name             | Photograph Collection |
| description      | Historical photographs of Millbrook Township, 1880–1960 |
| accession_policy | Photographs relating to Millbrook Township and its residents only |
| finding_aid      | (leave blank) |

### Steps
1. Open the Collections module.
2. Click **New Collection**.
3. Enter name, description, and accession_policy from the test data above.
4. Leave finding_aid blank.
5. Click **Save**.

### Expected Outcome
- One new row in `collections`:
  - `name = 'Photograph Collection'`
  - `description` as entered
  - `accession_policy` as entered
  - `finding_aid IS NULL`
  - `created_at` and `updated_at` set to approximately now
- Collection appears in the collections list.
- One new row in `audit_log`:
  - `action = 'CREATE'`, `table_name = 'collections'`
  - `user_id` matches the logged-in user

### Schema Coverage
- `collections`: INSERT
- `audit_log`: INSERT

---

## UC-COL-002 — Edit a Collection's Name and Description

**Actor:** Admin, Registrar  
**Phase:** 1 — Core Framework  
**Module:** Collections

### Preconditions
- "Photograph Collection" exists (UC-COL-001 completed).

### Test Data
| Field       | Old Value               | New Value |
| :---------- | :---------------------- | :-------- |
| name        | Photograph Collection   | Historic Photograph Collection |
| description | Historical photographs… | Documentary photographs of Millbrook Township and its residents, 1880–1960 |

### Steps
1. Open the Collections module.
2. Select "Photograph Collection".
3. Edit name and description to the new values above.
4. Click **Save**.

### Expected Outcome
- The `collections` row is updated:
  - `name = 'Historic Photograph Collection'`
  - `description` updated
  - `updated_at` refreshed; `created_at` unchanged
- One new row in `audit_log`:
  - `action = 'UPDATE'`
  - `changed_fields` contains old and new values for `name` and `description`

### Schema Coverage
- `collections`: UPDATE
- `audit_log`: INSERT

---

## UC-COL-003 — View All Collections

**Actor:** Any authenticated user  
**Phase:** 1 — Core Framework  
**Module:** Collections

### Preconditions
- At least two collections exist.

### Steps
1. Open the Collections module.
2. Observe the collections list.

### Expected Outcome
- All rows from `collections` are displayed with name and description.
- List is sorted alphabetically by name (or by `created_at` — confirm sort order in UI spec).
- No audit_log entry (read-only operation).

### Schema Coverage
- `collections`: SELECT

---

## UC-COL-004 — [NEGATIVE] Attempt to Delete a Collection That Contains Objects

**Actor:** Admin  
**Phase:** 1 — Core Framework  
**Module:** Collections

### Preconditions
- "Historic Photograph Collection" exists and has at least one object assigned to it.

### Steps
1. Open the Collections module.
2. Select "Historic Photograph Collection".
3. Click **Delete Collection** and confirm.

### Expected Outcome
- Operation is refused.
- The database `ON DELETE RESTRICT` on `objects.collection_id` blocks the DELETE.
- The application surfaces an error message: e.g. "Cannot delete a collection that contains objects. Reassign or remove all objects first."
- No rows are deleted from `collections` or `objects`.
- No audit_log entry for a failed operation.

### Schema Coverage
- `collections`: DELETE (blocked by RESTRICT from `objects.collection_id`)
- `objects`: unchanged

---

## UC-COL-005 — Delete an Empty Collection

**Actor:** Admin  
**Phase:** 1 — Core Framework  
**Module:** Collections

### Preconditions
- A collection named "Test Collection (empty)" exists with no objects assigned to it.

### Steps
1. Open the Collections module.
2. Select "Test Collection (empty)".
3. Click **Delete Collection** and confirm.

### Expected Outcome
- The row is removed from `collections`.
- Collection no longer appears in the list.
- One new row in `audit_log`:
  - `action = 'DELETE'`, `table_name = 'collections'`
  - `changed_fields` contains a snapshot of the deleted row

### Schema Coverage
- `collections`: DELETE
- `audit_log`: INSERT
