# MCIS Documentation Plan v0.4 — Critique

_2026-06-16_

Eight items, in rough priority order.

---

**1. Contributing Guide is too late (Phase 4)**
A Contributing Guide belongs in Phase 1 or at the latest Phase 2. If the repo is public on GitHub, contributors may show up before Phase 4. Waiting until "Community & Sustainability" to document how to contribute contradicts the open-source model.

**2. Extension Developer Guide depends on unscoped work**
The description mentions an "extension framework" but no such framework appears in the High Level Plan's phased roadmap. This entry is speculative — either note that explicitly, or defer it to "TBD" until the framework is scoped.

**3. No Quick Start or onboarding path for end users**
The Deployment Guide covers installation. The Administrator Guide covers admin tasks. The User Guides cover individual modules. But there's nothing that walks a new registrar through their first 30 minutes: log in, create a collection, add an object. That gap belongs somewhere — either a "Getting Started" guide or a dedicated section in the User Guide: Collections & Objects description.

**4. No SECURITY.md or security disclosure policy**
Standard for any open-source project accepting public contributions. Should be listed here, even if it's a one-page document.

**5. Use Case Library entry says "42 use cases" — don't hardcode counts**
That number will age badly. The description should say "use cases documenting operational workflows for every module" without the count, or note it as "42 as of v0.4."

**6. Phase 2 DB Schema listed as Complete — is it truly frozen?**
Schema documents tend to evolve as development proceeds. "Complete" implies it's finalized. If Phase 2 hasn't been built yet, this entry may be misleading. Consider a status like "Drafted" or add a note that schema docs are living documents.

**7. No document for release/version history beyond CHANGELOG**
The CHANGELOG tracks all changes. For a desktop application distributed to museums, a human-readable Release Notes document (one page per release, plain language) will matter to non-technical stakeholders. This could be as simple as noting it as a planned artifact.

**8. "In Progress" section serves no purpose right now**
An empty section with "*None at this time.*" is fine, but consider whether it adds value at this stage. It will become useful once active writing begins.
