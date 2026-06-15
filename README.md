# MCIS - Museum Collections Information System

_High Level Project Plan_

## 1. Project Overview
MCIS is a free, open source museum collections information system designed specifically for small, budget-constrained museums, historical societies, and specialized collections. It provides professional-grade collections management without the cost or complexity of commercial solutions such as PastPerfect, Argus, or The Museum System (TMS).

The system is built on a two-tier public/private architecture:

*   **Public Tier** — Selected collection records and artifact images are published to the museum's Internet Archive (IA) collections, providing free, permanent, searchable public access aligned with the GLAM (Galleries, Libraries, Archives, Museums) open access movement.
*   **Private Tier** — Full operational data is maintained in a secure, self-hosted SQL database covering collections management, donor records, loans, member communications, and internal inventory.

## 2. Goals & Guiding Principles

| Principle                 | Description                                                                     |
| :------------------------ | :------------------------------------------------------------------------------ |
| Free & Open Source        | No licensing fees. Community-developed and maintained under an open source license. |
| Affordable to Deploy      | Runs on modest hardware the museum already owns, or a low-cost VPS.           |
| Simple to Operate         | Low training curve for volunteers and part-time staff with high turnover.       |
| Multi-User Ready          | PyQt6 desktop clients connect to a shared SQL server — standard client/server architecture. |
| Modular by Design         | Each functional area is a standalone module. Museums adopt incrementally.       |
| Data Portable             | Museums own their data. No vendor lock-in. Standard SQL exports.                |
| Public Access Built In    | Internet Archive publishing is a first-class feature, not an afterthought.        |
| Grant Friendly            | Open access commitment and IMLS-aligned mission supports grant applications.    |

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
| Desktop Client    | Python / PyQt6             | Cross-platform (Windows, Linux, macOS). Proven in institutional deployments.   |
| Database Server   | PostgreSQL                 | Robust concurrent multi-user access. Free, open source, widely supported.      |
| ORM / DB Layer    | SQLAlchemy                 | Pythonic database abstraction. Supports future DB backend flexibility.         |
| IA Publishing     | internetarchive Python library | Official IA library for bulk upload and metadata management.                     |
| Packaging         | PyInstaller                | Cross-platform executable distribution for Windows and Linux.                  |

### 4.2 Deployment Model

The SQL database server runs on one dedicated machine the museum already owns (or an inexpensive VPS). Staff and volunteers install the lightweight PyQt6 client on their individual workstations. No web browser, no web server, and no complex middleware are required.

The Internet Archive publishing module runs as part of the desktop client, batching approved records and images to the museum's IA collection via the official API.

## 5. Module Architecture

MCIS is built as a modular system. Each module can be run standalone or integrated into the full MCIS shell. This allows museums to start with only what they need and expand over time.

### 5.1 Core Shell

The Core Shell is the foundation that all modules plug into. It is not optional.

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
| Location      | location_id, name, type (room/case/storage/offsite), parent_location_id, notes                                 |
| Loan          | loan_id, object_id, loan_type (in/out), borrower, start_date, end_date, insurance_value, condition_out, condition_in, agreement_ref |
| Donor         | donor_id, name, contact_info, gift_date, object_ids, restrictions, acknowledgment_sent                         |
| Member        | member_id, name, contact_info, tier, join_date, renewal_date, status                                           |
| Inventory Item | item_id, category (equipment/supply), name, vendor, purchase_date, quantity, maintenance_due                    |
| User          | user_id, name, role, hashed_password, last_login, active                                                      |

## 7. Phased Development Plan

### Phase 1 — Foundation

Establish the project infrastructure and core framework.

*   Create GitHub repository with README, LICENSE, CONTRIBUTING guide, and issue templates
*   Define database schema for Objects and Collections modules
*   Build Core Shell: DB connection, user auth, role system, module loader, navigation
*   Implement Collections module (basic CRUD)
*   Implement Objects module (basic CRUD with image attachment)
*   Unit test framework and CI pipeline

### Phase 2 — Collections Modules

Complete the core collections management feature set.

*   Locations module with object location history
*   Loans module with agreement tracking and deadline alerts
*   Donors module with acknowledgment letter generation
*   Search and reporting across Collections, Objects, Locations, Loans, Donors
*   PyInstaller packaging for Windows and Linux

### Phase 3 — Publishing

Implement the Internet Archive publishing workflow.

*   IA Publisher module: field mapping UI, publish queue, batch upload
*   Publish status tracking per Object record
*   Update and unpublish support
*   Documentation for IA collection setup and configuration

### Phase 4 — Operations Modules

Expand MCIS beyond collections into broader museum operations.

*   Members / Comms module with renewal tracking and mailing list export
*   Inventory module for equipment and supplies
*   Role-based access refinements for volunteer-level users

### Phase 5 — Community & Sustainability

Grow MCIS into a community-supported open source project.

*   Public documentation site (Read the Docs or GitHub Pages)
*   Sample/demo database with fictional museum data for evaluation
*   IMLS and foundation grant applications leveraging open access mission
*   Contributor onboarding and community governance model
*   Plugin/extension API for community-contributed modules

## 8. Competitive Landscape

| System                     | Cost                | Target Size | Open Source | IA Integration   |
| :------------------------- | :------------------ | :---------- | :---------- | :--------------- |
| PastPerfect Online         | Subscription fee    | Small–Mid   | No          | No               |
| Argus                      | Enterprise pricing  | Mid–Large   | No          | No               |
| The Museum System (TMS)    | Enterprise pricing  | Large       | No          | No               |
| CollectiveAccess           | Free                | Small–Large | Yes         | No               |
| MCIS (proposed)            | Free                | Small       | Yes         | Yes — built in   |

CollectiveAccess is the closest open source peer, but it is a web-based PHP application requiring a web server — a significant deployment burden for small museums with no IT staff. MCIS targets an even simpler deployment model and makes Internet Archive publishing a first-class feature.

## 9. Risks & Mitigations

| Risk                                                           | Mitigation                                                                                                   |
| :------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------- |
| Single developer bus factor                                    | Clear documentation, open source community building, modular architecture makes contributions accessible     |
| PyQt6 licensing (GPL)                                          | Review GPL compliance for distribution; consider PyQt6 commercial license or migration to PySide6 (LGPL) if needed |
| PostgreSQL deployment complexity for non-technical museums     | Provide Docker Compose option and step-by-step setup guides; consider SQLite as a single-user fallback     |
| Internet Archive API changes                                   | Pin to stable IA library version; abstract the publish interface so backends can be swapped                |
| Scope creep                                                    | Strict module boundaries and phased roadmap keep development focused                                         |

## 10. Immediate Next Steps

*   Create GitHub repository: mcis or open-mcis
*   Draft README with project mission, architecture overview, and contributing guide
*   Choose open source license (GPL v3 or MIT — note PyQt6 GPL implications)
*   Design initial database schema for Collections and Objects modules
*   Build Core Shell skeleton: DB connection, user login, module loader
*   Identify one pilot museum willing to test early releases
