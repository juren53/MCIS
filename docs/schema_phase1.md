# MCIS Phase 1 Schema Design

_Draft Version 0.1 — 2026-06-15_

Schema for the five entities required by Phase 1 (Core Framework): Collections, Objects, Media, Users, and AuditLog. Entities for Phase 2 (Locations, Loans, Donors) and beyond are deferred.

---

## Conventions

| Convention     | Rule                                                                                      |
| :------------- | :---------------------------------------------------------------------------------------- |
| Table names    | Plural snake_case: `collections`, `objects`, `media`, `users`, `audit_log`               |
| Primary keys   | `<singular>_id`, type `SERIAL` (or `BIGSERIAL` for high-volume tables)                  |
| Foreign keys   | Named `<referenced_singular>_id`                                                          |
| Timestamps     | `created_at`, `updated_at` — `TIMESTAMPTZ NOT NULL DEFAULT NOW()`                        |
| Booleans       | `is_*` prefix where the meaning is not otherwise obvious                                  |
| Bounded text   | `VARCHAR(n)` for fields with a known maximum; `TEXT` for unbounded narrative fields       |
| Soft delete    | Not used — rows are hard-deleted; the AuditLog captures what was removed and by whom     |

---

## 1. collections

Represents a named grouping of objects managed under a common accession policy. A museum may maintain multiple collections (e.g. "Photograph Collection", "Textile Collection").

| Column           | Type          | Constraints              | Description                                              |
| :--------------- | :------------ | :----------------------- | :------------------------------------------------------- |
| collection_id    | SERIAL        | PRIMARY KEY              |                                                          |
| name             | VARCHAR(255)  | NOT NULL UNIQUE          | Display name of the collection                           |
| description      | TEXT          |                          | Narrative description of scope and content               |
| accession_policy | TEXT          |                          | Rules governing what is accepted into this collection    |
| finding_aid      | TEXT          |                          | Free text or URL pointing to a finding aid document      |
| created_at       | TIMESTAMPTZ   | NOT NULL DEFAULT NOW()   |                                                          |
| updated_at       | TIMESTAMPTZ   | NOT NULL DEFAULT NOW()   |                                                          |

**Indexes:** unique index on `name` (implied by UNIQUE constraint)

**Notes:**
- `finding_aid` is intentionally free-form at this stage; a URL to an external document is the common case.
- `updated_at` must be maintained by the application on every write; a trigger or SQLAlchemy event listener is recommended.

---

## 2. objects

The central entity — one row per catalogued item. This table will be the most frequently queried in the system.

| Column           | Type          | Constraints                                                                 | Description                                                                 |
| :--------------- | :------------ | :-------------------------------------------------------------------------- | :-------------------------------------------------------------------------- |
| object_id        | SERIAL        | PRIMARY KEY                                                                 |                                                                             |
| collection_id    | INTEGER       | NOT NULL REFERENCES collections(collection_id)                              | Parent collection                                                           |
| accession_number | VARCHAR(100)  | NOT NULL UNIQUE                                                             | Museum-assigned identifier, e.g. "2024.001.001"                             |
| title            | VARCHAR(500)  | NOT NULL                                                                    | Display title                                                               |
| description      | TEXT          |                                                                             | Narrative description of the object                                         |
| date_made        | VARCHAR(100)  |                                                                             | Date the object was created by its maker. VARCHAR to support fuzzy dates ("circa 1920", "early 20th century") |
| accession_date   | DATE          |                                                                             | Date the object was formally accessioned into the collection                |
| medium           | VARCHAR(255)  |                                                                             | Materials and technique, e.g. "oil on canvas", "silver gelatin print"       |
| dimensions       | VARCHAR(255)  |                                                                             | Physical size, e.g. "12 × 18 in."                                          |
| condition        | VARCHAR(20)   | CHECK (condition IN ('Excellent','Good','Fair','Poor','Unknown'))           | Condition grade at time of last assessment                                  |
| condition_notes  | TEXT          |                                                                             | Narrative notes from the most recent condition assessment                   |
| provenance       | TEXT          |                                                                             | Ownership and acquisition history                                           |
| location_id      | INTEGER       | (FK constraint added in Phase 2 when locations table exists)                | Current storage or display location. NULL until Phase 2.                    |
| ia_published     | BOOLEAN       | NOT NULL DEFAULT FALSE                                                      | Whether the record has been published to Internet Archive                   |
| ia_identifier    | VARCHAR(255)  | UNIQUE                                                                      | Internet Archive item identifier, e.g. "mcis-photo-2024-001-001"           |
| created_at       | TIMESTAMPTZ   | NOT NULL DEFAULT NOW()                                                      |                                                                             |
| updated_at       | TIMESTAMPTZ   | NOT NULL DEFAULT NOW()                                                      |                                                                             |

