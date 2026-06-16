# MCIS Changelog

All notable changes to the MCIS project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [2026-06-16] — Use Cases

### Added
- `docs/use_cases/index.md` — master index of all 36 use cases with module, actor, and file references; includes sources and references section (schema documents, high-level plan, *Museum Registration Methods* AAM 5th ed., Collections Trust SPECTRUM standard)
- `docs/use_cases/UC-Phase1-Collections.md` — 5 use cases: create collection, edit collection, view all collections, [NEGATIVE] delete collection with objects, delete empty collection
- `docs/use_cases/UC-Phase1-Objects.md` — 9 use cases: accession object, accession with fuzzy date, edit object, condition assessment, assign credit line and rights statement, search by accession number, search by title keyword, [NEGATIVE] duplicate accession number, flag for IA publication
- `docs/use_cases/UC-Phase1-Media.md` — 7 use cases: add primary image, add non-primary image, [NEGATIVE] add second primary image, change primary image, reorder images, delete image, delete object with media cascade
- `docs/use_cases/UC-Phase1-Users.md` — 7 use cases: create Admin user, create Volunteer user, deactivate user, [NEGATIVE] login with deactivated account, [NEGATIVE] Volunteer attempts delete, ReadOnly views object, audit log captures change
- `docs/use_cases/UC-Phase2-Locations.md` — 7 use cases: create room, create case in hierarchy, assign object to location, move object, retire location, [NEGATIVE] delete parent with children, create off-site location
- `docs/use_cases/UC-Phase2-Loans.md` — 9 use cases: create outgoing loan, add objects to loan, record condition-out, return one object from multi-object loan, return last object and close loan, view overdue dashboard, cancel loan, [NEGATIVE] delete object on active loan, [NEGATIVE] add same object twice
- `docs/use_cases/UC-Phase2-Donors.md` — 7 use cases: add individual donor, add organization donor, link object to donor, mark anonymous, record acknowledgment, [NEGATIVE] set acknowledgment date without sent flag, deactivate deceased donor

---

## [2026-06-16] — Schema Design Phase 2 + Critique Fixes

### Added
- `docs/schema_phase2.md` — Phase 2 schema design covering `locations`, `loans`, `loan_objects`, and `donors`; includes Phase 1 migrations (location_id FK, donor_id on objects), full column definitions with types, constraints, indexes, entity relationship diagram, and deferred-to-Phase-3 section
- `CHANGELOG.md` — project changelog built from all existing commits using Keep a Changelog format

### Changed
- `docs/schema_phase2.md` — first critique pass:
  - Added `location_code VARCHAR(50) UNIQUE` to `locations` for labelling and signage
  - Added note on `locations.name` uniqueness limitations at the same hierarchy level
  - Removed `NOT NULL` from `loans.insurance_currency` (nullable when `insurance_value` is null)
  - Removed `'Overdue'` from `loans.status` CHECK constraint — overdue now computed on-the-fly as `end_date < CURRENT_DATE AND status = 'Active'`
  - Added `purpose TEXT` to `loans`
  - Added `notes TEXT` and `created_at TIMESTAMPTZ` to `loan_objects`
  - Promoted `UNIQUE (loan_id, object_id)` from index note to formal table constraint
  - Added `donor_type VARCHAR(20)` to `donors` to distinguish Individual from Organization
  - Clarified `name`/`organization` field semantics based on `donor_type`
  - Added `is_active BOOLEAN NOT NULL DEFAULT TRUE` to `donors`
  - Added `acknowledgment CHECK` constraint note to `donors`
  - Fixed entity relationship diagram — `loans` and `objects` are now correctly shown as co-parents of `loan_objects`
- `docs/schema_phase2.md` — second critique pass:
  - Removed `DEFAULT 'USD'` from `loans.insurance_currency` — no default; application always writes both fields together
  - Added `start_date` index to `loans` for fiscal-year and date-range queries
  - Added `updated_at TIMESTAMPTZ` to `loan_objects` (table has mutable columns — condition_in, returned_at, notes)
  - Clarified multi-object loan return condition: set `loans.status = 'Returned'` only when all `loan_objects.returned_at` are non-NULL
  - Promoted `donors` acknowledgment CHECK from note to actual table-level constraint
  - Added bottom-up deletion note to `locations` (RESTRICT requires children deleted before parent)
  - Restored `audit_log` to Phase 2 cumulative ER diagram; restructured diagram for clarity
