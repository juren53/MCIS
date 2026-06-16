# Use Cases — Phase 2: Donors Module

---

## UC-DNR-001 — Add an Individual Donor

**Actor:** Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Donors

### Preconditions
- No donor with the name "Eleanor Pruitt" exists.

### Test Data
| Field               | Value |
| :------------------ | :---- |
| donor_type          | Individual |
| name                | Eleanor Pruitt |
| organization        | (blank — not applicable for Individual) |
| email               | e.pruitt@example.com |
| phone               | 555-317-0042 |
| address             | 14 Maple Avenue, Millbrook, NY 12545 |
| is_anonymous        | FALSE |
| gift_restrictions   | Objects must remain in the permanent collection; no deaccession |
| acknowledgment_sent | FALSE |

### Steps
1. Open the Donors module.
2. Click **New Donor**.
3. Set donor_type to "Individual".
4. Enter name, email, phone, address, and gift_restrictions.
5. Leave acknowledgment_sent unchecked.
6. Click **Save**.

### Expected Outcome
- One new row in `donors`:
  - `donor_type = 'Individual'`
  - `name = 'Eleanor Pruitt'`, `organization IS NULL`
  - Contact fields (email, phone, address) as entered — these are PII
  - `is_anonymous = FALSE`
  - `gift_restrictions` as entered
  - `acknowledgment_sent = FALSE`, `acknowledgment_date IS NULL`
  - `is_active = TRUE`
- `audit_log`: `action = 'CREATE'`, `table_name = 'donors'`
- Note: email, phone, and address must not be logged in `audit_log.changed_fields` unless the deployment data retention policy explicitly permits it.

### Schema Coverage
- `donors`: INSERT
- `audit_log`: INSERT

---

## UC-DNR-002 — Add an Organization Donor

**Actor:** Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Donors

### Preconditions
- No donor named "Millbrook Heritage Trust" exists.

### Test Data
| Field             | Value |
| :---------------- | :---- |
| donor_type        | Organization |
| name              | Millbrook Heritage Trust |
| organization      | (not used for Organization type) |
| email             | grants@millbrookheritage.org |
| gift_restrictions | (none) |
| is_anonymous      | FALSE |

### Steps
1. Click **New Donor**.
2. Set donor_type to "Organization".
3. Enter name ("Millbrook Heritage Trust") and email. Leave organization blank.
4. Click **Save**.

### Expected Outcome
- New row in `donors`:
  - `donor_type = 'Organization'`
  - `name = 'Millbrook Heritage Trust'` (institution name in the `name` field)
  - `organization IS NULL` (unused for Organization type)
- Donor appears in the donor list.

### Schema Coverage
- `donors`: INSERT

---

## UC-DNR-003 — Link a Donated Object to Its Donor

**Actor:** Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Donors

### Preconditions
- Donor "Eleanor Pruitt" exists (`donor_id` assigned).
- Object "2024.001.002" exists with `donor_id IS NULL`.

### Steps
1. Open object "2024.001.002".
2. In the Donor field, select "Eleanor Pruitt".
3. Save.

### Expected Outcome
- `objects.donor_id` updated to Eleanor Pruitt's `donor_id`.
- If Eleanor Pruitt's donor record is later deleted, `objects.donor_id` is set to NULL (`ON DELETE SET NULL`) — the provenance field retains the narrative history.
- `audit_log`: `action = 'UPDATE'` on `objects`.

### Schema Coverage
- `objects`: UPDATE (`donor_id`)
- `audit_log`: INSERT

---

## UC-DNR-004 — Mark a Donor as Anonymous

**Actor:** Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Donors

### Preconditions
- Donor "Eleanor Pruitt" exists with `is_anonymous = FALSE`.

### Steps
1. Open Eleanor Pruitt's donor record.
2. Check **Anonymous donor**.
3. Save.

### Expected Outcome
- `donors.is_anonymous = TRUE`.
- The donor's name is suppressed on all public-facing displays (IA published records, public labels).
- The full internal record remains accessible to staff.
- Any object with `credit_line = 'Gift of Eleanor Pruitt, 2024'` should be reviewed — public display of that credit line is now inconsistent with the anonymous flag. (Application should warn; enforcement is a Phase 3/UI concern.)

### Schema Coverage
- `donors`: UPDATE (`is_anonymous`)
- `audit_log`: INSERT

---

## UC-DNR-005 — Record Acknowledgment Sent and Date

**Actor:** Registrar, Staff  
**Phase:** 2 — Collections Modules  
**Module:** Donors

### Preconditions
- Donor "Eleanor Pruitt" has `acknowledgment_sent = FALSE` and `acknowledgment_date IS NULL`.

### Test Data
| Field               | Value |
| :------------------ | :---- |
| acknowledgment_sent | TRUE |
| acknowledgment_date | 2024-04-02 |

### Steps
1. Open Eleanor Pruitt's donor record.
2. Check **Acknowledgment Sent**.
3. Enter acknowledgment_date "2024-04-02".
4. Save.

### Expected Outcome
- `acknowledgment_sent = TRUE`, `acknowledgment_date = '2024-04-02'`.
- The table-level CHECK `(acknowledgment_date IS NULL OR acknowledgment_sent = TRUE)` is satisfied.
- `audit_log`: `action = 'UPDATE'` recording both field changes.

### Schema Coverage
- `donors`: UPDATE (`acknowledgment_sent`, `acknowledgment_date`)
- `audit_log`: INSERT

---

## UC-DNR-006 — [NEGATIVE] Set acknowledgment_date Without acknowledgment_sent

**Actor:** Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Donors

### Preconditions
- Donor "Eleanor Pruitt" has `acknowledgment_sent = FALSE`.

### Steps
1. Open Eleanor Pruitt's donor record.
2. Leave **Acknowledgment Sent** unchecked.
3. Enter acknowledgment_date "2024-04-02".
4. Click **Save**.

### Expected Outcome
- The operation is blocked by the table-level CHECK: `CHECK (acknowledgment_date IS NULL OR acknowledgment_sent = TRUE)`.
- Application displays: *"Acknowledgment date cannot be set unless Acknowledgment Sent is checked."*
- No row updated; no audit_log entry.

### Schema Coverage
- `donors`: UPDATE (blocked by CHECK constraint)

---

## UC-DNR-007 — Deactivate a Deceased Donor

**Actor:** Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Donors

### Preconditions
- Donor "Eleanor Pruitt" is active (`is_active = TRUE`).

### Steps
1. Open Eleanor Pruitt's donor record.
2. Uncheck **Active** (or click **Deactivate**).
3. Optionally add a note: "Deceased, June 2025".
4. Save.

### Expected Outcome
- `donors.is_active = FALSE`.
- The row is NOT deleted — donation history and the link from objects to this donor are preserved.
- Inactive donors do not appear in donor search results when adding new donations.
- `audit_log`: `action = 'UPDATE'`

### Schema Coverage
- `donors`: UPDATE (`is_active`, `notes`)
- `audit_log`: INSERT
