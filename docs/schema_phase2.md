# MCIS Phase 2 Schema Design

_Draft Version 0.1 — 2026-06-16_

Schema for the entities required by Phase 2 (Collections Modules): Locations, Loans, LoanObjects, and Donors. Also documents the migrations that extend Phase 1 tables once these entities exist.

All conventions established in [schema_phase1.md](schema_phase1.md) apply here. This document notes only additions or deviations.

---

## Phase 1 migrations

The following changes are applied to existing Phase 1 tables at the start of Phase 2, once the new tables they reference have been created.

### objects — add FK constraint on location_id

```sql
ALTER TABLE objects
    ADD CONSTRAINT fk_objects_location
    FOREIGN KEY (location_id) REFERENCES locations(location_id)
    ON DELETE RESTRICT;
```

`location_id` was included in the Phase 1 `objects` table as a nullable INTEGER with no FK. This migration adds the constraint. `ON DELETE RESTRICT` prevents a location from being deleted while objects are assigned to it.

### objects — add donor_id

```sql
ALTER TABLE objects
    ADD COLUMN donor_id INTEGER
    REFERENCES donors(donor_id) ON DELETE SET NULL;
```

Links an object to its primary donor. `ON DELETE SET NULL` means deleting a donor record does not delete the object — the link is cleared and the provenance field retains the narrative history. A junction table for co-donations (multiple donors per object) is a future enhancement.

---

## 1. locations

Represents physical and logical places where objects are stored or displayed. Supports a parent/child hierarchy — a Room contains Cases; a Case contains individual positions.

| Column           | Type         | Constraints                                                          | Description                                                        |
| :--------------- | :----------- | :------------------------------------------------------------------- | :----------------------------------------------------------------- |
| location_id      | SERIAL       | PRIMARY KEY                                                          |                                                                    |
| name             | VARCHAR(255) | NOT NULL                                                             | Display name, e.g. "Gallery A", "Case 3", "Off-site Storage Unit" |
| location_type    | VARCHAR(20)  | NOT NULL CHECK (location_type IN ('Room','Case','Storage','Offsite'))| Physical classification of the location                            |
| parent_location_id | INTEGER    | REFERENCES locations(location_id) ON DELETE RESTRICT                 | Parent location in the hierarchy. NULL for top-level locations.    |
| description      | TEXT         |                                                                      | Narrative description of the location                              |
| notes            | TEXT         |                                                                      | Operational notes (climate control, access restrictions, etc.)     |
| is_active        | BOOLEAN      | NOT NULL DEFAULT TRUE                                                | Inactive locations cannot receive objects but history is retained  |
| created_at       | TIMESTAMPTZ  | NOT NULL DEFAULT NOW()                                               |                                                                    |
| updated_at       | TIMESTAMPTZ  | NOT NULL DEFAULT NOW()                                               |                                                                    |

**Indexes:**
- `parent_location_id` — hierarchy traversal
- `location_type` — filter by type
- `is_active` — exclude retired locations from active lookups

**Notes:**
- The hierarchy is intentionally shallow in typical use: Building → Room → Case → Position. Deep nesting is supported but not required.
- `ON DELETE RESTRICT` on the self-referential FK prevents deleting a parent location while child locations exist.
- Objects on loan are assigned to the borrower's location record (a location of type `Offsite`), not tracked separately. The Loans table carries the borrower contact details.
- Setting `is_active = FALSE` retires a location without removing its history from object location records.

---

## 2. loans

Records an outgoing or incoming loan agreement at the header level. The specific objects covered by each loan are tracked in `loan_objects`.

