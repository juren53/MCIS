# MCIS - Museum Collections Information System

_High Level Project Plan — Draft Version 0.1 — 2026-06-14-2310_

## 1. Project Overview
MCIS will be a free, open source museum collections information system designed specifically for small, budget-constrained museums, historical societies, and specialized collections. It will provide professional-grade collections management without the cost or complexity of commercial solutions such as PastPerfect, Argus, or The Museum System (TMS).

The system will be built on a two-tier public/private architecture:

*   **Public Tier** — Selected collection records and artifact images will be published to the museum's Internet Archive (IA) collections, providing free, permanent, searchable public access aligned with the GLAM (Galleries, Libraries, Archives, Museums) open access movement.
*   **Private Tier** — Full operational data will be maintained in a secure, self-hosted SQL database covering collections management, donor records, loans, member communications, and internal inventory.

## 2. Goals & Guiding Principles

| Principle                 | Description                                                                     |
| :------------------------ | :------------------------------------------------------------------------------ |
| Free & Open Source        | No licensing fees. Community-developed and maintained under an open source license. |
| Affordable to Deploy      | Will run on modest hardware the museum already owns, or a low-cost VPS.       |
| Simple to Operate         | Low training curve for volunteers and part-time staff with high turnover.       |
| Multi-User Ready          | PySide6 desktop clients will connect to a shared SQL server — standard client/server architecture. |
| Modular by Design         | Each functional area will be a standalone module. Museums can adopt incrementally. |
| Data Portable             | Museums will own their data. No vendor lock-in. Standard SQL exports.           |
| Public Access Built In    | Internet Archive publishing will be a first-class feature, not an afterthought.   |
| Grant Friendly            | Open access commitment and IMLS-aligned mission will support grant applications. |
| Accessible by Design      | UI will meet WCAG 2.1 AA keyboard navigation and screen reader standards.       |

## 3. Target Users

*   Small local historical societies operating on volunteer labor
*   Community art museums and galleries with limited IT support
*   Specialized collections (science, natural history, transportation, military) without dedicated registrars
*   Educational institutions managing teaching collections
*   Any museum currently managing collections in Excel spreadsheets or paper records

## 4. System Architecture

### 4.1 Technology Stack

| Layer             | Technology                 | Rationale                                                                      |
| :---------------- | :------------------------- | :----------------------------------------------------------------------------- |
| Desktop Client    | Python / PySide6           | Cross-platform (Windows, Linux, macOS). PySide6 is the official Qt for Python binding (maintained by Qt Group) and is licensed under LGPL v3, which allows MCIS to be distributed under MIT or GPL without requiring the application itself to be GPL. PyQt6 (Riverbank Computing) uses a GPL/commercial dual license — distributing a GPL application is permissible, but it constrains future licensing flexibility and complicates any commercial or permissively-licensed forks. PySide6 removes that constraint at no cost in functionality. |
| Database Server   | PostgreSQL                 | Robust concurrent multi-user access. Free, open source, widely supported.      |
| ORM / DB Layer    | SQLAlchemy                 | Pythonic database abstraction. Supports future DB backend flexibility.         |
| IA Publishing     | internetarchive Python library | Official IA library for bulk upload and metadata management.                     |
| Packaging         | PyInstaller                | Cross-platform executable distribution for Windows and Linux.                  |

### 4.2 Deployment Model

The MCIS database will be able to run on a computer the organization already owns but will run in a virtual isolated environment on that machine. Staff and volunteers will be able to install lightweight PySide6 client apps on their individual workstations using standard OS-specific installation processes. No web browser, no web server, and no complex middleware will be required.

The Internet Archive publishing module will run as part of the desktop client, batching approved records and images to the museum's IA collection via the official API.

## 5. Module Architecture

MCIS will be built as a modular system. Each module will be able to run standalone or integrated into the full MCIS shell. This will allow museums to start with only what they need and expand over time.

### 5.1 Core Shell

The Core Shell will be the foundation that all modules plug into. It will not be optional.

*   Database connection configuration and connection pooling
*   User authentication and role-based access control (Admin, Registrar, Staff, Volunteer, Read-Only)
*   Module enable/disable per installation
*   Shared navigation, consistent UI patterns, and application settings
*   Audit logging of record changes

### 5.2 Collections Modules

