# MCOS — Museum Collections & Operations System

_High Level Project Plan — Draft Version 0.22 — 2026-06-21-1703_

---

## 1. Project Overview

MCOS is a free, open source museum collections information system designed specifically for small, budget-constrained museums, historical societies, and specialized collections. It will deliver professional-grade collections management without the licensing costs or technical complexity of commercial platforms such as PastPerfect, Argus, or The Museum System (TMS).

**The problem MCOS addresses:** Hundreds of small museums, historical societies, and community archives currently manage their collections in Excel spreadsheets, paper registers, or aging standalone software. These institutions cannot afford commercial solutions — many cost thousands of dollars per year in subscription fees — and lack the IT staff to deploy and maintain web-based alternatives. Their collections remain undiscoverable, their data is at risk, and their staff and volunteers face unnecessary administrative burden.

**The MCOS approach:** A free, easy-to-install desktop application that any registrar, staff member, or trained volunteer can learn quickly, backed by a shared database that all workstations connect to simultaneously, with Internet Archive publishing built in from the start.

The system is built on a two-tier public/private architecture:

- **Private Tier** — Full operational data is maintained in a secure, self-hosted database covering collections management, donor records, loans, location tracking, and internal inventory. Data stays on hardware the museum controls — no cloud dependency, no subscription, no vendor lock-in.
- **Public Tier** — Selected collection records and artifact images can be published to the museum's Internet Archive (IA) collection, providing free, permanent, searchable public access aligned with the GLAM (Galleries, Libraries, Archives, Museums) open access movement.

MCOS is not a 'greenfield' project. It builds directly on production metadata tools developed for the Harry S. Truman Presidential Library — tools that have already solved problems MCOS will face: archival metadata schema design, IPTC/EXIF embedding in image files, batch processing pipelines, and field mapping to public publishing targets. See §11 for detail.

---

## 2. Goals & Guiding Principles

| Principle | Description |
| :--- | :--- |
| Free & Open Source | No licensing fees, ever. Community-developed and maintained. |
| Affordable to Deploy | Runs on hardware the museum already owns, or an inexpensive server. No annual subscription. |
| Simple to Operate | Intuitive and easy to use for staff and volunteers experienced in museum and archive work. A low learning curve is a first-order design requirement — not a nice-to-have. |
| Multi-User Ready | Multiple staff and volunteers can work simultaneously from their own workstations, connecting to a shared database. |
| Modular by Design | Museums adopt only the modules they need. Start with Collections and Objects; add Members, Donors, Locations, and Loans as capacity and interest grow. |
| Data Portable | Museums own their data in a standard, open format. Full import and export of data at any time. No vendor lock-in. |
| Public Access Built In | Internet Archive publishing is a core feature, available from Phase 1. Publishing an approved collection record to the public is a one-step workflow, not a separate project. |
| Standards Aligned | Field names, controlled vocabularies, and workflows reflect professional collections management standards (see §5). |
| Grant Friendly | Open access commitment and IMLS-aligned mission support grant applications from IMLS, NEH, state humanities councils, and private foundations. |
| Accessible by Design | Designed for users of varying technical ability; interface will be consistent, intuitive, and operable without specialist training. |

---

## 3. Target Institutions

MCOS is designed for institutions that are professionally committed to collections stewardship but cannot afford, or choose not to use, commercial collections management software:

- Small local historical societies operating on volunteer labor
- Community art museums and galleries with limited IT support
- Specialized collections (science, natural history, transportation, military, ethnic heritage) without dedicated registrars
- Educational institutions managing teaching or study collections
- Archives and special collections within libraries, universities, or cultural centers
- Any institution currently managing collections in Excel spreadsheets or paper records

**What these institutions have in common:** They take their collections seriously. Their registrars and volunteers understand accession numbers, provenance, condition reports, and donor acknowledgment. What they lack is affordable, professional-grade software that matches how they actually work.

---

## 4. What MCOS Will Do

MCOS is organized into functional modules. Each module integrates into the full MCOS application; smaller institutions can enable only the modules they need.

### Collections, Objects & Internet Archive Publishing

The foundation of any collections management system — with the feature that sets MCOS apart built in from the first release: direct Internet Archive publishing.

