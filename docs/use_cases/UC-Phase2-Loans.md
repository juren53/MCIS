# Use Cases — Phase 2: Loans Module

---

## UC-LNS-001 — Create an Outgoing Loan

**Actor:** Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Loans

### Preconditions
- No loan with agreement reference "LN-2024-001" exists.

### Test Data
| Field                | Value |
| :------------------- | :---- |
| loan_type            | Outgoing |
| counterparty_name    | Valley College Art Gallery |
| counterparty_contact | 100 College Drive, Millbrook, NY 12345; curator@valleycollege.edu; 555-204-8800 |
| start_date           | 2024-10-01 |
| end_date             | 2025-03-31 |
| insurance_value      | 5000.00 |
| insurance_currency   | USD |
| agreement_ref        | LN-2024-001 |
| purpose              | Travelling exhibition: "Rural Life in the Hudson Valley" |
| status               | Pending |

### Steps
1. Open the Loans module.
2. Click **New Loan**.
3. Enter all test data above. Leave agreement_notes blank.
4. Click **Save**.

### Expected Outcome
- One new row in `loans`:
  - `loan_type = 'Outgoing'`
  - `status = 'Pending'`
  - `counterparty_name`, `start_date`, `end_date`, `insurance_value`, `insurance_currency` as entered
  - `created_at` and `updated_at` set to now
- No `loan_objects` rows yet — objects are added in UC-LNS-002.
- `audit_log`: `action = 'CREATE'`, `table_name = 'loans'`

### Schema Coverage
- `loans`: INSERT
- `audit_log`: INSERT

---

## UC-LNS-002 — Add Multiple Objects to a Loan

**Actor:** Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Loans

### Preconditions
- Loan "LN-2024-001" exists with `status = 'Pending'`.
- Objects "2024.001.002" and "2024.001.003" exist and are not on any active loan.

### Steps
1. Open loan "LN-2024-001".
2. Click **Add Object**, select "2024.001.002 — Millbrook Post Office".
3. Click **Add Object** again, select "2024.001.003 — Main Street Looking North".
4. Save.

### Expected Outcome
- Two new rows in `loan_objects`:
  - Each references `loan_id` of LN-2024-001
  - `object_id` set for each object
  - `condition_out IS NULL` (recorded later in UC-LNS-003)
  - `returned_at IS NULL`
  - `created_at` set to now; `updated_at` set to now
- `audit_log`: two `CREATE` entries for `loan_objects`

### Schema Coverage
- `loan_objects`: INSERT ×2
- `audit_log`: INSERT ×2

---

## UC-LNS-003 — Record Condition-Out for Each Loan Object

**Actor:** Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Loans

### Preconditions
- Loan "LN-2024-001" has two objects with `condition_out IS NULL`.

### Test Data
| Object           | condition_out | condition_out_notes |
| :--------------- | :------------ | :------------------ |
| 2024.001.002     | Good          | Minor surface dust; no tears or foxing |
| 2024.001.003     | Good          | Excellent condition; no visible defects |

### Steps
1. Open loan "LN-2024-001".
2. For each object, enter condition_out and condition_out_notes.
3. Change loan status from "Pending" to "Active".
4. Save.

### Expected Outcome
- `loan_objects` rows updated with `condition_out` and `condition_out_notes`.
- `loans.status` updated to `'Active'`.
- `updated_at` refreshed on both `loan_objects` rows.
- `audit_log` records the status change and the condition updates.

### Schema Coverage
- `loan_objects`: UPDATE (`condition_out`, `condition_out_notes`, `updated_at`)
- `loans`: UPDATE (`status`)
- `audit_log`: INSERT

---

## UC-LNS-004 — Return One Object from a Multi-Object Loan

**Actor:** Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Loans

### Preconditions
- Loan "LN-2024-001" is Active with two objects; neither has been returned.

### Test Data (for object "2024.001.002")
| Field           | Value |
| :-------------- | :---- |
| condition_in    | Good |
| condition_in_notes | No change from condition-out; light surface dust cleaned |
| returned_at     | (current timestamp) |

### Steps
1. Open loan "LN-2024-001".
2. Select object "2024.001.002".
3. Enter condition_in, condition_in_notes, and confirm return date.
4. Click **Record Return** for this object.

### Expected Outcome
- `loan_objects` row for object "2024.001.002":
  - `condition_in = 'Good'`, `condition_in_notes` set
  - `returned_at` set to current timestamp
  - `updated_at` refreshed
