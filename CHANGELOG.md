# MCIS Changelog

All notable changes to the MCIS project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [2026-06-16-1342] — Drop Grant Applications Bullet from Phase 4

### Changed
- `docs/MCIS_High_Level_Plan.md` — removed "IMLS and foundation grant applications leveraging open access mission" from Phase 4 deliverables; vague aspirational boilerplate for an audience that knows the grant landscape; "Grant Friendly" principle in §2 carries that message adequately

---

## [2026-06-16-1329] — Move IA Publishing to Phase 1

### Changed
- `docs/MCIS_High_Level_Plan.md` — v0.2 → v0.3; Internet Archive publishing moved from standalone Phase 3 into Phase 1 as a core capability:
  - §2 "Public Access Built In" principle updated — publishing available from Phase 1
  - §4 Phase 1 section renamed "Collections, Objects & Internet Archive Publishing — Phase 1"; added Basic IA Publisher bullet (flag → approve → upload primary image + core metadata → write-back ia_identifier)
  - §4 Phase 2 section updated — Full IA Publisher added (configurable field mapping UI, batch queue dashboard, update/unpublish, multi-image upload)
  - §4 standalone "Internet Archive Publishing — Phase 3" section removed
  - §4 Operations Modules renumbered Phase 4 → Phase 3; Data Migration renumbered Phase 6 → Phase 5
  - §5 LIDO reference updated Phase 6 → Phase 5
  - §7 Phase 1 milestone updated to include basic IA publishing
  - §7 Phase 2 milestone updated to include full IA publisher
  - §7 standalone Phase 3 (Publishing) removed; old Phase 4/5/6 renumbered to Phase 3/4/5
  - §9 Competitive Landscape IA Integration column updated — "Yes — Phase 1" (was "Yes — built in")
  - §9 closing paragraph updated to note IA publishing available from first release

---

## [2026-06-16-1324] — Simplify Accessible by Design Principle

### Changed
- `docs/MCIS_High_Level_Plan.md` — "Accessible by Design" principle reworded to "UI will be cross-platform and standards-based"; removed WCAG 2.1 AA reference (web standard, not applicable to a desktop app; jargon for this audience)

---

## [2026-06-16-1316] — Remove Advisory Board Section from Plan

### Changed
- `docs/MCIS_High_Level_Plan.md` — removed §9 Advisory Board (group is informal and dynamic; no need to document it in the plan); renumbered Competitive Landscape, Risks, and Glossary as §9–11

---

## [2026-06-16-1312] — High-Level Plan Revised for Advisory Board

### Changed
- `docs/MCIS_High_Level_Plan.md` — major revision (v0.1 → v0.2) to target a senior museum director and archivist advisory board audience:
  - Added "The problem MCIS addresses" framing paragraph to §1
  - Added "What these institutions have in common" to §3 (Target Institutions)
  - Restructured §4 as "What MCIS Will Do" — reframed from module names to collections-operations capabilities
  - **Added §5 Standards Alignment** — SPECTRUM, Dublin Core, LIDO, AAM Core Standards, IMLS Data Stewardship; how MCIS aligns with each
  - Rewrote §6 Technology Overview — simplified rationale in plain language; added "what it means for your institution" framing; added single-user SQLite fallback note and data ownership paragraph
  - Updated §7 Phase 0 milestone to reflect current progress (schema and use cases complete)
  - **Added §8 Current Status** — table showing completion state of each Phase 0 deliverable
  - **Added §9 Advisory Board** — advisory board role, what members are being asked to review now, and how to provide feedback
  - Updated §10 Competitive Landscape — more nuanced CollectiveAccess comparison
  - Added "adoption by target institutions" and "schema decisions diverge from practice" risks to §11
  - Added AAM, SPECTRUM to §12 Glossary; updated TMS description (Gallery Systems → Axiell)
  - Renumbered sections (Competitive Landscape is now §10, Risks §11, Glossary §12)

---

## [2026-06-16-1309] — Remove Stale HTML Source

### Removed
- `docs/MCIS_High_Level_Plan.html` — original LibreOffice export; superseded by `docs/MCIS_High_Level_Plan.md` which is the living document going forward

---

## [2026-06-16-1302] — Phase 1 Schema Critique Fixes

### Changed
- `docs/schema_phase1.md`:
  - Added `maker VARCHAR(255)` to `objects` — artist, maker, manufacturer, or creator; free text with note on controlled vocabulary as a future enhancement
  - Added `condition_date DATE` to `objects` — structured date of most recent condition assessment; replaces embedding dates in `condition_notes` free text
  - Added `UNIQUE` constraint to `media.file_path` — prevents two records referencing the same physical file (storage accounting and safe delete)
  - Added `finding_aid_is_url BOOLEAN NOT NULL DEFAULT FALSE` to `collections` — explicit URL vs free-text flag; avoids heuristic URL detection in the UI
  - Added single-role limitation note to `users` — one role per user; junction table path documented if combined roles are needed in future
  - Added `ia_identifier` assignment timing note to `objects` — NULL until Phase 3 upload succeeds; assigned by IA API, not by the registrar
  - Added accepted MIME types list to `media` notes — `image/jpeg`, `image/tiff`, `image/png`, `image/gif`, `application/pdf`; video and audio deferred
  - Added single-collection UX note to `collections` — UI should pre-select the only collection when one exists rather than requiring user to choose from a one-item list
  - Updated document timestamp

