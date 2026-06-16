# MCIS — Museum Collections Information System

_High Level Project Plan — Draft Version 0.5 — 2026-06-16-1417_

---

## 1. Project Overview

MCIS is a free, open source museum collections information system designed specifically for small, budget-constrained museums, historical societies, and specialized collections. It will deliver professional-grade collections management without the licensing costs or technical complexity of commercial platforms such as PastPerfect, Argus, or The Museum System (TMS).

**The problem MCIS addresses:** Hundreds of small museums, historical societies, and community archives currently manage their collections in Excel spreadsheets, paper registers, or aging standalone software. These institutions cannot afford commercial solutions — many cost thousands of dollars per year in subscription fees — and lack the IT staff to deploy and maintain web-based alternatives. Their collections remain undiscoverable, their data is at risk, and their staff and volunteers face unnecessary administrative burden.

**The MCIS approach:** A free, easy-to-install desktop application that any registrar, staff member, or trained volunteer can learn quickly, backed by a shared database that all workstations connect to simultaneously, with Internet Archive publishing built in from the start.

The system is built on a two-tier public/private architecture:

- **Private Tier** — Full operational data is maintained in a secure, self-hosted database covering collections management, donor records, loans, location tracking, and internal inventory. Data stays on hardware the museum controls — no cloud dependency, no subscription, no vendor lock-in.
- **Public Tier** — Selected collection records and artifact images can be published to the museum's Internet Archive (IA) collection, providing free, permanent, searchable public access aligned with the GLAM (Galleries, Libraries, Archives, Museums) open access movement.

---

## 2. Goals & Guiding Principles

| Principle | Description |
| :--- | :--- |
| Free & Open Source | No licensing fees, ever. Community-developed and maintained. |
| Affordable to Deploy | Runs on hardware the museum already owns, or an inexpensive server. No annual subscription. |
| Simple to Operate | Designed for volunteers and part-time staff with high turnover. Low training curve is a first-order design requirement — not a nice-to-have. |
| Multi-User Ready | Multiple staff and volunteers can work simultaneously from their own workstations, connecting to a shared database. |
| Modular by Design | Museums adopt only the modules they need. Start with Collections and Objects; add Loans, Donors, and Members as capacity and interest grow. |
| Data Portable | Museums own their data in a standard, open format. Full export at any time. No vendor lock-in. |
| Public Access Built In | Internet Archive publishing is a core feature, available from Phase 1. Publishing an approved collection record to the public is a one-step workflow, not a separate project. |
| Standards Aligned | Field names, controlled vocabularies, and workflows reflect professional collections management standards (see §5). |
| Grant Friendly | Open access commitment and IMLS-aligned mission supports grant applications from IMLS, NEH, state humanities councils, and private foundations. |
| Accessible by Design | Designed for users of varying technical ability; interface will be consistent, intuitive, and operable without specialist training. |

---

## 3. Target Institutions

MCIS is designed for institutions that are professionally committed to collections stewardship but cannot afford, or choose not to use, commercial collections management software:

- Small local historical societies operating on volunteer labor
- Community art museums and galleries with limited IT support
- Specialized collections (science, natural history, transportation, military, ethnic heritage) without dedicated registrars
- Educational institutions managing teaching or study collections
- Archives and special collections within libraries, universities, or cultural centers
- Any institution currently managing collections in Excel spreadsheets or paper records

**What these institutions have in common:** They take their collections seriously. Their registrars and volunteers understand accession numbers, provenance, condition reports, and donor acknowledgment. What they lack is affordable, professional-grade software that matches how they actually work.

---

## 4. What MCIS Will Do

MCIS is organized into functional modules. Each module integrates into the full MCIS application; smaller institutions can enable only the modules they need.

### Collections, Objects & Internet Archive Publishing

The foundation of any collections management system — with the feature that sets MCIS apart built in from the first release: direct Internet Archive publishing.

- Create and manage named collections with accession policies and finding aids (free text or URL)
- Accession individual objects with full cataloguing fields: title, maker, date made, medium, dimensions, provenance, credit line, rights statement, condition grade, condition notes, and structured condition assessment date
- Attach multiple images and documents (JPEG, TIFF, PNG, PDF) per object, with primary image designation and sort order
- Field-specific and full-text search across the entire catalogue
- Role-based access control: Admin, Registrar, Staff, Volunteer, and Read-Only — each role has appropriate capabilities
- Complete audit trail of every record creation, edit, and deletion, with user attribution and before/after field values
- **Basic IA Publisher** — Registrar flags objects for publication; Admin approves; the system uploads the object record with its primary image and core metadata (title, maker, date, description, rights statement) to the museum's Internet Archive collection; IA identifier is written back to the object record; publication status is visible on every object

### Locations, Loans & Donors

The operational core for a working museum or historical society, paired with a full-featured IA publisher.