- Create and manage named collections with accession policies and finding aids (free text or URL)
- Accession individual objects with full cataloging fields: title, maker, date made, medium, dimensions, provenance, credit line, rights statement, condition grade, condition notes, and structured condition assessment date
- Attach multiple images and documents (JPEG, TIFF, PNG, PDF) per object, with primary image designation, sort order, and IPTC/EXIF metadata embedding via ExifTool (and/or Exiv2) — ensuring attached images carry accession number, rights statement, and credit information within the file itself
- **Photo Ingest Module** — An interface layer between existing, proven scanning software and Collections rather than a scanner driver of its own. As images (JPEG, TIFF, PNG, PDF) are scanned individually or in batches, the module ingests them, checks for and reports on existing metadata, and facilitates review, correction, and attachment to objects as the user or institution deems necessary. It integrates the pre-existing Tag Writer photo-metadata functions (IPTC/XMP/EXIF editing via the bundled ExifTool and/or Exiv2) — tools already in production use at the Truman Library — so scanned images are inspected and corrected in the same HSTL field set used throughout MCOS, ensuring each attached file carries accession number, rights statement, credit, and collection within the file itself before it is committed to the catalog
- **Audio Ingest Module** — The audio counterpart to the Photo Ingest Module: an interface layer between existing, proven audio digitization and transfer software and Collections rather than a capture tool of its own. As recordings (MP3, WAV) are transferred individually or in batches, the module ingests them, checks for and reports on existing metadata, and facilitates review, correction, and attachment to objects as the user or institution deems necessary. It integrates the pre-existing Audio Tag Writer (ATW) functions (the HSTL ID3v2.3 archival frame set, with cross-schema aliases for iTunes, XMP, and Windows Media) — modules already functioning in production at the Truman Library — so transferred recordings are inspected and corrected in the same archival field set used throughout MCOS, ensuring each attached file carries accession number, speakers, date recorded, rights/restrictions, and collection within the file itself before it is committed to the catalog
- Field-specific and full-text search across the entire catalogue
- Role-based access control: Admin, Registrar, Staff, Volunteer, and Read-Only — each role has appropriate capabilities
- Complete audit trail of every record creation, edit, and deletion, with user attribution and before/after field values
- **Basic IA Publisher** — Registrar flags objects for publication; Admin approves; the system uploads the object record with its primary image and core metadata (title, maker, date, description, rights statement) to the museum's Internet Archive collection; IA identifier is written back to the object record; publication status is visible on every object
- **QR Code Generator** *(standalone module)* — Once an IA identifier is assigned, generates a printable QR code label (PNG or PDF) for display beside the artifact in the museum; visitors scan with a smartphone camera and reach the artifact's IA page directly; see [github.com/juren53/qr-code-generator](https://github.com/juren53/qr-code-generator)

### Members, Donors, Locations & Loans

The operational core for a working museum or historical society, paired with a full-featured IA publisher.

- **Members / Comms** — Membership tiers, renewal tracking, lapsed-member outreach lists, newsletter distribution
- **Donors** — Individual and organization donor records; link donated objects to their donors; gift restrictions; acknowledgment letter generation; anonymous donor flag with public suppression; deactivation for deceased or lapsed donors
- **Locations** — Hierarchical location tree (building → room → case → shelf); record where every object is; full movement history; off-site and loan-destination locations
- **Loans** — Outgoing and incoming loan agreements with multi-object support; condition-out / condition-in recording; insurance value and currency; return deadlines and overdue alerts; loan status (Pending, Active, Returned, Cancelled)
- **Full IA Publisher** — Configurable field mapping UI (MCOS fields → IA metadata); batch publish queue management and dashboard; update and unpublish support; multi-image upload

### Operations Modules

*Additional operations modules may be scoped here as requirements emerge.*

- **Inventory** — Equipment and supply records, vendor contacts, maintenance schedules

### Data Migration

- CSV/Excel import with configurable field mapping — drawing on batch processing and Unicode handling experience from the HSTL photo and audio metadata pipelines
- LIDO and other standard interchange format support
- Batch image import with IPTC metadata extraction and automatic object association

