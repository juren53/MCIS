# MCOS — Museum Collections & Operations System

MCOS is a free, open source collections management system designed for small museums, historical societies, and specialized collections that need professional-grade tools without the cost or complexity of commercial software such as PastPerfect, Argus, or The Museum System (TMS).

## Overview

MCOS separates public discovery from private operations through a two-tier architecture:

- **Public tier** — selected records and images published to the museum's Internet Archive collection for free, permanent public access
- **Private tier** — full operational data maintained in a secure, self-hosted database covering collections, loans, donors, members, and inventory

A lightweight desktop client can be installed on a laptop or workstation and connect to a shared PostgreSQL database running on one machine the museum already owns — or run against a single-file SQLite database for single-user installations. No web server or cloud subscription required.

## Key principles

- Free and open source — no licensing fees
- Internet Archive publishing as a first-class feature, available from the first release
- Runs on modest hardware the museum already owns
- Intuitive and easy to use for staff and volunteers experienced in museum and archive work — a low training curve is a first-order design requirement
- Modular — museums can adopt only the modules they need and expand over time
- Data portable — museums own their data; no vendor lock-in; standard SQL exports
- Accessible by design — consistent, intuitive interface operable without specialist training

## Technology stack

| Layer          | Technology                     |
| :------------- | :----------------------------- |
| Desktop client | Python / PySide6               |
| Database       | PostgreSQL (multi-user) / SQLite (single-user) |
| Database layer | SQLAlchemy                     |
| IA publishing  | Internet Archive API           |
| Image metadata | ExifTool (IPTC/EXIF embedding) |
| Packaging      | PyInstaller (Windows, Linux, macOS) |

## Project status

MCOS is in **Phase 0 — Project Infrastructure**. Schema design and use case documentation are complete; application development is beginning.

For the full project plan — including module architecture, phased roadmap, data model, competitive landscape, risks, glossary, and key reference documents — see [docs/MCOS_High_Level_Plan.md](docs/MCOS_High_Level_Plan.md).

## Documentation

| Document | Description |
| :------- | :---------- |
| [High Level Plan](docs/MCOS_High_Level_Plan.md) | Full project plan: architecture, phases, entity model, risks, glossary, and [key reference documents](docs/MCOS_High_Level_Plan.md#14-key-documents) |
| [Phase 1 Schema](docs/schema_phase1.md) | Database schema — Collections, Objects, Media, Users, AuditLog |
| [Phase 2 Schema](docs/schema_phase2.md) | Database schema — Locations, Loans, Donors; Phase 1 migrations |
| [Use Cases](docs/use_cases/index.md) | 42 use cases documenting operational workflows for every module across Phases 1 and 2 |
| [Documentation Plan](docs/MCOS_Documentation_Plan.md) | All project documentation — complete, in progress, and planned |

## Modules

MCOS is modular by design. The following standalone modules are available as separate repositories:

| Module | Repository | Description |
| :----- | :--------- | :---------- |
| QR Code Generator | [juren53/qr-code-generator](https://github.com/juren53/qr-code-generator) | Generates printable QR codes linking physical Museum artifacts to their Internet Archive records |

## Getting involved

We are looking for:

- A pilot museum willing to test early releases
- Contributors familiar with Python, PySide6, or PostgreSQL
- Museum professionals who can advise on collections workflow requirements

Visit the [GitHub repository](https://github.com/juren53/MCOS) to open an issue or start a discussion.

## License

License will be MIT or GPL v3 — decision expected before the first code release.

---

_2026-06-22-0623_
