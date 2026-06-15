# MCIS — Museum Collections Information System

MCIS will be a free, open source collections management system designed for small museums, historical societies, and specialized collections that need professional-grade tools without the cost or complexity of commercial software such as PastPerfect, Argus, or The Museum System (TMS).

## What it will do

MCIS will be built on a two-tier architecture:

- **Public tier** — selected records and images published to the museum's Internet Archive collection for free, permanent public access
- **Private tier** — full operational data maintained in a secure, self-hosted PostgreSQL database covering collections, loans, donors, members, and inventory

## Key principles

- Free and open source — no licensing fees
- Runs on modest hardware the museum already owns
- Low training curve for volunteers and part-time staff with high turnover
- Multi-user client/server — PySide6 desktop clients connecting to a shared database
- Modular — museums can adopt only the modules they need and expand over time
- Data portable — museums own their data; no vendor lock-in; standard SQL exports
- Internet Archive publishing as a first-class feature, not an afterthought
- Accessible by design — UI will meet WCAG 2.1 AA keyboard navigation and screen reader standards

## Technology stack

| Layer           | Technology                      |
| :-------------- | :------------------------------ |
| Desktop client  | Python / PySide6                |
| Database        | PostgreSQL                      |
| ORM             | SQLAlchemy                      |
| IA publishing   | internetarchive Python library  |
| Packaging       | PyInstaller (Windows and Linux) |

## Project status

MCIS is in early development, currently in **Phase 0 — Project Infrastructure** (schema design, repository setup, CI configuration).

For the full project plan — including module architecture, phased roadmap, entity model, competitive landscape, risks, and glossary — see [docs/MCIS_High_Level_Plan.md](docs/MCIS_High_Level_Plan.md).

## Getting involved

We are looking for:

- A pilot museum willing to test early releases
- Contributors familiar with Python, PySide6, or PostgreSQL
- Museum professionals who can advise on collections workflow requirements

Open an issue or start a discussion to get involved.

## License

License to be determined — MIT or GPL v3 under consideration. See the [project plan](docs/MCIS_High_Level_Plan.md) for the licensing rationale.