---

## 5. Standards Alignment

MCOS field names and controlled vocabularies are informed by established collections management standards. Alignment with these standards improves data interoperability, satisfies grant reporting requirements, and means that museums using MCOS are building records that are exportable into future systems.

| Standard | How MCOS Aligns |
| :--- | :--- |
| **Dublin Core** | Core descriptive fields — title, maker (creator), date made, description, rights statement — map directly to Dublin Core elements; used as the basis for Internet Archive metadata export |
| **IPTC** (International Press Telecommunications Council) | Image metadata fields — headline (title), caption (description), object name (accession number), byline (photographer), credit, source (collection), copyright notice (restrictions) — are embedded directly in attached image files using ExifTool (and/or Exiv2); field mapping proven in the HSTL photo pipeline and Tag Writer |
| **SPECTRUM** (Collections Trust, UK) | Object accession, object entry, location and movement control, loans in/out, condition checking, and the audit trail align with SPECTRUM unit of practice definitions; the workflow design in the use case library explicitly references SPECTRUM procedures |
| **LIDO** (Lightweight Information Describing Objects) | Phase 4 data migration tools will support LIDO XML import/export for interoperability with digital aggregators |
| **AAM Standards** | Object records include the fields required by the *National Standards and Best Practices for U.S. Museums* (American Alliance of Museums, 2008): unique accession number, title, provenance, rights statement, and condition |
| **IMLS Data Stewardship** | Audit logging, data export, and backup guidance align with IMLS data stewardship expectations for grantees; the rights statement field is required before any object can be published publicly |

**Note:** MCOS does not claim certification under any of these standards. The alignment documented here reflects best-effort conformance at the schema and workflow level, and will be refined through the advisory board [TBD] review process.

### Grant Alignment

The standards work above is one half of the grant case; the other half is how MCOS maps to what funders actually evaluate. Grant reviewers at IMLS, NEH, state humanities councils, and private foundations consistently weigh public access, rights management, data stewardship, interoperability, and sustainability. MCOS is designed so a small institution can answer each of these with a concrete feature rather than a promise.

| Funder Priority | How MCOS Supports It |
| :--- | :--- |
| **Public access & dissemination** | Internet Archive publishing is built in from Phase 1 — publishing an approved record is a one-step workflow, not a separate grant deliverable. The Truman Home Music Collection (§11) is a live proof of the end-to-end workflow producing free, permanent public access. |
| **Rights & permissions** | A rights statement is required before any object can be published publicly, so digitization funded by a grant cannot inadvertently publish material without clear rights. |
| **Data stewardship** | Complete audit trail with user attribution and before/after values, full database export in standard SQL, and deployment guidance for automated and off-site backups (§10) align with IMLS data stewardship expectations for grantees. |
| **Interoperability & open formats** | Records align with Dublin Core, IPTC, SPECTRUM, LIDO, and AAM standards and export in non-proprietary formats — satisfying funder preferences for portable, reusable outputs over vendor-locked deliverables. |
| **Sustainability** | Free and open source, running on hardware the institution already owns, with no subscription — the project demonstrably continues beyond the grant period without recurring cost. |
| **Mission alignment** | The open access, GLAM-aligned mission supports collections access and community-anchor goals common to IMLS and NEH humanities programs. |

**Note:** Sustainability beyond a single developer is the open question a reviewer is most likely to probe; the advisory board [TBD] and community governance model (Phase 5) are the planned mitigations (§10).

---

## 6. Technology Overview

MCOS is a desktop application — not a web application. This is a deliberate choice. Small museums without IT staff should not need to run a web server, manage SSL certificates, or depend on a cloud subscription. Staff install MCOS like any other application on their Windows, Linux, or macOS workstations. The database runs on one machine the museum already owns.

