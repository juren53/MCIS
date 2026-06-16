# Use Cases — Phase 1: Users Module

---

## UC-USR-001 — Create an Admin User

**Actor:** Admin  
**Phase:** 1 — Core Framework  
**Module:** Users

### Preconditions
- No user with username "moates" or email "m.oates@millbrookhs.org" exists.

### Test Data
| Field      | Value |
| :--------- | :---- |
| username   | moates |
| full_name  | Margaret Oates |
| email      | m.oates@millbrookhs.org |
| password   | (strong password, min 12 chars) |
| role       | Admin |

### Steps
1. Open the Users module (Admin only).
2. Click **New User**.
3. Enter all test data above.
4. Click **Save**.

### Expected Outcome
- One new row in `users`:
  - `username = 'moates'`, `full_name = 'Margaret Oates'`
  - `role = 'Admin'`
  - `hashed_password` is a bcrypt hash — plaintext is not stored
  - `is_active = TRUE`
  - `last_login IS NULL`
- `audit_log`: `action = 'CREATE'`, `table_name = 'users'`
- Password hash must NOT appear in `audit_log.changed_fields`.

### Schema Coverage
- `users`: INSERT
- `audit_log`: INSERT

---

## UC-USR-002 — Create a Volunteer User

**Actor:** Admin  
**Phase:** 1 — Core Framework  
**Module:** Users

### Preconditions
- No user with username "tbright" exists.

### Test Data
| Field     | Value |
| :-------- | :---- |
| username  | tbright |
| full_name | Tom Bright |
| email     | t.bright@millbrookhs.org |
| role      | Volunteer |

### Steps
1. Click **New User**, enter test data, set role to "Volunteer".
2. Click **Save**.

### Expected Outcome
- New row in `users` with `role = 'Volunteer'`, `is_active = TRUE`.
- Tom Bright can log in and create object records but cannot edit or delete.

### Schema Coverage
- `users`: INSERT

---

## UC-USR-003 — Deactivate a User

**Actor:** Admin  
**Phase:** 1 — Core Framework  
**Module:** Users

### Preconditions
- User "tbright" (Tom Bright) exists and is active.

### Steps
1. Open the Users module.
2. Select "Tom Bright".
3. Uncheck **Active** (or click **Deactivate**).
4. Save.

### Expected Outcome
- `users.is_active = FALSE` for tbright.
- The row is NOT deleted — audit trail is preserved.
- Any `audit_log` rows referencing tbright's `user_id` remain intact and meaningful.
- `audit_log`: `action = 'UPDATE'` on `users`, recording the `is_active` change.

### Schema Coverage
- `users`: UPDATE (`is_active`)
- `audit_log`: INSERT

---

## UC-USR-004 — [NEGATIVE] Login with a Deactivated Account

**Actor:** —  
**Phase:** 1 — Core Framework  
**Module:** Users

### Preconditions
- User "tbright" has `is_active = FALSE` (UC-USR-003 completed).

### Steps
1. At the login screen, enter username "tbright" and the correct password.
2. Click **Login**.

### Expected Outcome
- Login is refused.
- Application displays: *"This account has been deactivated. Contact your administrator."*
- `last_login` is NOT updated.
- No session is created.

### Schema Coverage
- `users`: SELECT (`is_active` check)

---

## UC-USR-005 — [NEGATIVE] Volunteer Attempts to Delete an Object

**Actor:** Volunteer (tbright)  
**Phase:** 1 — Core Framework  
**Module:** Users

### Preconditions
- Tom Bright is logged in with role "Volunteer".
- At least one object exists.

### Steps
1. Open an object record.
2. Attempt to click **Delete Object** (button should be absent or disabled for Volunteers).
3. If accessible, confirm the delete.

### Expected Outcome
- The delete action is blocked at the application layer before any SQL is issued.
- Application displays: *"You do not have permission to delete records."*
- No row deleted; no audit_log entry.

### Schema Coverage
- Role enforcement (application layer) — no DB write

---

## UC-USR-006 — ReadOnly User Views Object Details

**Actor:** ReadOnly  
**Phase:** 1 — Core Framework  
**Module:** Users

### Preconditions
- User "awyatt" (Anne Wyatt) exists with `role = 'ReadOnly'`.
- At least one object with media exists.

### Steps
1. Log in as "awyatt".
2. Navigate to the Objects module.
3. Open an object record and view its details and images.

### Expected Outcome
- Object details and images are fully visible.
- All edit, add, and delete controls are absent or disabled.
- No audit_log entry (read-only access).

### Schema Coverage
- `objects`: SELECT
- `media`: SELECT

---

## UC-USR-007 — Audit Log Captures a Record Change

**Actor:** Any  
**Phase:** 1 — Core Framework  
**Module:** Users / AuditLog

### Preconditions
- User "dchen" (David Chen, Registrar) is logged in.
- Object "2024.001.002" exists.

### Steps
1. Log in as dchen.
2. Open object "2024.001.002" and change `condition` from "Fair" to "Good".
3. Save.
4. (As Admin) open the audit log and filter for table = "objects", record_id = <id of 2024.001.002>.

### Expected Outcome
- One new row in `audit_log`:
  - `user_id` matches dchen's user_id
  - `table_name = 'objects'`
  - `record_id` = object's `object_id`
  - `action = 'UPDATE'`
  - `changed_fields = {"condition": {"old": "Fair", "new": "Good"}}`
  - `logged_at` set to approximately now

### Schema Coverage
- `audit_log`: SELECT, INSERT
- `objects`: UPDATE
