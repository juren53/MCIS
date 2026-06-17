# MCIS Documentation Plan

_Draft Version 0.7 — 2026-06-17-1300_

This document tracks all MCIS documentation — completed, in progress, and planned. Each document is assigned a target phase, a primary audience, and a brief description of its scope.

---

## Status Summary

| Status | Count |
| :--- | :--- |
| Complete | 8 |
| In Progress | 0 |
| Planned | 16 |

---

## Complete

| Document | Location | Audience | Description |
| :--- | :--- | :--- | :--- |
| High Level Project Plan | `docs/MCIS_High_Level_Plan.md` | Stakeholders, advisory board, contributors | Strategic overview: goals, target institutions, module architecture, phased roadmap, competitive landscape, risks |
| Phase 1 Database Schema | `docs/schema_phase1.md` | Developers, contributors, advisory board | Full schema for Phase 1 entities: Collections, Objects, Media, Users, AuditLog; conventions, indexes, design decisions, ER diagram |
| Phase 2 Database Schema (Locations, Loans, Donors) | `docs/schema_phase2.md` | Developers, contributors, advisory board | Full schema for Phase 2 entities: Locations, Loans, loan_objects, Donors; Phase 1 migrations, indexes, ER diagram |
| Use Case Library | `docs/use_cases/` | Developers, advisory board, pilot institutions | Use cases documenting operational workflows for every module; includes negative cases for constraint and role enforcement |
| README | `README.md` | All | Concise project summary; links to all key documents |
| CHANGELOG | `CHANGELOG.md` | All | Running record of all notable project changes with datetime-stamped entries |
| Documentation Plan | `docs/MCIS_Documentation_Plan.md` | All | This document — tracks all project documentation: complete, in progress, and planned |
| QR Code Generator README | [`github.com/juren53/qr-code-generator`](https://github.com/juren53/qr-code-generator) | All | Standalone module overview: origin, prototype findings (WHM ship's bell and bridge clock), testing results, open questions on backend, artifact numbering, and physical display |

---

## In Progress

*None at this time.*

---

## Planned

Documents are listed in the phase when they are expected to be drafted. All dates and phase assignments are subject to revision.

### Phase 1 — Core Framework

| Document | Audience | Description |
| :--- | :--- | :--- |
| **Deployment Guide** | Sysadmin, IT support | Step-by-step installation instructions: PostgreSQL setup and database creation; client installation on Windows, Linux, and macOS; first-run configuration; Internet Archive credential setup; automated daily backup schedule and off-site copy instructions. Includes a guided setup option for easier installation. |
| **System Architecture Document** | Developers, contributors | Detailed technical reference for developers: application and module structure; database connection layer; audit log design; role-based access control implementation; Internet Archive publisher integration; ExifTool image metadata embedding; single-file database fallback. Intended for contributors building or extending MCIS. |
| **Test Plan** | Developers, contributors | Automated test strategy and structure: test framework and tooling; unit, integration, and end-to-end test coverage goals; how to run the test suite locally; conventions for writing new tests; CI test execution. |
| **Contributing Guide** | Open source contributors | How to contribute to MCIS: development environment setup, coding standards, branch and PR conventions, running the test suite, community governance |
| **SECURITY.md** | All, open source contributors | Security disclosure policy and responsible reporting procedure for the MCIS open source project. Standard GitHub open source practice. |
| **Release Notes** | All | Plain-language summary of changes for each release: new features, bug fixes, known issues. One document per release, written for non-technical users and institutional stakeholders. |
| **Administrator Guide** | Admin users | User management (create, edit, deactivate); role assignment; system configuration; database backup and restore procedures; troubleshooting common issues |
| **User Guide: Collections & Objects** | Registrars, Staff, Volunteers | How to use the Collections and Objects modules: creating collections, accessioning objects, attaching images and documents with IPTC/EXIF metadata embedding, condition assessments, searching and browsing the catalogue, flagging objects for Internet Archive publication. Includes the Phase 1 basic IA publishing workflow (flag → approve → publish → status). |
| **Quick Start Guide** | Registrars, Staff, Volunteers | First-session walkthrough: logging in, creating a collection, accessioning a first object, attaching an image, and publishing a record to Internet Archive. Designed for new users with no prior MCIS experience. |
| **User Guide: QR Code Generator** | Registrars, Staff | How to generate and print QR code labels: triggering generation after an object is published to IA, label format and size options, batch printing for large sets, and physical display considerations for the Museum environment. |

### Phase 2 — Operational Modules

| Document | Audience | Description |
| :--- | :--- | :--- |
| **Phase 2 Database Schema (Members/Comms)** | Developers, contributors, advisory board | Schema for the Members/Comms module: membership tiers, renewal tracking, mailing list management; Phase 2 migrations and indexes |
| **User Guide: Members, Donors, Locations & Loans** | Registrars, Staff | How to use the Members, Donors, Locations, and Loans modules: managing the location hierarchy, recording object movements, creating and managing loan agreements, recording condition-out/in, donor records and acknowledgment letters, membership tiers and renewal tracking |
| **Internet Archive Publisher Guide** | Registrars, Admins | Full Phase 2 IA publishing workflow: setting up an Internet Archive account and collection; configuring field mapping (MCIS fields → IA metadata); managing the batch publish queue; publishing, updating, and unpublishing records. Basic Phase 1 IA publishing is covered in the User Guide: Collections & Objects. |

### Phase 3 — Operations Modules

| Document | Audience | Description |
| :--- | :--- | :--- |
| **User Guide: Inventory** | Staff | How to use the Inventory module: equipment and supply records, maintenance schedules |

### Phase 4 — Data Migration

| Document | Audience | Description |
| :--- | :--- | :--- |
| **Data Migration Guide** | Registrars, Sysadmin | How to import existing collection data into MCIS: CSV/Excel import with field mapping; PastPerfect data export handling; LIDO XML import; batch image import with IPTC metadata extraction and association; validation and error reporting |

### Phase 5 — Community & Sustainability

| Document | Audience | Description |
| :--- | :--- | :--- |
| **Extension Developer Guide** | Developers | How to build new modules using the MCIS extension framework: module registration, UI integration points, database migration conventions, packaging and distribution. Contingent on extension framework design — to be scoped before this document is drafted. |

---

## Notes

- User guides will be written module-by-module as each Phase reaches completion, not in advance. Writing user documentation before the UI is final produces documentation that immediately needs rewriting.
- The Deployment Guide is the highest-priority planned document — it is needed before any pilot institution can begin testing Phase 1.
- No schema documents are planned for Phases 3–5. The Members/Comms schema (Phase 2) is planned but not yet drafted — see Phase 2 above. Operations and Data Migration modules will be designed when those phases are scoped; Community & Sustainability (Phase 4) involves no new database entities.

---

_2026-06-17-1300_