| Component | Technology | What It Means for Your Institution |
| :--- | :--- | :--- |
| Desktop client | Python / PySide6 | Runs on Windows, Linux, and macOS. Free to distribute. Staff install it and use it — no browser required. |
| Database | PostgreSQL (multi-user) / SQLite (single-user) | Multi-user: PostgreSQL runs on existing hardware with concurrent access for multiple staff simultaneously. Single-user option: SQLite — no separate database server required; the database lives in a single file alongside the application. Both are free and open source. |
| IA Publishing | Internet Archive API | The official Internet Archive tool for metadata and file upload — ensures compatibility with IA's current and future API. |
| Image Metadata | ExifTool (and/or Exiv2) | Embeds IPTC/EXIF metadata directly into attached image files at ingest — accession number, rights statement, credit, and collection. Ensures images carry provenance data within the file regardless of where they are copied or published. Bundled with the application; no separate installation required. |
| Packaging | PyInstaller | Staff install MCOS like any other desktop application. No Python installation or technical knowledge required on their machines. |

**Data ownership:** All data lives in a PostgreSQL database on hardware the museum controls. Export the full database at any time in standard SQL format. There is no cloud sync, no vendor account required, and no data stored outside the museum's own infrastructure.

---

## 7. Phased Development Plan

Development proceeds in phases. Each phase produces a testable, usable milestone — a pilot institution can begin using MCOS from Phase 1 onward and benefit from each subsequent phase without waiting for the full roadmap to complete.

### Phase 0 — Project Infrastructure *(Complete)*

*Establish the repository, documentation, and schema design before any application code is written.*

- GitHub repository created, README and project documentation published
- Full database schema designed and documented for Phase 1 (Collections, Objects, Media, Users, Audit) and Phase 2 (Members, Donors, Locations, Loans)
- A collection of use cases documenting operational workflows for every module

*Milestone reached: Schema documented and reviewed, repository public, use cases complete.*

### Phase 1 — Core Framework + Basic IA Publishing

*A working collections database with Internet Archive publishing from day one.*