| Column              | Type           | Constraints                                                                            | Description                                                          |
| :------------------ | :------------- | :------------------------------------------------------------------------------------- | :------------------------------------------------------------------- |
| loan_id             | SERIAL         | PRIMARY KEY                                                                            |                                                                      |
| loan_type           | VARCHAR(10)    | NOT NULL CHECK (loan_type IN ('Outgoing','Incoming'))                                  | Outgoing: museum lends to external party. Incoming: museum borrows.  |
| counterparty_name   | VARCHAR(255)   | NOT NULL                                                                               | Name of the borrowing or lending institution or individual           |
| counterparty_contact| TEXT           |                                                                                        | Address, email, and phone for the counterparty                       |
| start_date          | DATE           | NOT NULL                                                                               | Date the loan begins                                                 |
| end_date            | DATE           |                                                                                        | Expected return date. NULL for open-ended loans.                     |
| insurance_value     | NUMERIC(12,2)  |                                                                                        | Declared insurance value covering all objects in the loan            |
| insurance_currency  | CHAR(3)        | NOT NULL DEFAULT 'USD'                                                                 | ISO 4217 currency code                                               |
| agreement_ref       | VARCHAR(255)   |                                                                                        | Loan agreement document reference number or filename                 |
| agreement_notes     | TEXT           |                                                                                        | Notes about special terms or conditions in the agreement             |
| status              | VARCHAR(10)    | NOT NULL DEFAULT 'Pending' CHECK (status IN ('Pending','Active','Returned','Overdue','Cancelled')) | Lifecycle status of the loan                          |
| created_at          | TIMESTAMPTZ    | NOT NULL DEFAULT NOW()                                                                 |                                                                      |
| updated_at          | TIMESTAMPTZ    | NOT NULL DEFAULT NOW()                                                                 |                                                                      |

**Indexes:**
- `loan_type` — filter outgoing vs incoming
- `status` — active loan dashboard and overdue alerts
- `end_date` — deadline alert queries (`WHERE end_date < NOW() AND status = 'Active'`)
- `counterparty_name` — search by institution

**Notes:**
- `counterparty_name` and `counterparty_contact` store the external party for both loan types. For outgoing loans this is the borrower; for incoming loans this is the lender.
- `insurance_value` covers the full loan. Per-object insurance values, if needed, are a future enhancement on `loan_objects`.
- `status = 'Overdue'` is set by the application when `end_date` has passed and the loan has not been returned. The application is responsible for running periodic checks and updating this field.
- Loan agreements (PDF files) should be stored in the media directory and referenced by `agreement_ref`, not stored as BLOBs in the database.

---

## 3. loan_objects

Junction table linking loans to the specific objects they cover. Also records the condition of each object at departure and return.

| Column             | Type         | Constraints                                                                   | Description                                                         |
| :----------------- | :----------- | :---------------------------------------------------------------------------- | :------------------------------------------------------------------ |
| loan_object_id     | SERIAL       | PRIMARY KEY                                                                   |                                                                     |
| loan_id            | INTEGER      | NOT NULL REFERENCES loans(loan_id) ON DELETE RESTRICT                         | Parent loan                                                         |
| object_id          | INTEGER      | NOT NULL REFERENCES objects(object_id) ON DELETE RESTRICT                     | Object covered by the loan                                          |
| condition_out      | VARCHAR(20)  | CHECK (condition_out IN ('Excellent','Good','Fair','Poor','Unknown'))          | Condition grade recorded when object left the museum                |
| condition_out_notes| TEXT         |                                                                               | Narrative condition notes at departure                              |
| condition_in       | VARCHAR(20)  | CHECK (condition_in IN ('Excellent','Good','Fair','Poor','Unknown'))           | Condition grade recorded when object was returned                   |
| condition_in_notes | TEXT         |                                                                               | Narrative condition notes at return                                 |
| returned_at        | TIMESTAMPTZ  |                                                                               | Timestamp the object was checked back in. NULL while still on loan. |

**Indexes:**
- `(loan_id, object_id)` — UNIQUE; an object cannot appear twice on the same loan
- `object_id` — find all loans (past and present) for a given object