---

## [2026-06-16-1220] — Use Case Critique Fixes + Schema Update

### Added
- `docs/use_cases/UC-Phase1-Users.md` — UC-USR-008: successful login updates `last_login`; UC-USR-009: user changes password (hash excluded from audit log); UC-USR-010: Admin reviews audit log (filter by table, user, date range)
- `docs/use_cases/UC-Phase2-Loans.md` — UC-LNS-010: create incoming loan (`loan_type = 'Incoming'`); UC-LNS-011: find all loans for a specific object (loan history)
- `docs/use_cases/UC-Phase2-Locations.md` — UC-LOC-008: list all objects at a location
- `docs/use_cases/UC-Phase2-Donors.md` — UC-DNR-008: search donors by name (case-insensitive, excludes inactive by default)

### Changed
- `docs/schema_phase1.md`:
  - Added `ia_queued BOOLEAN NOT NULL DEFAULT FALSE` to `objects` — publish queue flag, separate from `ia_published` (confirmed live on IA); documented two-flag IA workflow
  - Added `ia_queued` index to `objects`
  - Documented cascade-delete audit policy: parent DELETE is logged with full row snapshot; cascaded child rows are not individually logged
- `docs/use_cases/UC-Phase1-Objects.md`:
  - UC-OBJ-009: corrected to set `ia_queued` (not `ia_published`) for publish queue flagging; updated precondition to cite UC-OBJ-005
  - UC-OBJ-002: added missing `audit_log` INSERT to expected outcome
  - Dependency citations added to preconditions for UC-OBJ-003 through UC-OBJ-009
- `docs/use_cases/UC-Phase1-Collections.md`:
  - UC-COL-003: added sort order (alphabetical by name) and empty-list expected outcome; strengthened preconditions
  - Dependency citations added to UC-COL-002, UC-COL-004
- `docs/use_cases/UC-Phase1-Media.md`:
  - UC-MED-004: replaced SQL implementation detail in expected outcome with observable UI behaviour (atomic swap, no moment where both or neither are primary)
  - UC-MED-007: resolved open question — cascade-deleted media rows are not individually logged (references schema_phase1.md audit_log notes)
  - Dependency citations added to all UC-MED UCs
- `docs/use_cases/UC-Phase1-Users.md`:
  - UC-USR-003/UC-USR-005 ordering conflict documented in preconditions of both UCs (tbright must be active for UC-USR-005)
  - Dependency citations added throughout
- `docs/use_cases/UC-Phase2-Loans.md` — dependency citations added to UC-LNS-002 through UC-LNS-005, UC-LNS-008, UC-LNS-009
- `docs/use_cases/UC-Phase2-Locations.md` — dependency citations added to UC-LOC-002 through UC-LOC-006
- `docs/use_cases/UC-Phase2-Donors.md` — dependency citations added to UC-DNR-003 through UC-DNR-007
- `docs/use_cases/index.md` — new UCs added; ordering notes for UC-USR-003/005; total now 42 use cases

---

## [2026-06-16-1202] — README Documentation Table

### Changed
- `README.md` — added Documentation section: a table linking all key project documents (high-level plan, Phase 1 schema, Phase 2 schema, use cases index)

---

## [2026-06-16-1159] — Use Cases

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

## [2026-06-16-1115] — Schema Design Phase 2 + Critique Fixes

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

## [2026-06-15-0319] — Phase 1 Schema Design

### Added
- `docs/schema_phase1.md` — Phase 1 schema design covering `collections`, `objects`, `media`, `users`, and `audit_log`; includes conventions table, full column definitions with types/constraints/notes, indexes, design decision rationale, entity relationships diagram, and deferred-to-Phase-2 section

### Changed
- `README.md` — deployment sentence reworded to be role-neutral ("A lightweight desktop client can be installed on a laptop or workstation")

---

## [2026-06-14-2200] — Project Documentation

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

- **2026-06-16** — Phase 1 schema critique fixes (maker, condition_date, file_path UNIQUE, finding_aid_is_url, MIME types, single-role note); CHANGELOG timestamps updated to HHMM; use case critique fixes (42 cases, 6 new UCs); ia_queued added to Phase 1 schema; README documentation table; use cases collection (36 cases initial); Phase 2 schema design and critique passes; deployment model clarified; CHANGELOG added
- **2026-06-15** — Phase 1 schema design (collections, objects, media, users, audit_log) added; README deployment wording improved
- **2026-06-14** — Project launched: high-level plan converted from HTML and fully revised; README written and refined; critique passes on both documents
