# MCIS Use Cases — Index

Use cases for validating schema correctness and UI behaviour as each module is developed. Each use case includes specific test data, step-by-step actions, expected database state, and the schema tables exercised.

Negative cases (constraint violations, role denials) are marked **[NEGATIVE]** and are equally important — they verify that the system refuses invalid operations.

## Sources and References

These use cases were derived from:

- **MCIS schema design documents** — `docs/schema_phase1.md` and `docs/schema_phase2.md`. Constraints, FK behaviours, CHECK rules, and column definitions in the use cases reflect those documents directly.
- **MCIS High Level Project Plan** — `docs/MCIS_High_Level_Plan.md`. Module scope, role definitions (Admin/Registrar/Staff/Volunteer/ReadOnly), and the phased development plan inform which use cases belong to which phase.
- **Standard museum collections management practice** — accession numbering conventions, fuzzy date handling ("circa 1910"), condition vocabulary (Excellent/Good/Fair/Poor/Unknown), loan condition-out/condition-in workflow, and donor acknowledgment practice reflect common registrarial standards as described in *Museum Registration Methods* (5th ed., AAM Press) and the Collections Trust (UK) SPECTRUM standard.
- **GLAM open access practice** — Internet Archive flagging and anonymous donor display reflect GLAM sector norms for open access publishing.

### Fictional Test Institution
All test data uses a fictional institution: **Millbrook Historical Society**, with fictional staff, donors, objects, and locations. No real individuals or institutions are represented.

---

## Phase 1 — Core Framework

| ID          | Use Case                                      | Actor             | File |
| :---------- | :-------------------------------------------- | :---------------- | :--- |
| UC-COL-001  | Create a new collection                       | Admin, Registrar  | [Collections](UC-Phase1-Collections.md) |
| UC-COL-002  | Edit a collection's name and description      | Admin, Registrar  | [Collections](UC-Phase1-Collections.md) |
| UC-COL-003  | View all collections                          | Any               | [Collections](UC-Phase1-Collections.md) |
| UC-COL-004  | **[NEGATIVE]** Delete a collection with objects | Admin           | [Collections](UC-Phase1-Collections.md) |
| UC-COL-005  | Delete an empty collection                    | Admin             | [Collections](UC-Phase1-Collections.md) |
| UC-OBJ-001  | Accession a new object                        | Registrar         | [Objects](UC-Phase1-Objects.md) |
| UC-OBJ-002  | Accession an object with a fuzzy date_made    | Registrar         | [Objects](UC-Phase1-Objects.md) |
| UC-OBJ-003  | Edit an existing object record                | Registrar, Staff  | [Objects](UC-Phase1-Objects.md) |
| UC-OBJ-004  | Record a condition assessment                 | Registrar, Staff  | [Objects](UC-Phase1-Objects.md) |
| UC-OBJ-005  | Assign credit line and rights statement       | Registrar         | [Objects](UC-Phase1-Objects.md) |
| UC-OBJ-006  | Search for an object by accession number      | Any               | [Objects](UC-Phase1-Objects.md) |
| UC-OBJ-007  | Search for objects by title keyword           | Any               | [Objects](UC-Phase1-Objects.md) |
| UC-OBJ-008  | **[NEGATIVE]** Duplicate accession number     | Registrar         | [Objects](UC-Phase1-Objects.md) |
| UC-OBJ-009  | Flag an object for IA publication             | Registrar         | [Objects](UC-Phase1-Objects.md) |
| UC-MED-001  | Add a primary image to an object              | Staff             | [Media](UC-Phase1-Media.md) |
| UC-MED-002  | Add a second non-primary image                | Staff             | [Media](UC-Phase1-Media.md) |
| UC-MED-003  | **[NEGATIVE]** Add a second primary image     | Staff             | [Media](UC-Phase1-Media.md) |
| UC-MED-004  | Change which image is primary                 | Staff             | [Media](UC-Phase1-Media.md) |
| UC-MED-005  | Reorder images using sort_order               | Staff             | [Media](UC-Phase1-Media.md) |
| UC-MED-006  | Delete a single image                         | Registrar         | [Media](UC-Phase1-Media.md) |
| UC-MED-007  | Delete an object and verify media cascade     | Admin             | [Media](UC-Phase1-Media.md) |
| UC-USR-001  | Create an Admin user                          | Admin             | [Users](UC-Phase1-Users.md) |
| UC-USR-002  | Create a Volunteer user                       | Admin             | [Users](UC-Phase1-Users.md) |
| UC-USR-003  | Deactivate a user                             | Admin             | [Users](UC-Phase1-Users.md) |
| UC-USR-004  | **[NEGATIVE]** Login with deactivated account | —                 | [Users](UC-Phase1-Users.md) |
| UC-USR-005  | **[NEGATIVE]** Volunteer attempts to delete   | Volunteer         | [Users](UC-Phase1-Users.md) |
| UC-USR-006  | ReadOnly user views object details            | ReadOnly          | [Users](UC-Phase1-Users.md) |
| UC-USR-007  | Audit log captures a record change            | Any               | [Users](UC-Phase1-Users.md) |

