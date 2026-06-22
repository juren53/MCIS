# MCOS Controlled Vocabulary — Development and Maintenance Guide

_Draft Version 0.1 — 2026-06-22-1121_

---

## 1. What Is a Controlled Vocabulary and Why Does MCOS Need One?

A controlled vocabulary is a defined, authoritative list of terms used to describe a specific field. Instead of letting each cataloguer type whatever they choose in, say, the Medium field — "oil on canvas", "Oil on Canvas", "oils on canvas board", "O/C" — a controlled vocabulary constrains entries to a fixed list of preferred terms ("Oil on canvas") with approved variants and scope notes.

For MCOS, controlled vocabularies matter for three reasons:

**Search and retrieval.** A user searching for all photographs in the collection should not have to know whether a previous cataloguer typed "photograph", "Photograph", "photo", or "gelatin silver print". Consistent terms make search reliable.

**Interoperability and standards alignment.** MCOS publishes to the Internet Archive and aligns with Dublin Core, SPECTRUM, and AAM standards. These standards expect consistent, recognizable terminology. A condition field that uses "OK" instead of "Good" breaks field-mapping assumptions in downstream systems.

**Data portability.** When a museum exports its data or migrates to a future system, consistent values are far easier to map than free-text entries that accumulated years of inconsistent usage.

MCOS already has one implicit controlled vocabulary: the `condition` field is constrained by a database CHECK to `Excellent`, `Good`, `Fair`, `Poor`, or `Unknown`. This document extends that principle to every field in MCOS that benefits from a defined term list.

---

## 2. Scope — Fields Requiring Controlled Vocabularies

The following fields are candidates for controlled vocabularies, grouped by priority.

### Priority 1 — Implemented at Phase 1 launch

These fields are in the Phase 1 schema and must have controlled vocabularies before the first museum begins cataloguing, because inconsistency introduced at this stage is expensive to correct later.

| Field | Table | Current State | Proposed Source |
| :--- | :--- | :--- | :--- |
| `condition` | `objects` | CHECK constraint: Excellent, Good, Fair, Poor, Unknown | Already defined — no change needed |
| `rights_statement` | `objects` | Free text | RightsStatements.org standard terms (see §3) |
| `media_role` | `media` | Not yet designed | MCOS-defined terms (see §4) |
| User roles | `users` | Hardcoded: Admin, Registrar, Staff, Volunteer, Read-Only | Already defined — no change needed |

### Priority 2 — Implemented at Phase 2 launch

These fields are introduced in Phase 2 or are required before meaningful cross-collection querying becomes possible.

| Field | Table | Current State | Proposed Source |
| :--- | :--- | :--- | :--- |
| `loan_status` | `loans` | Not yet implemented | MCOS-defined terms: Pending, Active, Returned, Cancelled |
| `loan_type` | `loans` | Not yet implemented | MCOS-defined: Outgoing, Incoming |
| `location_type` | `locations` | Not yet implemented | MCOS-defined: Storage, Display, Off-site, Loan, Unknown |
| `accession_method` | `objects` | Not yet in schema | MCOS-defined (see §4) |

### Priority 3 — Recommended for Phase 1 but not required

These fields carry real interoperability and search value but are currently free text. Adding controlled vocabularies for them should be weighed against the risk of over-constraining small institutions with unusual collections.

| Field | Table | Notes |
| :--- | :--- | :--- |
| `medium` | `objects` | Getty AAT is the authoritative source; a curated MCOS subset is more practical |
| `object_type` | `objects` | Not yet in schema; useful for filtering and reporting |
| `period` / `era` | `objects` | Not yet in schema; Getty AAT covers this |

---

## 3. External Authority Files to Draw From

MCOS does not need to define terms that authoritative external sources have already defined. Where an appropriate external vocabulary exists, MCOS should adopt or profile it rather than invent its own.

### Getty Art & Architecture Thesaurus (AAT)