- **Locations** — Hierarchical location tree (building → room → case → shelf); record where every object is; full movement history; off-site and loan-destination locations
- **Loans** — Outgoing and incoming loan agreements with multi-object support; condition-out / condition-in recording; insurance value and currency; return deadlines and overdue alerts; loan status (Pending, Active, Returned, Cancelled)
- **Donors** — Individual and organization donor records; link donated objects to their donors; gift restrictions; acknowledgment letter generation; anonymous donor flag with public suppression; deactivation for deceased or lapsed donors
- **Full IA Publisher** — Configurable field mapping UI (MCIS fields → IA metadata); batch publish queue management and dashboard; update and unpublish support; multi-image upload

### Operations Modules

- **Members / Comms** — Membership tiers, renewal tracking, lapsed-member outreach lists, newsletter distribution
- **Inventory** — Equipment and supply records, vendor contacts, maintenance schedules

### Data Migration

- CSV/Excel import with configurable field mapping — for institutions migrating from spreadsheets
- Support for LIDO and other standard museum data interchange formats
- Batch image import with automatic association to object records

---

## 5. Standards Alignment

MCIS field names and controlled vocabularies are informed by established collections management standards. Alignment with these standards improves data interoperability, satisfies grant reporting requirements, and means that museums using MCIS are building records that are exportable into future systems.

| Standard | How MCIS Aligns |
| :--- | :--- |
| **Dublin Core** | Core descriptive fields — title, maker (creator), date made, description, rights statement — map directly to Dublin Core elements; used as the basis for Internet Archive metadata export |
| **SPECTRUM** (Collections Trust, UK) | Object accession, object entry, location and movement control, loans in/out, condition checking, and the audit trail align with SPECTRUM unit of practice definitions; the workflow design in the use case library explicitly references SPECTRUM procedures |
| **LIDO** (Lightweight Information Describing Objects) | Phase 5 data migration tools will support LIDO XML import/export for interoperability with digital aggregators |
| **AAM Standards** | Object records include the fields required by the *National Standards and Best Practices for U.S. Museums* (American Alliance of Museums, 2008): unique accession number, title, provenance, rights statement, and condition |
| **IMLS Data Stewardship** | Audit logging, data export, and backup guidance align with IMLS data stewardship expectations for grantees; the rights statement field is required before any object can be published publicly |

**Note:** MCIS does not claim certification under any of these standards. The alignment documented here reflects best-effort conformance at the schema and workflow level, and will be refined through the advisory board [TBD] review process.

---

## 6. Technology Overview

MCIS is a desktop application — not a web application. This is a deliberate choice. Small museums without IT staff should not need to run a web server, manage SSL certificates, or depend on a cloud subscription. Staff install MCIS like any other application on their Windows or Linux workstations. The database runs on one machine the museum already owns.

| Component | Technology | What It Means for Your Institution |
| :--- | :--- | :--- |
| Desktop client | Python / PySide6 | Runs on Windows, Linux, and macOS. Free to distribute. Staff install it and use it — no browser required. |
| Database | PostgreSQL | A mature, professional-grade database. Runs on existing hardware. Concurrent access by multiple staff simultaneously. Free and open source. |
| IA Publishing | Internet Archive API | The official Internet Archive tool for metadata and file upload — ensures compatibility with IA's current and future API. |
| Packaging | PyInstaller | Staff install MCIS like any other desktop application. No Python installation or technical knowledge required on their machines. |

**Single-user option:** A SQLite fallback is planned for very small institutions with only one user — in this mode, no separate database server is required at all. The database lives in a single file alongside the application.

**Data ownership:** All data lives in a PostgreSQL database on hardware the museum controls. Export the full database at any time in standard SQL format. There is no cloud sync, no vendor account required, and no data stored outside the museum's own infrastructure.

---

## 7. Phased Development Plan

Development proceeds in phases. Each phase produces a testable, usable milestone — a pilot institution can begin using MCIS from Phase 1 onward and benefit from each subsequent phase without waiting for the full roadmap to complete.

### Phase 0 — Project Infrastructure *(In Progress)*

*Establish the repository, documentation, and schema design before any application code is written.*

- GitHub repository created, README and project documentation published
- Full database schema designed and documented for Phase 1 (Collections, Objects, Media, Users, Audit) and Phase 2 (Locations, Loans, Donors)
- A collection of use cases documenting operational workflows for every module
- Advisory board [TBD] — formation underway

*Milestone: Schema documented and reviewed, repository public, use cases complete.*

### Phase 1 — Core Framework + Basic IA Publishing

*A working collections database with Internet Archive publishing from day one.*

- Application shell: database connection, user login, role-based access control
- Collections module: create and manage named collections with policies and finding aids
- Objects module: full accession record with image and document attachment
- Audit log: complete change history with user attribution and before/after field values
- **Basic IA Publisher:** Registrar flags object for publication; Admin approves; system uploads object record with primary image and core metadata (title, maker, date, description, rights statement) to the museum's IA collection; IA identifier written back to the object record; publication status tracked per object
- IA configuration: IA account credentials and target collection identifier stored in application settings

*Milestone: A registrar can log in, accession objects with images, and publish an approved record to Internet Archive.*

### Phase 2 — Collections Modules + Full IA Publisher