- `loan_objects` row for object "2024.001.003": unchanged (`returned_at IS NULL`).
- `loans.status` remains `'Active'` — loan is not closed until all objects are returned.
- `audit_log`: UPDATE on the returned `loan_objects` row.

### Schema Coverage
- `loan_objects`: UPDATE (`condition_in`, `condition_in_notes`, `returned_at`, `updated_at`)
- `audit_log`: INSERT

---

## UC-LNS-005 — Return the Last Object and Close the Loan

**Actor:** Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Loans

### Preconditions
- Loan "LN-2024-001": object "2024.001.002" already returned; object "2024.001.003" still on loan.

### Test Data (for object "2024.001.003")
| Field              | Value |
| :----------------- | :---- |
| condition_in       | Good |
| condition_in_notes | Minor handling scuff on back of mat, not present at departure |
| returned_at        | (current timestamp) |

### Steps
1. Open loan "LN-2024-001".
2. Record return for object "2024.001.003" (condition_in, notes, returned_at).
3. Save.

### Expected Outcome
- `loan_objects` row for "2024.001.003" updated: `returned_at` set, `condition_in` and notes recorded.
- Application detects that all `loan_objects` now have non-NULL `returned_at` (query: `SELECT COUNT(*) FROM loan_objects WHERE loan_id = ? AND returned_at IS NULL` = 0).
- `loans.status` updated to `'Returned'`.
- `audit_log` records: UPDATE on `loan_objects` and UPDATE on `loans.status`.

### Schema Coverage
- `loan_objects`: UPDATE
- `loans`: UPDATE (`status`)
- `audit_log`: INSERT ×2

---

## UC-LNS-006 — View Overdue Loans Dashboard

**Actor:** Any authenticated user  
**Phase:** 2 — Collections Modules  
**Module:** Loans

### Preconditions
- At least one loan exists with `end_date < CURRENT_DATE AND status = 'Active'`.

### Steps
1. Open the Loans module.
2. Select the **Overdue** filter or view.

### Expected Outcome
- All loans matching `end_date < CURRENT_DATE AND status = 'Active'` are listed.
- Overdue state is computed by the query — no stored `'Overdue'` status value.
- Each overdue loan shows days overdue (computed from `end_date` to today).
- No audit_log entry (read-only operation).

### Schema Coverage
- `loans`: SELECT (`end_date`, `status`)

---

## UC-LNS-007 — Cancel a Pending Loan

**Actor:** Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Loans

### Preconditions
- A loan "LN-2024-002" exists with `status = 'Pending'` and no objects yet added.

### Steps
1. Open loan "LN-2024-002".
2. Click **Cancel Loan** and confirm.

### Expected Outcome
- `loans.status` updated to `'Cancelled'`.
- Loan row is retained (not deleted) for historical reference.
- `audit_log`: `action = 'UPDATE'` recording the status change.

### Schema Coverage
- `loans`: UPDATE (`status`)
- `audit_log`: INSERT

---

## UC-LNS-008 — [NEGATIVE] Delete an Object That Is on an Active Loan

**Actor:** Admin  
**Phase:** 2 — Collections Modules  
**Module:** Loans

### Preconditions
- Object "2024.001.003" has a row in `loan_objects` referencing an Active loan.

### Steps
1. Open object "2024.001.003".
2. Click **Delete Object** and confirm.

### Expected Outcome
- DELETE is blocked by `ON DELETE RESTRICT` on `loan_objects.object_id`.
- Application displays: *"Cannot delete an object that is referenced by a loan record."*
- No rows deleted.

### Schema Coverage
- `objects`: DELETE (blocked by RESTRICT from `loan_objects.object_id`)

---

## UC-LNS-009 — [NEGATIVE] Add the Same Object Twice to a Loan

**Actor:** Registrar  
**Phase:** 2 — Collections Modules  
**Module:** Loans

### Preconditions
- Loan "LN-2024-001" already includes object "2024.001.002".

### Steps
1. Open loan "LN-2024-001".
2. Click **Add Object** and again select "2024.001.002".
3. Save.

### Expected Outcome
- INSERT is blocked by the `UNIQUE (loan_id, object_id)` table constraint.
- Application displays: *"Object 2024.001.002 is already included in this loan."*
- No duplicate row inserted.

### Schema Coverage
- `loan_objects`: INSERT (blocked by UNIQUE constraint)