## Phase 2 — Collections Modules

| ID          | Use Case                                          | Actor             | File |
| :---------- | :------------------------------------------------ | :---------------- | :--- |
| UC-LOC-001  | Create a room location                            | Admin, Registrar  | [Locations](UC-Phase2-Locations.md) |
| UC-LOC-002  | Create a case inside a room (hierarchy)           | Admin, Registrar  | [Locations](UC-Phase2-Locations.md) |
| UC-LOC-003  | Assign an object to a location                    | Registrar, Staff  | [Locations](UC-Phase2-Locations.md) |
| UC-LOC-004  | Move an object to a different location            | Registrar, Staff  | [Locations](UC-Phase2-Locations.md) |
| UC-LOC-005  | Retire a location                                 | Admin             | [Locations](UC-Phase2-Locations.md) |
| UC-LOC-006  | **[NEGATIVE]** Delete a parent location with children | Admin         | [Locations](UC-Phase2-Locations.md) |
| UC-LOC-007  | Create an off-site location for a borrowed object | Registrar         | [Locations](UC-Phase2-Locations.md) |
| UC-LNS-001  | Create an outgoing loan                           | Registrar         | [Loans](UC-Phase2-Loans.md) |
| UC-LNS-002  | Add multiple objects to a loan                    | Registrar         | [Loans](UC-Phase2-Loans.md) |
| UC-LNS-003  | Record condition-out for each loan object         | Registrar         | [Loans](UC-Phase2-Loans.md) |
| UC-LNS-004  | Return one object from a multi-object loan        | Registrar         | [Loans](UC-Phase2-Loans.md) |
| UC-LNS-005  | Return the last object and close the loan         | Registrar         | [Loans](UC-Phase2-Loans.md) |
| UC-LNS-006  | View overdue loans dashboard                      | Any               | [Loans](UC-Phase2-Loans.md) |
| UC-LNS-007  | Cancel a pending loan                             | Registrar         | [Loans](UC-Phase2-Loans.md) |
| UC-LNS-008  | **[NEGATIVE]** Delete an object on active loan    | Admin             | [Loans](UC-Phase2-Loans.md) |
| UC-LNS-009  | **[NEGATIVE]** Add same object twice to a loan    | Registrar         | [Loans](UC-Phase2-Loans.md) |
| UC-DNR-001  | Add an individual donor                           | Registrar         | [Donors](UC-Phase2-Donors.md) |
| UC-DNR-002  | Add an organization donor                         | Registrar         | [Donors](UC-Phase2-Donors.md) |
| UC-DNR-003  | Link a donated object to its donor                | Registrar         | [Donors](UC-Phase2-Donors.md) |
| UC-DNR-004  | Mark a donor as anonymous                         | Registrar         | [Donors](UC-Phase2-Donors.md) |
| UC-DNR-005  | Record acknowledgment sent and date               | Registrar, Staff  | [Donors](UC-Phase2-Donors.md) |
| UC-DNR-006  | **[NEGATIVE]** Set acknowledgment_date without sent flag | Registrar  | [Donors](UC-Phase2-Donors.md) |
| UC-DNR-007  | Deactivate a deceased donor                       | Registrar         | [Donors](UC-Phase2-Donors.md) |
