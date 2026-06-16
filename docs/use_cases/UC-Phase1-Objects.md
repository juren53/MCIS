# Use Cases — Phase 1: Objects Module

---

## UC-OBJ-001 — Accession a New Object

**Actor:** Registrar  
**Phase:** 1 — Core Framework  
**Module:** Objects

### Preconditions
- "Historic Photograph Collection" exists.
- No object with accession number "2024.001.002" exists.

### Test Data
| Field            | Value |
| :--------------- | :---- |
| collection       | Historic Photograph Collection |
| accession_number | 2024.001.002 |
| title            | Millbrook Post Office, Exterior View |
| description      | Front facade of the original post office building on Elm Street |
| date_made        | 1923 |
| accession_date   | 2024-03-15 |
| medium           | Silver gelatin print |
| dimensions       | 5 × 7 in. |
| condition        | Good |

### Steps
1. Open the Objects module.
2. Click **New Object**.
3. Select collection "Historic Photograph Collection".
4. Enter all test data above; leave provenance, credit_line, and rights_statement blank.
5. Click **Save**.

### Expected Outcome
- One new row in `objects`:
  - `collection_id` matches "Historic Photograph Collection"
  - `accession_number = '2024.001.002'`
  - `title`, `description`, `date_made`, `medium`, `dimensions` as entered
  - `accession_date = '2024-03-15'`
  - `condition = 'Good'`
  - `location_id IS NULL`, `ia_published = FALSE`, `ia_identifier IS NULL`
  - `created_at` and `updated_at` set to approximately now
- Object appears in the objects list.
- One new row in `audit_log`: `action = 'CREATE'`, `table_name = 'objects'`

### Schema Coverage
- `objects`: INSERT
- `audit_log`: INSERT

---

## UC-OBJ-002 — Accession an Object with a Fuzzy Date

**Actor:** Registrar  
**Phase:** 1 — Core Framework  
**Module:** Objects

### Preconditions
- "Historic Photograph Collection" exists.

### Test Data
| Field            | Value |
| :--------------- | :---- |
| accession_number | 2024.001.003 |
| title            | Main Street Looking North |
| date_made        | circa 1910 |
| accession_date   | 2024-03-15 |
| medium           | Silver gelatin print |
| condition        | Good |

### Steps
1. Click **New Object** and select the collection.
2. Enter accession_number, title, date_made ("circa 1910"), accession_date, medium, and condition.
3. Click **Save**.

### Expected Outcome
- `date_made = 'circa 1910'` stored as-is — no parsing, no rejection.
- `accession_date = '2024-03-15'` stored as a strict DATE.
- Both fields coexist on the same row without conflict.

### Schema Coverage
- `objects`: INSERT (`date_made VARCHAR`, `accession_date DATE`)

---

## UC-OBJ-003 — Edit an Existing Object Record

**Actor:** Registrar, Staff  
**Phase:** 1 — Core Framework  
**Module:** Objects

### Preconditions
- Object "2024.001.002 — Millbrook Post Office, Exterior View" exists.

### Test Data
| Field       | Old Value   | New Value |
| :---------- | :---------- | :-------- |
| title       | Millbrook Post Office, Exterior View | Millbrook Post Office — Elm Street, Exterior |
| dimensions  | 5 × 7 in.   | 5 × 7 in. (127 × 178 mm) |

### Steps
1. Open object "2024.001.002".
2. Edit title and dimensions to the new values.
3. Click **Save**.

### Expected Outcome
- `title` and `dimensions` updated in `objects`.
- `updated_at` refreshed; `created_at` unchanged.
- One new row in `audit_log`:
  - `action = 'UPDATE'`
  - `changed_fields` contains old and new values for `title` and `dimensions` only

### Schema Coverage
- `objects`: UPDATE
- `audit_log`: INSERT

---

## UC-OBJ-004 — Record a Condition Assessment

**Actor:** Registrar, Staff  
**Phase:** 1 — Core Framework  
**Module:** Objects

### Preconditions
- Object "2024.001.002" exists with `condition = 'Good'`.