**Indexes:**
- `collection_id` — FK lookup
- `accession_number` — unique (implied by UNIQUE constraint)
- `ia_published` — filter for the IA publish queue
- `(title)` — full-text search candidate; consider `tsvector` index in a later phase

**Notes:**
- `date_made` captures when the object was created by its maker, not when it was acquired. VARCHAR is intentional — museum dates are commonly approximate or expressed as ranges.
- `accession_date` is a strict `DATE` field for the formal accession event, which always has a known calendar date.
- `location_id` is included as a nullable INTEGER in Phase 1 with no FK constraint. The `locations` table and the FK constraint are added in Phase 2.
- `condition` uses a CHECK constraint rather than a lookup table for simplicity at this stage. A full condition report history (separate table, multiple assessments per object) is a candidate Phase 2 enhancement.

---

## 3. media

One or more image or document files associated with an object. Multiple media records per object are supported; exactly one per object should be flagged `is_primary = TRUE`.

| Column      | Type          | Constraints                                      | Description                                                              |
| :---------- | :------------ | :----------------------------------------------- | :----------------------------------------------------------------------- |
| media_id    | SERIAL        | PRIMARY KEY                                      |                                                                          |
| object_id   | INTEGER       | NOT NULL REFERENCES objects(object_id) ON DELETE CASCADE | Parent object                                                  |
| file_path   | VARCHAR(1000) | NOT NULL                                         | Path to the file relative to the configured media root directory         |
| file_type   | VARCHAR(50)   | NOT NULL                                         | MIME type, e.g. "image/jpeg", "image/tiff", "application/pdf"           |
| caption     | TEXT          |                                                  | Optional caption or description of the image                             |
| is_primary  | BOOLEAN       | NOT NULL DEFAULT FALSE                           | Whether this is the main display image for the object                    |
| upload_date | TIMESTAMPTZ   | NOT NULL DEFAULT NOW()                           |                                                                          |

**Indexes:**
- `object_id` — retrieve all media for a given object
- Partial unique index: `(object_id) WHERE is_primary = TRUE` — enforces at most one primary image per object

**Notes:**
- `ON DELETE CASCADE` means deleting an object automatically deletes all its associated media records.
- `file_path` is stored as a path relative to a configurable media root so the database is portable across machines with different directory layouts.
- The partial unique index on `(object_id) WHERE is_primary = TRUE` is a PostgreSQL feature. Application code must also enforce this invariant when the database is SQLite (single-user fallback).
- The application layer is responsible for validating MIME types and file extensions before insert.

---

## 4. users

Staff, volunteers, and administrators who access the system.

| Column          | Type          | Constraints                                                                              | Description                                                   |
| :-------------- | :------------ | :--------------------------------------------------------------------------------------- | :------------------------------------------------------------ |
| user_id         | SERIAL        | PRIMARY KEY                                                                              |                                                               |
| username        | VARCHAR(100)  | NOT NULL UNIQUE                                                                          | Login name                                                    |
| full_name       | VARCHAR(255)  | NOT NULL                                                                                 | Display name                                                  |
| email           | VARCHAR(255)  | NOT NULL UNIQUE                                                                          | Contact email; used for account management                    |
| hashed_password | VARCHAR(255)  | NOT NULL                                                                                 | bcrypt hash — plaintext passwords are never stored            |
| role            | VARCHAR(20)   | NOT NULL CHECK (role IN ('Admin','Registrar','Staff','Volunteer','ReadOnly'))            | Access control role                                           |
| is_active       | BOOLEAN       | NOT NULL DEFAULT TRUE                                                                    | Inactive users cannot log in; records are retained for audit  |
| last_login      | TIMESTAMPTZ   |                                                                                          | NULL until first successful login                             |
| created_at      | TIMESTAMPTZ   | NOT NULL DEFAULT NOW()                                                                   |                                                               |
| updated_at      | TIMESTAMPTZ   | NOT NULL DEFAULT NOW()                                                                   |                                                               |

