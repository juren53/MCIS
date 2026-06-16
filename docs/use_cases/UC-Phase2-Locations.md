# Use Cases — Phase 2: Locations Module

---

## UC-LOC-001 — Create a Room Location

**Actor:** Admin, Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Locations

### Preconditions
- Phase 2 schema migration has run (`locations` table exists; `objects.location_id` FK constraint is active).
- No location named "Main Gallery" exists.

### Test Data
| Field          | Value |
| :------------- | :---- |
| name           | Main Gallery |
| location_code  | MGAL |
| location_type  | Room |
| parent         | (none — top-level) |
| description    | Primary public exhibition space |

### Steps
1. Open the Locations module.
2. Click **New Location**.
3. Enter name, location_code, type "Room", no parent.
4. Click **Save**.

### Expected Outcome
- One new row in `locations`:
  - `name = 'Main Gallery'`, `location_code = 'MGAL'`
  - `location_type = 'Room'`
  - `parent_location_id IS NULL`
  - `is_active = TRUE`
- `audit_log`: `action = 'CREATE'`, `table_name = 'locations'`

### Schema Coverage
- `locations`: INSERT
- `audit_log`: INSERT

---

## UC-LOC-002 — Create a Case Inside a Room (Hierarchy)

**Actor:** Admin, Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Locations

### Preconditions
- "Main Gallery" (MGAL) exists as a Room location (UC-LOC-001 completed).

### Test Data
| Field          | Value |
| :------------- | :---- |
| name           | Display Case A |
| location_code  | MGAL-CA |
| location_type  | Case |
| parent         | Main Gallery |

### Steps
1. Click **New Location**.
2. Enter name "Display Case A", code "MGAL-CA", type "Case".
3. Select "Main Gallery" as parent.
4. Click **Save**.

### Expected Outcome
- New row in `locations` with `parent_location_id` = location_id of "Main Gallery".
- Hierarchy is navigable: Main Gallery → Display Case A.
- `audit_log`: `action = 'CREATE'`

### Schema Coverage
- `locations`: INSERT (self-referential FK via `parent_location_id`)
- `audit_log`: INSERT

---

## UC-LOC-003 — Assign an Object to a Location

**Actor:** Registrar, Staff  
**Phase:** 2 — Collections Modules  
**Module:** Locations

### Preconditions
- Object "2024.001.002" exists with `location_id IS NULL`.
- "Display Case A" (MGAL-CA) exists.

### Steps
1. Open object "2024.001.002".
2. Click **Assign Location**.
3. Select "Display Case A" from the location picker.
4. Save.

### Expected Outcome
- `objects.location_id` updated to the `location_id` of "Display Case A".
- The FK constraint added in the Phase 2 migration is now enforced.
- `audit_log`: `action = 'UPDATE'`, `changed_fields` records old (`NULL`) and new location_id.

### Schema Coverage
- `objects`: UPDATE (`location_id`)
- `audit_log`: INSERT

---

## UC-LOC-004 — Move an Object to a Different Location

**Actor:** Registrar, Staff  
**Phase:** 2 — Collections Modules  
**Module:** Locations

### Preconditions
- Object "2024.001.002" is assigned to "Display Case A".
- A second location "Archive Shelf 3" (location_code: ARCH-S3, type: Storage) exists.

### Steps
1. Open object "2024.001.002".
2. Change the assigned location from "Display Case A" to "Archive Shelf 3".
3. Save.

### Expected Outcome
- `objects.location_id` updated to "Archive Shelf 3".
- Previous location assignment is not retained in Phase 2 — only the current location is stored. (Full location history is a future enhancement.)
- `audit_log` records the location change with old and new `location_id` values.

### Schema Coverage
- `objects`: UPDATE (`location_id`)
- `audit_log`: INSERT

---

## UC-LOC-005 — Retire a Location

**Actor:** Admin  
**Phase:** 2 — Collections Modules  
**Module:** Locations

### Preconditions
- "Display Case A" (MGAL-CA) exists and is active.
- No objects are currently assigned to it (they have been moved).

### Steps
1. Open "Display Case A" in the Locations module.
2. Uncheck **Active** and save.

### Expected Outcome
- `locations.is_active = FALSE` for "Display Case A".
- The location no longer appears in active location pickers when assigning objects.
- Historical records that previously referenced this location remain intact.
- `audit_log`: `action = 'UPDATE'`

### Schema Coverage
- `locations`: UPDATE (`is_active`)
- `audit_log`: INSERT

---

## UC-LOC-006 — [NEGATIVE] Delete a Parent Location with Children

**Actor:** Admin  
**Phase:** 2 — Collections Modules  
**Module:** Locations

### Preconditions
- "Main Gallery" has at least one child location ("Display Case A").

### Steps
1. Open "Main Gallery" in the Locations module.
2. Click **Delete Location** and confirm.

### Expected Outcome
- DELETE is blocked by `ON DELETE RESTRICT` on `locations.parent_location_id`.
- Application displays: *"Cannot delete a location that contains sub-locations. Delete or reassign child locations first."*
- No rows deleted.

### Schema Coverage
- `locations`: DELETE (blocked by self-referential RESTRICT)

---

## UC-LOC-007 — Create an Off-Site Location for a Loaned Object

**Actor:** Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Locations

### Preconditions
- An outgoing loan to "Valley College Art Gallery" is being prepared.

### Test Data
| Field          | Value |
| :------------- | :---- |
| name           | Valley College Art Gallery |
| location_code  | OFFSITE-VCAG |
| location_type  | Offsite |
| description    | Borrower location for 2024 loan agreement LN-2024-001 |

### Steps
1. Click **New Location**.
2. Enter name, code, type "Offsite", description.
3. Save.
4. Assign the loaned objects to this location via UC-LOC-003.

### Expected Outcome
- New row in `locations` with `location_type = 'Offsite'`.
- Objects on loan can be assigned this location so the system always knows where each object is.
- Loan counterparty contact details remain in the `loans` table; this record holds only the location reference.

### Schema Coverage
- `locations`: INSERT
- `objects`: UPDATE (`location_id`) — covered in UC-LOC-003