### Test Data
| Field           | Value |
| :-------------- | :---- |
| condition       | Fair |
| condition_notes | Minor foxing on lower right corner; no structural damage. Assessed 2024-09-10 by D. Chen. |

### Steps
1. Open object "2024.001.002".
2. Change condition from "Good" to "Fair".
3. Enter condition_notes.
4. Click **Save**.

### Expected Outcome
- `condition = 'Fair'`, `condition_notes` updated.
- `audit_log` records the condition change with old value `'Good'` and new value `'Fair'`.

### Schema Coverage
- `objects`: UPDATE (`condition`, `condition_notes`)
- `audit_log`: INSERT

---

## UC-OBJ-005 — Assign Credit Line and Rights Statement

**Actor:** Registrar  
**Phase:** 1 — Core Framework  
**Module:** Objects

### Preconditions
- Object "2024.001.002" exists; `credit_line` and `rights_statement` are both NULL.

### Test Data
| Field            | Value |
| :--------------- | :---- |
| credit_line      | Gift of Eleanor Pruitt, 2024 |
| rights_statement | No known copyright restrictions. |

### Steps
1. Open object "2024.001.002".
2. Enter credit_line and rights_statement.
3. Click **Save**.

### Expected Outcome
- `credit_line` and `rights_statement` updated.
- Object is now eligible for Internet Archive publication (rights_statement populated).

### Schema Coverage
- `objects`: UPDATE (`credit_line`, `rights_statement`)
- `audit_log`: INSERT

---

## UC-OBJ-006 — Search for an Object by Accession Number

**Actor:** Any authenticated user  
**Phase:** 1 — Core Framework  
**Module:** Objects

### Preconditions
- Object "2024.001.002" exists.

### Steps
1. In the Objects module search bar, enter "2024.001.002".
2. Execute the search.

### Expected Outcome
- Exactly one result returned: "Millbrook Post Office — Elm Street, Exterior".
- Search uses the unique index on `accession_number` (fast lookup).
- No audit_log entry.

### Schema Coverage
- `objects`: SELECT on `accession_number`

---

## UC-OBJ-007 — Search for Objects by Title Keyword

**Actor:** Any authenticated user  
**Phase:** 1 — Core Framework  
**Module:** Objects

### Preconditions
- At least two objects exist with "Millbrook" in the title.

### Steps
1. In the Objects module, enter "millbrook" in the title search field.
2. Execute the search (case-insensitive).

### Expected Outcome
- All matching objects returned.
- Results are case-insensitive (ILIKE or equivalent).
- No audit_log entry.

### Schema Coverage
- `objects`: SELECT on `title`

---

## UC-OBJ-008 — [NEGATIVE] Duplicate Accession Number

**Actor:** Registrar  
**Phase:** 1 — Core Framework  
**Module:** Objects

### Preconditions
- Object with accession_number "2024.001.002" already exists.

### Steps
1. Click **New Object**.
2. Enter accession_number "2024.001.002" with any other valid data.
3. Click **Save**.

### Expected Outcome
- INSERT is rejected by the UNIQUE constraint on `objects.accession_number`.
- Application displays: *"An object with accession number 2024.001.002 already exists."*
- No row inserted; no audit_log entry.

### Schema Coverage
- `objects`: INSERT (blocked by UNIQUE on `accession_number`)

---

## UC-OBJ-009 — Flag an Object for IA Publication

**Actor:** Registrar  
**Phase:** 1 — Core Framework  
**Module:** Objects

### Preconditions
- Object "2024.001.002" exists with `rights_statement` populated and `ia_published = FALSE`.

### Steps
1. Open object "2024.001.002".
2. Click **Queue for IA Publication** (or equivalent flag control).
3. Confirm.

### Expected Outcome
- `ia_published` flag set in preparation for Phase 3 publish queue.
- (Actual IA upload is a Phase 3 feature — this use case only tests the flag.)
- `audit_log` records the change to `ia_published`.

### Schema Coverage
- `objects`: UPDATE (`ia_published`)
- `audit_log`: INSERT