*Full operational capability and a complete Internet Archive publishing workflow.*

- Locations: hierarchical location tracking, movement history, off-site locations
- Loans: outgoing and incoming loan agreements, condition-out/in, insurance, overdue alerts
- Donors: records linked to donated objects, gift restrictions, acknowledgment letters
- **Full IA Publisher:** configurable field mapping UI; batch publish queue management and dashboard; update and unpublish support; multi-image upload to IA
- Cross-module search and reporting
- Packaged installer for Windows, Linux, and macOS

*Milestone: A registrar can record a multi-object loan, track location history, generate a donor acknowledgment letter, and manage the full IA publish queue.*

### Phase 3 — Operations Modules

*Expand beyond collections into broader museum administration.*

- Members / Comms: membership tracking, renewals, mailing lists
- Inventory: equipment and supplies, maintenance schedules

*Milestone: Member renewals can be tracked and a lapsed-member contact list exported.*

### Phase 4 — Community & Sustainability

*Grow MCIS into a self-sustaining open source project.*

- Public documentation website
- Demo database with a realistic fictional museum for evaluation and training
- Contributor onboarding guide and community governance model
- An extension framework allowing other developers to contribute new modules

*Milestone: Documentation site live, demo database available for download, at least one external contributor.*

### Phase 5 — Data Migration

*Help institutions that already have data bring it in.*

- CSV/Excel import with configurable field mapping
- LIDO and other standard interchange format support
- Batch image import with automatic object association

*Milestone: A museum can import a full spreadsheet or PastPerfect data export and review the results in MCIS.*

---

## 8. Current Status

MCIS is in **Phase 0 — Project Infrastructure**.

| Deliverable | Status |
| :--- | :--- |
| GitHub repository | Complete |
| README and project documentation | Complete |
| Phase 1 database schema | Complete |
| Phase 2 database schema | Complete |
| Use case library (42 use cases) | Complete |
| Open source license | In progress |
| CI pipeline configuration | Pending |
| Advisory board [TBD] engagement | In progress |
| Phase 1 application development | Not yet started |

The schema and use case work is complete. The next step is beginning Phase 1 application development, with advisory board [TBD] review of the schema documents happening in parallel.

---

## 9. Competitive Landscape

| System | Cost | Target Size | Open Source | IA Integration |
| :--- | :--- | :--- | :--- | :--- |
| PastPerfect (Desktop & Online) | Free–Subscription | Small–Mid | No | No |
| Argus | Enterprise pricing | Mid–Large | No | No |
| The Museum System (TMS) | Enterprise pricing | Large | No | No |
| CollectiveAccess | Free | Small–Large | Yes | No |
| MCIS *(in development)* | Free | Small | Yes | Yes — Phase 1 |

CollectiveAccess is the closest open source peer — a capable, mature system with a strong community. It is, however, a web application requiring a PHP web server, database server, and ongoing server administration. For a small historical society with no IT staff, that deployment burden is often prohibitive regardless of software cost. MCIS offers a simpler path: download the installer, point it at the database, and begin cataloguing — with Internet Archive publishing available from the first release.

---

## 10. Risks & Mitigations

| Risk | Mitigation |
| :--- | :--- |
| Single developer | Clear documentation, modular architecture, and open source community building reduce dependency on any one person. Advisory board [TBD] support strengthens the sustainability and grant case. |
| Adoption by target institutions | Pilot museum programme; realistic demo database for evaluation; advisory board [TBD] endorsement lends credibility to early adopters |
| Schema decisions diverge from practice | Advisory board [TBD] review before Phase 1 development begins; use case library documents the expected behaviour at every step |
| PostgreSQL complexity for non-technical staff | Docker Compose deployment option; step-by-step setup guide; SQLite single-user fallback |
| Internet Archive API changes | The publishing component is designed to be updated independently if the Internet Archive API changes; pin to a stable, tested version. |
| Scope creep | Strict module boundaries and phased roadmap |
| Data loss | Deployment guide includes automated daily backup schedules and off-site backup instructions; aligns with IMLS data stewardship guidance |
| Donor and member PII | Encrypted database connections required; deployment guide covers encryption-at-rest options and data retention policy |

---

## 11. Glossary

| Term | Definition |
| :--- | :--- |
| AAM | American Alliance of Museums — the primary professional association for museums in the United States. |
| GLAM | Galleries, Libraries, Archives, and Museums — the cultural heritage sector collectively. |
| IA | Internet Archive (archive.org) — a non-profit digital library offering free, permanent storage and public access. |
| IMLS | Institute of Museum and Library Services — the primary federal grant-making agency for US museums and libraries. |
| LIDO | Lightweight Information Describing Objects — an XML schema for museum object data exchange and aggregation. |
| PII | Personally Identifiable Information — data that can identify an individual (name, address, contact details). |
| SPECTRUM | Collections management standard published by Collections Trust (UK) — widely adopted internationally as a benchmark for collections management procedures. |
| TMS | The Museum System — a commercial collections management platform by Gallery Systems (now Axiell). |
---

_2026-06-16-1417_