- Application shell: database connection, user login, role-based access control
- Collections module: create and manage named collections with policies and finding aids
- Objects module: full accession record with image and document attachment
- Audit log: complete change history with user attribution and before/after field values
- **Basic IA Publisher:** Registrar flags object for publication; Admin approves; system uploads object record with primary image and core metadata (title, maker, date, description, rights statement) to the museum's IA collection; IA identifier written back to the object record; publication status tracked per object
- **QR Code Generator** *(standalone module)* — Once an IA identifier is assigned, generate a printable QR code label (PNG or PDF) for display beside the artifact in the museum; see [github.com/juren53/qr-code-generator](https://github.com/juren53/qr-code-generator)

*Milestone: A registrar can log in, accession objects with images, publish an approved record to Internet Archive, and print a QR code label for display beside the artifact.*

### Phase 2 — Operational Modules

*Full operational capability and a complete Internet Archive publishing workflow.*

- Members / Comms: membership tiers, renewal tracking, lapsed-member outreach lists, newsletter distribution
- Donors: records linked to donated objects, gift restrictions, acknowledgment letters
- Locations: hierarchical location tracking, movement history, off-site locations
- Loans: outgoing and incoming loan agreements, condition-out/in, insurance, overdue alerts
- **Full IA Publisher:** configurable field mapping UI; batch publish queue management and dashboard; update and unpublish support; multi-image upload to IA
- Cross-module search and reporting
- Packaged installer for Windows, Linux, and macOS

*Note: Members/Comms schema to be designed before Phase 2 development begins.*

*Milestone: A registrar can record a multi-object loan, track location history, generate a donor acknowledgment letter, and manage the full IA publish queue.*

### Phase 3 — Operations Modules

*Expand beyond collections into broader museum administration. Additional operations modules may be scoped here as requirements emerge.*

- Inventory: equipment and supplies, maintenance schedules

*Milestone: Equipment and supply records are tracked and a maintenance schedule can be produced.*

### Phase 4 — Data Migration

*Make it easy to import pre-existing data from a variety of sources.*

- CSV/Excel import with configurable field mapping — drawing on batch processing and Unicode handling experience from the HSTL photo and audio metadata pipelines
- LIDO and other standard interchange format support
- Batch image import with IPTC metadata extraction and automatic object association

*Milestone: A museum can import a full spreadsheet or PastPerfect data export and review the results in MCOS.*

### Phase 5 — Community & Sustainability

*Grow MCOS into a self-sustaining open source project.*

- Public documentation website
- Demo database with a realistic fictional museum for evaluation and training
- Contributor onboarding guide and community governance model
- An extension framework allowing other developers to contribute new modules

*Milestone: Documentation site live, demo database available for download, at least one external contributor.*

---

## 8. Current Status

MCOS is in **Phase 0 — Project Infrastructure**.

| Deliverable | Status |
| :--- | :--- |
| GitHub repository | Complete |
| README and project documentation | Complete |
| Phase 1 database schema | Complete |
| Phase 2 database schema (Locations, Loans, Donors) | Complete |
| Phase 2 database schema (Members/Comms) | Not yet started |
| Use case library | Complete |
| QR Code Generator module repository | Complete |
| Open source license | In progress |
| Phase 1 application development | Not yet started |

The schema and use case work is complete. The next step is beginning Phase 1 application development, with advisory board [TBD] review of the schema documents planned for the same period.

**Project Repository:** MCOS is publicly hosted at [https://github.com/juren53/MCOS](https://github.com/juren53/MCOS). GitHub is a widely used platform for open source software development. For MCOS it serves several purposes: it stores all project files with a complete history of every change; it provides a public issue tracker where museum professionals and developers can report problems, request features, and discuss design questions; and it makes the codebase freely accessible to anyone who wants to contribute. No GitHub account is required to browse the repository or read the documentation — an account is only needed to open an issue or submit code.

---

## 9. Competitive Landscape

| System | Cost | Target Size | Open Source | IA Integration |
| :--- | :--- | :--- | :--- | :--- |
| PastPerfect (Desktop & Online) | Free–Subscription | Small–Mid | No | No |
| Argus | Enterprise pricing | Mid–Large | No | No |
| The Museum System (TMS) | Enterprise pricing | Large | No | No |
| CollectiveAccess | Free | Small–Large | Yes | No |
| MCOS *(in development)* | Free | Small | Yes | Yes — Phase 1 |

CollectiveAccess is the closest open source peer — a capable, mature system with a strong community. It is, however, a web application requiring a PHP web server, database server, and ongoing server administration. For a small historical society with no IT staff, that deployment burden is often prohibitive regardless of software cost. MCOS offers a simpler path: download the installer, point it at the database, and begin cataloguing — with Internet Archive publishing available from the first release.

---

## 10. Risks & Mitigations

| Risk | Mitigation |
| :--- | :--- |
| Single developer | Clear documentation, modular architecture, and open source community building reduce dependency on any one person. Advisory board [TBD], once formed, will strengthen the sustainability and grant case. |
| Adoption by target institutions | Pilot museum programme; realistic demo database for evaluation; advisory board [TBD] endorsement, once formed, will lend credibility to early adopters |
| Schema decisions diverge from practice | Advisory board [TBD] review before Phase 1 development begins; use case library documents the expected behavior at every step |
| PostgreSQL complexity for non-technical staff | Guided installation wizard (planned); step-by-step setup guide; single-file database option for institutions with only one user |
| Internet Archive API changes | The publishing component is designed to be updated independently if the Internet Archive API changes; pin to a stable, tested version. |
| Scope creep | Strict module boundaries and phased roadmap |
| Data loss | Deployment guide includes automated daily backup schedules and off-site backup instructions; aligns with IMLS data stewardship guidance |
| Donor and member PII | Encrypted database connections required; deployment guide covers encryption-at-rest options and data retention policy |
| Data security | Sensitive data is handled exclusively through well-audited Python security libraries — `cryptography` for encryption, `bcrypt` or `argon2-cffi` for password hashing, and the standard `ssl` module for encrypted database connections — avoiding custom cryptographic code; deployment guide covers encryption-at-rest configuration for both PostgreSQL and SQLite deployments |

---

## 11. Prior Work & Related Projects

MCOS grows from production tools and prototypes built for the Harry S. Truman Presidential Library (HSTL) and Weston Historical Museum (WHM). Each project addresses a problem MCOS will face — and each represents working code, field-tested metadata schemas, and proven workflows that can be adapted rather than rebuilt from scratch.

Image and audio files processed at the Truman Library using the HPM, Tag Writer, and Audio Tag Writer (ATW) code have been uploaded to and validated in the U.S. National Archives (NARA) Catalog, and are currently available online — concrete proof that these metadata schemas and tools produce records accepted and published by a national repository, not just in a developer's test environment.

### HPM — HSTL Photo Metadata Framework

A Python/PyQt6 desktop application that orchestrates an 8-step batch pipeline: Excel metadata spreadsheet → CSV conversion → Unicode cleanup → TIFF processing → IPTC/EXIF metadata embedding (via ExifTool and/or Exiv2) → JPEG conversion → resizing → watermarking for restricted images. The pipeline prepares photos for upload to the NARA catalog. HPM is packaged with PyInstaller, ships as a single Windows executable, and has a full acceptance test suite.

**MCOS relevance:** The ExifTool (and/or Exiv2) integration, batch image processing pipeline, IPTC field mapping, and PyInstaller packaging approach are all directly applicable to the MCOS Objects module and the Phase 5 Data Migration tools. The HPM pattern of ingesting a CSV export from an existing spreadsheet or database and mapping its columns to a defined metadata schema is the direct model for MCOS's CSV data import feature — allowing institutions to migrate records from Excel, PastPerfect exports, or other legacy systems by mapping their existing field names to the MCOS data structure at import time.

**Repository:** [github.com/juren53/HST-Metadata](https://github.com/juren53/HST-Metadata)

### Tag Writer

A Python/PyQt6 image metadata editor for cases where batch processing is not applicable — a registrar or archivist needs to view or correct the metadata on a single image. Tag Writer edits IPTC/XMP/EXIF fields with a form-based UI, includes a full-size image viewer, bundles ExifTool (and/or Exiv2) for fast image paging, and supports multiple color themes and JSON export/import.

The HSTL IPTC field set Tag Writer implements — headline (title), caption-abstract (description), object name (accession number), byline (photographer), credit, source (collection), copyright notice (restrictions) — is the direct basis for the MCOS object metadata schema and its mapping to Internet Archive metadata fields.

**MCOS relevance:** The IPTC field set and its Dublin Core alignment directly inform the MCOS Objects module schema and the IA metadata export. The single-record editing workflow is the model for the MCOS object record form.

**Repository:** [github.com/juren53/tag-writer](https://github.com/juren53/tag-writer)

### Audio Tag Writer (ATW)

A Python/PyQt6 audio metadata editor with three modes: Archival Recording, Music, and Scientific. The Archival Recording mode writes the complete HSTL ID3v2.3 frame set — title, description, accession number, speakers, date recorded, restrictions, location, production/copyright, collection, source URL — with cross-schema aliases for iTunes, XMP, and Windows Media compatibility. ATW supports MP3 and WAV read/write and is packaged as a Windows executable.

**MCOS relevance:** The HSTL archival metadata profile demonstrates a complete, production-validated archival schema. The accession-number-centric structure, rights/restrictions handling, and collection hierarchy directly parallel the MCOS object and media attachment models. The JSON export/import pattern is applicable to MCOS data portability requirements.

**Repository:** [github.com/juren53/audio-tag-writer](https://github.com/juren53/audio-tag-writer)

### Truman Home Music Collection

A proof-of-concept project managing the musical holdings at the Harry S. Truman National Historic Site (U.S. National Park Service). Phase 1 is complete — a selection of items from the collection is publicly accessible at [archive.org/details/trumanhomemusic](https://archive.org/details/trumanhomemusic). The code and scripts are prototype-quality rather than production-ready.

**MCOS relevance:** This project is the direct proof of concept for MCOS's core Internet Archive publishing model — a real cultural heritage collection, live on IA, published by the same developer. Where HPM, Tag Writer, and ATW prove out the metadata and tooling side, the Truman Home Music Collection proves out the end-to-end IA publishing workflow: collection setup, item upload, metadata mapping, and public access. MCOS formalizes and generalizes this workflow for any small museum.

**Repository:** [github.com/juren53/TrumanMusicCollection](https://github.com/juren53/TrumanMusicCollection)

**Live collection:** [archive.org/details/trumanhomemusic](https://archive.org/details/trumanhomemusic)

### QR Code Generator

A prototype QR code module provides contextual information for artifacts currently on display with no information label. Six QR codes were generated and linked to placeholder records uploaded to the Internet Archive — including a steamboat bell and anchor bracket and a steamboat drum alarm clock — demonstrating the end-to-end workflow: upload artifact record to IA, generate a QR code, print and display it beside the artifact, visitor scans with a smartphone camera and reaches the artifact's IA page directly.

Testing confirmed that printed QR codes scan measurably faster than codes displayed on screen, and that codes as small as 2 cm (~0.75 inches) scan successfully. Key trade-offs identified for the Museum environment include QR code size, lighting conditions, and visitor phone hardware. The prototype established that the technology is mature and implementation at WHM is very doable with relatively little friction.

**MCOS relevance:** This is the direct prototype for the MCOS QR Code Generator module. It proves out the full visitor-facing workflow — IA record → QR code → printed label → artifact display — and surfaces the practical questions (label size, display format, artifact numbering) that the module must resolve. The prototype has been formalized as a standalone MCOS module.

**Repository:** [github.com/juren53/qr-code-generator](https://github.com/juren53/qr-code-generator)

**Placeholder IA records:** [archive.org/details/@juren?query=Weston](https://archive.org/details/@juren?query=Weston)

---

## 12. Implementing MCOS

MCOS is designed to be as accessible as possible for small institutions with limited technical resources, but a realistic picture of what implementation actually involves is important. Many of the standalone modules — the QR Code Generator, Tag Writer, and Audio Tag Writer — can be downloaded, installed, and running within minutes by following the step-by-step documentation in each module's GitHub repository. These tools are self-contained and require no database setup or network configuration. The simpler single-user MCOS configuration, where the entire database lives in a single file on one workstation using SQLite, is also fully self-contained and requires no separate database server — making it accessible even for institutions with minimal technical support.

A full, multi-user MCOS deployment — one where a shared PostgreSQL database serves multiple staff and volunteer workstations simultaneously — involves additional steps that will require some IT familiarity or outside support. Installing and configuring a PostgreSQL database server, setting up user accounts and role-based access, and connecting workstations across a local network are well-documented tasks, but they are not click-and-go operations for users without prior experience. Institutions without in-house IT capability should expect to involve a technically experienced volunteer, a local IT professional, or a peer institution with relevant experience for initial setup. MCOS will provide a guided installation wizard, detailed setup documentation, and worked configuration examples for both deployment paths to make that process as straightforward as possible. Everything required to install, configure, and run MCOS and its modules — installation packages, schema files, setup guides, a demo database with realistic fictional data, and a community issue tracker for questions — will be freely available on the MCOS GitHub repository, with the goal to make it easy and straightforward to get up and running.

---

## 13. Glossary

| Term | Definition |
| :--- | :--- |
| AAM | American Alliance of Museums — the primary professional association for museums in the United States, founded 1906. Publishes the Core Standards for Museums, which defines the minimum object documentation fields — unique accession number, title, provenance, rights statement, and condition record — that MCOS implements on every object record. |
| Dublin Core | A widely adopted standard for describing digital resources — fifteen core elements including title, creator, date, description, and rights. Used as the basis for Internet Archive metadata. |
| GLAM | Galleries, Libraries, Archives, and Museums — the cultural heritage sector collectively. |
| IA | Internet Archive (archive.org) — a non-profit digital library offering free, permanent storage and public access. |
| IMLS | Institute of Museum and Library Services — the primary federal grant-making agency for US museums and libraries. |
| LIDO | Lightweight Information Describing Objects — an XML schema for museum object data exchange and aggregation. |
| PII | Personally Identifiable Information — data that can identify an individual (name, address, contact details). |
| SPECTRUM | Collections management standard published by Collections Trust (UK), currently at version 5.1 (September 2022). Defines 21 procedures covering the full range of collections management activity — from object entry and accessioning through loans, location control, condition checking, rights management, and deaccessioning. Required standard for UK Arts Council Accreditation; widely adopted internationally as a benchmark for professional collections stewardship. |
| TMS | The Museum System — a commercial collections management platform by Gallery Systems (now Axiell). |
---

_2026-06-17-1725_