- `docs/schema_phase1.md` — addressed critique points:
  - Added `ON DELETE RESTRICT` as the default FK behaviour convention
  - Added UTF-8 encoding convention
  - Moved `updated_at` auto-maintenance note from `collections` to Conventions
  - Added `ON DELETE RESTRICT` to `objects.collection_id` FK
  - Added `credit_line TEXT` to `objects` (public attribution, distinct from provenance)
  - Added `rights_statement TEXT` to `objects` (Phase 3 Internet Archive dependency)
  - Renamed `media.upload_date` to `created_at` for consistency across all tables
  - Added `original_filename VARCHAR(255)` to `media`
  - Added `file_size BIGINT` to `media`
  - Added `sort_order INTEGER NOT NULL DEFAULT 0` to `media`
  - Changed `audit_log.record_id` from `INTEGER` to `BIGINT`
- `docs/MCIS_High_Level_Plan.md` — clarified deployment model in §4.2:
  - Database runs in a virtual isolated environment on existing hardware
  - Client install uses standard OS-specific installation processes
- Timestamps updated on plan and schema documents

---

## [2026-06-15] — Phase 1 Schema Design

### Added
- `docs/schema_phase1.md` — Phase 1 schema design covering `collections`, `objects`, `media`, `users`, and `audit_log`; includes conventions table, full column definitions with types/constraints/notes, indexes, design decision rationale, entity relationships diagram, and deferred-to-Phase-2 section

### Changed
- `README.md` — deployment sentence reworded to be role-neutral ("A lightweight desktop client can be installed on a laptop or workstation")

---

## [2026-06-14] — Project Documentation

### Added
- `docs/MCIS_High_Level_Plan.md` — full high-level project plan converted from HTML source; covers project overview, goals and guiding principles, target users, system architecture, module architecture, core database entities, phased development plan (Phases 0–6), competitive landscape, risks and mitigations, immediate next steps, and glossary
- `README.md` — concise project summary document linking to the full plan; covers overview, two-tier architecture, key principles, technology stack, project status, and getting-involved section

### Changed
- `docs/MCIS_High_Level_Plan.md`:
  - Applied future tense throughout to reflect in-development status
  - Added document version/date metadata subtitle
  - Switched desktop client from PyQt6 to PySide6 (LGPL v3) with licensing rationale
  - Added Accessible by Design principle to goals table
  - Added `Media` and `AuditLog` entities to Section 6
  - Fixed `Loan` entity to use a `LoanObject` junction table (was incorrectly 1:1 with Object)
  - Split Phase 1 into Phase 0 (Project Infrastructure) and Phase 1 (Core Framework)
  - Renamed misplaced §7.6 Data Conversion to Phase 6 — Data Migration with correct heading
  - Added milestone acceptance criteria to every phase
  - Updated competitive table: "proposed" → "in development"
  - Added backup/recovery and donor/member PII risks to Section 9
  - Marked completed items in Immediate Next Steps
  - Added Section 11 Glossary (10 terms)
  - Added Data Conversion section (later promoted to Phase 6)
- `README.md`:
  - Rewritten as a concise project summary; detailed content moved to plan document
  - Renamed "What it will do" section to "Overview"
  - Added framing sentence before two-tier architecture description
  - Added deployment context paragraph (client/server, no web server required)
  - Removed architecture fact from principles list
  - Renamed "ORM" row to "Database layer" for non-technical readers
  - Added GitHub URL to Getting Involved section
  - Reframed license uncertainty as a near-term commitment
  - Added Data Portable and Accessible by Design principles

---

## Version History Summary

- **2026-06-16** — Use cases collection added (36 cases, 7 modules, Phases 1–2); Phase 2 schema design added; two critique passes on Phase 2 schema; Phase 1 schema critique pass; deployment model clarified in plan; CHANGELOG added
- **2026-06-15** — Phase 1 schema design (collections, objects, media, users, audit_log) added; README deployment wording improved
- **2026-06-14** — Project launched: high-level plan converted from HTML and fully revised; README written and refined; critique passes on both documents