| Module      | Key Functions                                                                                               |
| :---------- | :---------------------------------------------------------------------------------------------------------- |
| Collections | Accession records, collection hierarchy, collection-level policies and finding aids                       |
| Objects     | Individual item records, condition reports, provenance, images, conservation history, IA publish flag and status |
| Locations   | Rooms, display cases, storage areas, off-site locations, loan locations. Object location history tracking. |
| Loans       | Incoming and outgoing loan agreements, insurance requirements, condition at loan, return deadlines, borrower contacts |
| Donors      | Donor contact records, gift and bequest history, acknowledgment letter generation, restricted gift tracking |

### 5.3 Operations Modules

| Module           | Key Functions                                                                                               |
| :--------------- | :---------------------------------------------------------------------------------------------------------- |
| Members / Comms  | Membership tiers and renewals, member directory, newsletter distribution lists, event notices, lapsed member tracking |
| Inventory        | Equipment and supply records, vendor contacts, purchase history, maintenance schedules, consumable stock levels |

### 5.4 Publishing Module

| Module      | Key Functions                                                                                                 |
| :---------- | :------------------------------------------------------------------------------------------------------------ |
| IA Publisher | Publish queue management, field mapping (MCIS fields → IA metadata), batch upload to Internet Archive, update and unpublish support, publish status tracking per object |

## 6. Core Database Entities

The following primary entities form the heart of the MCIS data model. Detailed schema design is a Phase 2 deliverable.

| Entity        | Key Attributes (summary)                                                                                      |
| :------------ | :------------------------------------------------------------------------------------------------------------ |
| Collection    | collection_id, name, description, accession_policy, finding_aid                                                |
| Object        | object_id, collection_id, accession_number, title, description, date_created, medium, dimensions, condition, provenance, location_id, ia_published, ia_identifier |
| Media         | media_id, object_id, file_path, file_type, caption, is_primary, upload_date                                    |
| Location      | location_id, name, type (room/case/storage/offsite), parent_location_id, notes                                 |
| Loan          | loan_id, loan_type (in/out), borrower, start_date, end_date, insurance_value, agreement_ref. Objects linked via LoanObject junction (loan_id, object_id, condition_out, condition_in) |
| Donor         | donor_id, name, contact_info, gift_date, object_ids, restrictions, acknowledgment_sent                         |
| Member        | member_id, name, contact_info, tier, join_date, renewal_date, status                                           |
| Inventory Item | item_id, category (equipment/supply), name, vendor, purchase_date, quantity, maintenance_due                    |
| User          | user_id, name, role, hashed_password, last_login, active                                                      |
| AuditLog      | log_id, user_id, table_name, record_id, action (create/update/delete), changed_fields, timestamp              |

## 7. Phased Development Plan

### Phase 0 — Project Infrastructure

Establish repository, tooling, and schema design before coding begins.

*   Create GitHub repository with README, LICENSE, CONTRIBUTING guide, and issue templates
*   Design full database schema for all core entities
*   Configure unit test framework and CI pipeline

_Milestone: Schema documented and reviewed, repository public, CI passing on an empty test suite._

### Phase 1 — Core Framework

Build the Core Shell and the two foundational collection modules.

*   Core Shell: database connection, user authentication, role-based access control, module loader, navigation
*   Collections module (basic CRUD)
*   Objects module (basic CRUD with image and media management)

_Milestone: A staff user can log in, create a collection, add objects with images, and query them._

### Phase 2 — Collections Modules

Complete the core collections management feature set.

*   Locations module with object location history
*   Loans module with agreement tracking and deadline alerts
*   Donors module with acknowledgment letter generation
*   Search and reporting across Collections, Objects, Locations, Loans, Donors
*   PyInstaller packaging for Windows and Linux

_Milestone: A registrar can record a multi-object loan, track an object's location history, and generate a donor acknowledgment letter._

### Phase 3 — Publishing

Implement the Internet Archive publishing workflow.

*   IA Publisher module: field mapping UI, publish queue, batch upload
*   Publish status tracking per Object record
*   Update and unpublish support
*   Documentation for IA collection setup and configuration

_Milestone: An approved object record can be published to Internet Archive and its IA status is reflected in MCIS._

### Phase 4 — Operations Modules

Expand MCIS beyond collections into broader museum operations.

*   Members / Comms module with renewal tracking and mailing list export
*   Inventory module for equipment and supplies
*   Role-based access refinements for volunteer-level users