**Roles:**

| Role       | Capabilities                                                     |
| :--------- | :--------------------------------------------------------------- |
| Admin      | Full access; user management; system configuration               |
| Registrar  | Full collections access; cannot manage users                     |
| Staff      | Create and edit records; cannot delete                           |
| Volunteer  | Create records only; cannot edit or delete                       |
| ReadOnly   | View only; no write access                                       |

**Indexes:** `username`, `email` (unique — implied by UNIQUE constraints)

**Notes:**
- Users are never hard-deleted. Setting `is_active = FALSE` revokes access while preserving the audit trail — AuditLog records that reference the user remain meaningful.
- Password hashing must use bcrypt with a work factor of at least 12. The application layer is responsible for hashing before insert and for never logging or transmitting plaintext passwords.
- Role-based capability enforcement is the application's responsibility; the database stores only the role value.

---

## 5. audit_log

Append-only record of every create, update, and delete operation on tracked tables. Provides the change history required for collections stewardship and grant compliance.

| Column         | Type          | Constraints                                                       | Description                                                                     |
| :------------- | :------------ | :---------------------------------------------------------------- | :------------------------------------------------------------------------------ |
| log_id         | BIGSERIAL     | PRIMARY KEY                                                       | BIGSERIAL — audit logs accumulate at high volume over time                      |
| user_id        | INTEGER       | REFERENCES users(user_id)                                         | User who performed the action. NULL for system or migration-initiated changes.  |
| table_name     | VARCHAR(100)  | NOT NULL                                                          | Name of the table that was changed                                              |
| record_id      | INTEGER       | NOT NULL                                                          | Primary key value of the affected row                                           |
| action         | VARCHAR(10)   | NOT NULL CHECK (action IN ('CREATE','UPDATE','DELETE'))           | Type of change                                                                  |
| changed_fields | JSONB         |                                                                   | For UPDATE: `{"field": {"old": ..., "new": ...}}`. For CREATE/DELETE: full row snapshot. |
| logged_at      | TIMESTAMPTZ   | NOT NULL DEFAULT NOW()                                            | Timestamp of the change                                                         |

**Indexes:**
- `(table_name, record_id)` — retrieve the full change history of a specific record
- `user_id` — retrieve all actions by a specific user
- `logged_at` — time-range queries and log rotation

**Notes:**
- This table is append-only. No UPDATE or DELETE should ever be issued against it. It is recommended to revoke UPDATE and DELETE privileges on `audit_log` from the application database user at deployment time.
- `changed_fields` stores a full row snapshot for CREATE and DELETE actions, and only the changed fields (with old and new values) for UPDATE actions.
- `user_id` is nullable to support system-initiated actions (e.g. migration imports, scheduled jobs) where no authenticated user session is present.
- Audit log population will be handled by SQLAlchemy event listeners in the application layer, not database triggers, to keep the logic in Python and fully testable.

---

## Entity relationships (Phase 1)

```
users
  │
  └── audit_log (user_id → users.user_id, nullable)

collections
  │
  └── objects (collection_id → collections.collection_id)
        │
        └── media (object_id → objects.object_id, CASCADE DELETE)
```

`audit_log` cross-references all tables by `(table_name, record_id)` rather than typed foreign keys, so no FK constraints connect it to individual tables.

---

## Deferred to Phase 2

The following tables are out of scope for Phase 1 and will be designed as part of the Phase 2 schema document:

- `locations`
- `loans` and `loan_objects` (junction)
- `donors`

The FK constraint linking `objects.location_id` to `locations.location_id` will be added as a schema migration at the start of Phase 2.