**Notes:**
- `ON DELETE RESTRICT` on both FKs: a loan cannot be deleted while it has objects, and an object cannot be deleted while it is on a loan record. Cancelling a loan sets `status = 'Cancelled'` on the `loans` row rather than deleting it.
- `condition_out` should be recorded before the object leaves. `condition_in` is recorded at return. Both are nullable to allow partial completion.
- The condition vocabulary matches `objects.condition` deliberately. Any future change to the allowed values must be applied consistently across both tables.
- `returned_at` is set when the last object on a loan is checked back in; the application should then set `loans.status = 'Returned'`.

---

## 4. donors

Contact records for individuals and organisations who have donated objects to the collection.

| Column               | Type         | Constraints              | Description                                                                   |
| :------------------- | :----------- | :----------------------- | :---------------------------------------------------------------------------- |
| donor_id             | SERIAL       | PRIMARY KEY              |                                                                               |
| name                 | VARCHAR(255) | NOT NULL                 | Full name of the donor or organisation                                        |
| organization         | VARCHAR(255) |                          | Institution or organisation name if the donor represents one                 |
| email                | VARCHAR(255) |                          | Contact email — PII; subject to data retention policy                        |
| phone                | VARCHAR(50)  |                          | Contact phone — PII; subject to data retention policy                        |
| address              | TEXT         |                          | Mailing address — PII; subject to data retention policy                      |
| is_anonymous         | BOOLEAN      | NOT NULL DEFAULT FALSE   | If TRUE, suppress donor name from public-facing displays                      |
| gift_restrictions    | TEXT         |                          | Any restrictions placed by the donor on use or display of donated objects     |
| acknowledgment_sent  | BOOLEAN      | NOT NULL DEFAULT FALSE   | Whether a formal acknowledgment letter has been sent                          |
| acknowledgment_date  | DATE         |                          | Date the acknowledgment letter was sent                                       |
| notes                | TEXT         |                          | Internal notes about the donor relationship                                   |
| created_at           | TIMESTAMPTZ  | NOT NULL DEFAULT NOW()   |                                                                               |
| updated_at           | TIMESTAMPTZ  | NOT NULL DEFAULT NOW()   |                                                                               |

**Indexes:**
- `name` — donor search
- `acknowledgment_sent` — filter for pending acknowledgments

**Notes:**
- `email`, `phone`, and `address` are PII and must be handled according to the data retention policy described in the deployment guide. Consider whether these fields should be encrypted at rest.
- `is_anonymous` suppresses the donor name in public-facing contexts (IA publishing, public labels) but the full record is retained internally.
- `gift_restrictions` applies to all objects donated by this donor. Per-object restrictions are tracked in `objects.provenance` at this stage; a separate `restrictions` table is a future enhancement.
- `acknowledgment_sent` / `acknowledgment_date` track the most recent acknowledgment. A donor who makes multiple gifts will need the acknowledgment flags reset after each new gift — a full gift history table is a Phase 4 candidate.
- The link from a donated object back to its donor is `objects.donor_id` (added by Phase 2 migration above). Multiple donors per object is not supported in Phase 2.

---

## Entity relationships (Phase 2, cumulative)

```
donors
  │
  └── objects (donor_id → donors.donor_id, SET NULL on delete)

locations
  │
  ├── locations (parent_location_id → locations.location_id, self-referential)
  │
  └── objects (location_id → locations.location_id, RESTRICT on delete)

collections
  │
  └── objects (collection_id → collections.collection_id, RESTRICT on delete)
        │
        ├── media (object_id → objects.object_id, CASCADE on delete)
        │
        └── loan_objects (object_id → objects.object_id, RESTRICT on delete)
              │
              └── loans (loan_id → loans.loan_id, RESTRICT on delete)

users
  │
  └── audit_log (user_id → users.user_id, nullable)
```

---

## Deferred to Phase 3

The following are out of scope for Phase 2 and will be addressed in the Phase 3 schema document:

- IA Publisher queue table (tracks publish jobs and their status)
- Any schema changes required to support batch IA metadata mapping

---

_2026-06-16-1110_