_Milestone: Member renewals can be tracked and a lapsed-member mailing list can be exported._

### Phase 5 — Community & Sustainability

Grow MCIS into a community-supported open source project.

*   Public documentation site (Read the Docs or GitHub Pages)
*   Sample/demo database with fictional museum data for evaluation
*   IMLS and foundation grant applications leveraging open access mission
*   Contributor onboarding and community governance model
*   Plugin/extension API for community-contributed modules

_Milestone: Documentation site live, demo database available for download, at least one external contributor._

### Phase 6 — Data Migration

Tools and processes for migrating existing museum data into MCIS from spreadsheets, legacy systems, or standard interchange formats.

*   CSV/Excel import utility with configurable field mapping
*   Support for common museum data standards (e.g., LIDO, CIDOC CRM) where applicable
*   Batch image import and association with object records

_Milestone: A museum can import a full spreadsheet or PastPerfect export and review the results in MCIS._

## 8. Competitive Landscape

| System                     | Cost                | Target Size | Open Source | IA Integration   |
| :------------------------- | :------------------ | :---------- | :---------- | :--------------- |
| PastPerfect Online         | Subscription fee    | Small–Mid   | No          | No               |
| Argus                      | Enterprise pricing  | Mid–Large   | No          | No               |
| The Museum System (TMS)    | Enterprise pricing  | Large       | No          | No               |
| CollectiveAccess           | Free                | Small–Large | Yes         | No               |
| MCIS (in development)      | Free                | Small       | Yes         | Yes — built in   |

CollectiveAccess is the closest open source peer, but it is a web-based PHP application requiring a web server — a significant deployment burden for small museums with no IT staff. MCIS will target an even simpler deployment model and will make Internet Archive publishing a first-class feature.

## 9. Risks & Mitigations

| Risk                                                           | Mitigation                                                                                                   |
| :------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------- |
| Single developer bus factor                                    | Clear documentation, open source community building, modular architecture makes contributions accessible     |
| Qt binding licensing                                           | PySide6 (LGPL v3) chosen over PyQt6 (GPL) — see Technology Stack rationale. No further action required unless the binding is changed. |
| PostgreSQL deployment complexity for non-technical museums     | Provide Docker Compose option and step-by-step setup guides; consider SQLite as a single-user fallback     |
| Internet Archive API changes                                   | Pin to stable IA library version; abstract the publish interface so backends can be swapped                |
| Scope creep                                                    | Strict module boundaries and phased roadmap keep development focused                                         |
| Data loss / no backup                                          | Deployment guide will include automated pg_dump schedules and off-site backup instructions; required for IMLS data stewardship compliance |
| Donor and member PII                                           | Client–server traffic must use TLS; deployment guide will cover encryption-at-rest options and a data retention policy |

## 10. Immediate Next Steps

*   ~~Create GitHub repository (juren53/MCIS)~~ — Done
*   ~~Draft README with project mission and architecture overview~~ — Done
*   Choose open source license (MIT or GPL v3 — PySide6 LGPL is compatible with both)
*   Complete full database schema design for all core entities (Phase 0 deliverable)
*   Build Core Shell skeleton: database connection, user login, module loader
*   Identify one pilot museum willing to test early releases

## 11. Glossary

| Term      | Definition                                                                                       |
| :-------- | :----------------------------------------------------------------------------------------------- |
| CIDOC CRM | Conceptual Reference Model — an ISO standard ontology for cultural heritage information.         |
| GLAM      | Galleries, Libraries, Archives, and Museums — the cultural heritage sector collectively.         |
| IA        | Internet Archive (archive.org) — a non-profit digital library offering free permanent storage.   |
| IMLS      | Institute of Museum and Library Services — the primary federal grant-making agency for US museums. |
| LIDO      | Lightweight Information Describing Objects — an XML schema for museum object data exchange.      |
| ORM       | Object-Relational Mapper — a library that maps database rows to programming language objects.    |
| PII       | Personally Identifiable Information — data that can identify an individual (name, address, etc.). |
| TMS       | The Museum System — a commercial collections management software by Gallery Systems.             |
| VPS       | Virtual Private Server — a rented virtual machine hosted in a cloud data center.                 |
| WCAG      | Web Content Accessibility Guidelines — accessibility standards published by the W3C.             |

---

_2026-06-16-1101_