The most comprehensive authority for art and material culture terminology. Covers medium and materials, object types, stylistic periods, geographic terms, and more. Searchable at [vocab.getty.edu/aat](https://vocab.getty.edu/aat). Each term carries a persistent numeric identifier (e.g. AAT 300128347 = "oil paint (paint)") that survives label changes.

**Use in MCOS:** Preferred source for `medium` and `object_type` terms. Practical approach: an MCOS-maintained subset of AAT terms, organized by collection type (art, photograph, document, textile, three-dimensional object), allowing cataloguers to pick from a manageable list while the underlying identifier preserves interoperability.

### RightsStatements.org

Twelve standardized rights statements for cultural heritage institutions, developed by Europeana and the Digital Public Library of America. Each statement has a persistent URI, a short label, and a clear definition. Accepted by the Internet Archive and many aggregators.

**Use in MCOS:** The `rights_statement` field should offer the RightsStatements.org terms as a pick list in the UI, with the full URI stored in the database. The most commonly applicable terms for small US museum collections are:

| Label | URI Suffix | Meaning |
| :--- | :--- | :--- |
| In Copyright | `/InC/1.0/` | Work is in copyright |
| In Copyright — Educational Use Permitted | `/InC-EDU/1.0/` | In copyright; educational use permitted |
| No Copyright — United States | `/NoC-US/1.0/` | US government work; no copyright |
| No Known Copyright | `/NKC/` | Copyright holder unknown; good-faith search made |
| Public Domain Mark | `/PDM/1.0/` | Work is in the public domain |
| Copyright Not Evaluated | `/CNE/1.0/` | Rights not yet assessed |

**Note:** A rights statement is required before any object can be published to the Internet Archive. The UI should enforce this gate — `rights_statement` must be set before `ia_queued` can be set to TRUE.

### SPECTRUM 5.1 (Collections Trust)

SPECTRUM procedure definitions inform vocabulary terms for loan status, object entry methods, and condition terminology. When MCOS condition grades align with SPECTRUM's recommended condition vocabulary, records are more portable between systems.

### Dublin Core Metadata Element Set

Dublin Core's controlled vocabulary for `dc:type` (Collection, Dataset, Event, Image, InteractiveResource, MovingImage, PhysicalObject, Service, Software, Sound, StillImage, Text) provides a field that IA uses. MCOS should map object types to this list for IA export, even if the internal vocabulary is richer.

---

## 4. MCOS-Defined Terms

For fields where no external authority is appropriate, MCOS defines its own terms. These are maintained in this document and in lookup tables in the database (see §6).

### Condition (`objects.condition`)

Already implemented as a CHECK constraint.

| Term | Scope Note |
| :--- | :--- |
| Excellent | No visible deterioration; no repair needed |
| Good | Minor wear; stable; no immediate treatment needed |
| Fair | Moderate deterioration or damage; monitoring required |
| Poor | Significant deterioration or damage; treatment needed |
| Unknown | Condition has not been assessed |

### Accession Method (`objects.accession_method`) _(proposed)_

How the object entered the collection.

| Term | Scope Note |
| :--- | :--- |
| Gift | Donated by an individual or organization |
| Bequest | Received as a bequest from a deceased donor's estate |
| Purchase | Acquired with museum funds |
| Transfer | Transferred from another institution or government body |
| Field Collection | Collected directly by museum staff or authorized party |
| Found in Collection | Object discovered in holdings with no documentation of acquisition |
| Loan to Own | Object originally received on loan; legal title later transferred |
| Unknown | Acquisition method not documented |

### Media Role (`media.media_role`) _(proposed)_

The function or view captured by an attached image or document.

| Term | Scope Note |
| :--- | :--- |
| Primary | The main representative image; used as the thumbnail and IA cover image |
| Detail | Close-up of a feature, inscription, maker's mark, or area of damage |
| Verso | Reverse side of a two-dimensional object |
| Installation View | Object shown in its display context |
| Conservation | Image taken specifically to document condition |
| Document | Attached document (deed of gift, loan agreement, condition report, etc.) |
| Other | Does not fit any category above |

### Loan Status (`loans.loan_status`) _(Phase 2)_

| Term | Scope Note |
| :--- | :--- |
| Pending | Loan agreement initiated but not yet active |
| Active | Loan is in force; object is on loan |
| Returned | Object has been returned and condition-in documented |
| Cancelled | Loan was cancelled before it became active |
| Overdue | Active loan has passed its agreed return date |

### Location Type (`locations.location_type`) _(Phase 2)_

| Term | Scope Note |
| :--- | :--- |
| Storage | Object is in a storage area not accessible to the public |
| Display | Object is on display in a public gallery or exhibit |
| Off-site | Object is at an external location (conservation lab, exhibition venue, etc.) |
| Loan | Object is currently on outgoing loan |
| Unknown | Location not recorded or unknown |

---

## 5. How Controlled Vocabularies Are Developed

### Step 1 — Identify the field

A field is a candidate for a controlled vocabulary when:
- It will be used for filtering, reporting, or faceted search
- It maps to a term in an external standard (Dublin Core, IPTC, SPECTRUM, AAT)
- It will appear in Internet Archive metadata
- Its values need to be consistent across cataloguers and over time

Free text is appropriate when the content is always narrative (provenance, condition notes, description) and will not be used as a filter or exported to a standardized field.

### Step 2 — Consult external authorities first

Before defining a term list, check whether AAT, RightsStatements.org, Dublin Core, or SPECTRUM already defines the vocabulary for this field. If they do, adopt or profile their terms rather than reinventing. Record the external authority in this document.

### Step 3 — Draft the term list

For MCOS-defined vocabularies:
- Aim for the fewest terms that cover the real cases. Sparse, precise lists are more useful than exhaustive ones.
- Write a scope note for each term that tells a cataloguer when to use it and — crucially — when not to.
- Include an "Unknown" or "Other" escape valve for terms that genuinely don't fit. This prevents cataloguers from forcing records into the wrong category.
- Avoid synonyms within the list. Pick one term and note the others as excluded alternatives in the scope note.

### Step 4 — Review with domain expertise

Before any vocabulary is deployed in a production database, it should be reviewed by:
- A working registrar or collections manager (MCOS advisory board [TBD])
- At least one pilot museum with a relevant collection type

GitHub Issues or Discussions is the appropriate venue for this review — opening an issue tagged `vocabulary` ensures the discussion is public and searchable.

### Step 5 — Implement in the schema

New or changed vocabulary terms require:
1. A database migration adding or altering the lookup table or CHECK constraint (see §6)
2. An update to this document
3. A CHANGELOG entry

Terms are never simply renamed in place without a migration — renaming a term in code without updating existing records creates silent inconsistency.

---

## 6. Storage in the Schema

MCOS uses two mechanisms for controlled vocabularies, chosen by how frequently terms are expected to change.

### CHECK constraints (for stable, small sets)

Used for `condition` and similar fields where the set of values is fixed by professional standards and unlikely to ever change. Enforced at the database level — no invalid value can be inserted.

```sql
condition VARCHAR(20) CHECK (condition IN ('Excellent','Good','Fair','Poor','Unknown'))
```

**Trade-off:** A CHECK constraint is simple and requires no join. Changing the list requires a schema migration (`ALTER TABLE ... DROP CONSTRAINT ... ADD CONSTRAINT ...`). Use for sets of five or fewer terms that are stable across years.

### Lookup tables (for larger or evolving sets)

Used for vocabularies that museums may need to extend or that will grow over time, such as `medium`, `object_type`, or `rights_statement`.

```sql
-- Example lookup table pattern
CREATE TABLE cv_medium (
    medium_id   SERIAL PRIMARY KEY,
    term        VARCHAR(255) NOT NULL UNIQUE,
    scope_note  TEXT,
    aat_id      VARCHAR(50),          -- Getty AAT identifier if applicable
    is_active   BOOLEAN NOT NULL DEFAULT TRUE,
    sort_order  INTEGER
);

-- objects.medium_id references cv_medium(medium_id)
```

Key conventions for lookup tables:
- `is_active` allows a term to be retired without deleting rows that reference it. The UI hides inactive terms in pick lists but existing records remain valid.
- `aat_id` (or equivalent) records the external authority identifier for any term drawn from AAT or another controlled source.
- `sort_order` controls display order in pick lists independently of alphabetical or insertion order.
- The table prefix `cv_` distinguishes vocabulary tables from entity tables.

### Hybrid: stored URI, displayed label

For `rights_statement`, the database stores the full RightsStatements.org URI (e.g. `https://rightsstatements.org/vocab/InC/1.0/`) and the UI presents the human-readable label. This keeps the stored value authoritative and machine-readable while the pick list remains usable.

---

## 7. Maintenance and Governance

### Proposing a new term

Anyone — a cataloguer, a pilot museum, a developer — can propose a new term or a change to an existing term by opening a GitHub Issue tagged `vocabulary`. The issue should include:
- The field and vocabulary affected
- The proposed term
- A draft scope note
- The reason the existing terms do not cover the case

### Approval

- **MCOS-defined vocabularies:** The project maintainer reviews the proposal, consults the advisory board [TBD] if available, and merges a change to this document alongside the schema migration.
- **External authority terms:** If a term exists in AAT or another authority, cite the authority and identifier. No novel scope note is needed — link to the external definition.

### Retiring a term

Terms are never hard-deleted from lookup tables while records reference them. The process is:
1. Set `is_active = FALSE` — the term disappears from pick lists but existing records are unaffected.
2. If migration to a replacement term is needed, write a data migration script and document it in the CHANGELOG.
3. Update this document to note the retired term and its replacement.

### Version tracking

This document is the authoritative reference. Every change to a vocabulary — new term, scope note edit, retirement — requires:
- A pull request updating this document
- A corresponding database migration
- A CHANGELOG entry citing this document's version number

The document version in the header (`Version 0.1`) increments with each change. Git history provides the full audit trail.

### Schema migrations and backward compatibility

When a CHECK constraint changes, existing data must be validated against the new constraint before the migration commits. The migration script should:
1. Report any rows that would violate the new constraint
2. Either correct them (with a logged transformation) or fail with an actionable error
3. Then apply `ALTER TABLE` to add the updated constraint

When a lookup table term is renamed, update all referencing rows before renaming the term row, within a single transaction.

---

## 8. Relationship to External Standards

| Standard | Vocabulary Intersection with MCOS |
| :--- | :--- |
| **Dublin Core** | `dc:type` maps to MCOS `object_type`; `dc:rights` maps to MCOS `rights_statement` URI |
| **IPTC** | `copyright notice` maps to MCOS `rights_statement`; image type maps to MCOS `media_role` |
| **RightsStatements.org** | Primary source for `rights_statement` values |
| **Getty AAT** | Primary authority for `medium` and `object_type` terms; use AAT identifiers in lookup tables |
| **SPECTRUM** | Condition vocabulary and accession method terminology align with SPECTRUM definitions |
| **AAM** | Condition assessment and rights documentation requirements drive which vocabulary fields are required vs. optional |

The goal is not full certification against any of these standards but field-level alignment: an MCOS export should produce metadata that any collections professional recognizes, and any aggregator or future system can consume without manual normalization.

---

## 9. Quick Reference — Vocabulary Status by Field

| Field | Table | Phase | Status | Source |
| :--- | :--- | :--- | :--- | :--- |
| `condition` | `objects` | 1 | Defined — CHECK constraint | MCOS-defined, SPECTRUM-aligned |
| `rights_statement` | `objects` | 1 | Defined — pick list required | RightsStatements.org |
| `media_role` | `media` | 1 | Draft — not yet in schema | MCOS-defined (see §4) |
| User roles | `users` | 1 | Defined — hardcoded | MCOS-defined |
| `accession_method` | `objects` | 1 | Draft — not yet in schema | MCOS-defined (see §4) |
| `loan_status` | `loans` | 2 | Draft | MCOS-defined, SPECTRUM-aligned |
| `loan_type` | `loans` | 2 | Draft | MCOS-defined |
| `location_type` | `locations` | 2 | Draft | MCOS-defined |
| `medium` | `objects` | 3 | Not started — currently free text | Getty AAT subset |
| `object_type` | `objects` | 3 | Not started — field not yet in schema | Getty AAT / Dublin Core |

---

_2026-06-22-1121_
