# DECISIONS_LOG.md — Research Intelligence RAG




### D-001 · APPROVED 2026-10-10 (owner)
**Decision:** Navigation is project-scoped. Global nav = Dashboard, Projects, Settings. Chat, Papers, Compare, Literature Review, Search are tabs inside a project.
**Reasoning:** All research objects belong to a topic. Global "Library"/"Chat" items force repeated paper-set selection and risk mixing projects.
**Alternatives:** Flat global nav with project selector (simpler to build, higher scope-error risk).
**Affected:** PRODUCT_SPEC §3; DESIGN_SYSTEM shell/breakpoints; HANDOFF routes.

### D-002 · APPROVED 2026-10-10 (owner)
**Decision:** Every answer is rendered as claim blocks labelled Source-supported / Synthesis / Conflicting / Hypothesis; uncited factual claims are not allowed in default style.
**Reasoning:** Core trust requirement; prevents fluent but unsupported text from looking authoritative.
**Alternatives:** Plain prose with footnotes (less visible provenance).
**Affected:** PRODUCT_SPEC §4.2; DESIGN_SYSTEM §6.7; HANDOFF §6 backend contract #2. **Requires backend structured output; labels are fallible and must be measured.**

### D-003 · APPROVED 2026-10-10 (owner)
**Decision:** Verbatim source text is always visually distinct (serif, tinted, left-bordered, "Original text" label) from generated text.
**Reasoning:** Users must tell evidence from AI generation at a glance.
**Affected:** DESIGN_SYSTEM §2, §3, §6.7.

### D-004 · APPROVED 2026-10-10 (owner)
**Decision:** Missing comparison data displays "Not reported in the source"; no inferred values.
**Affected:** PRODUCT_SPEC §4.4; HANDOFF §6 #4.

### D-005 · APPROVED 2026-10-10 (owner)
**Decision:** MVP ships light theme only; tokens are semantic so dark can follow.
**Affected:** DESIGN_SYSTEM §2, Settings (Appearance shows light only; no dead toggle).

### D-006 · APPROVED 2026-10-10 (owner)
**Decision:** Paper detail is a drawer, not a route; Source Inspector is a right-side panel (400px, collapsible; sheet on small screens).
**Reasoning:** Preserves reading context while verifying.
**Affected:** PRODUCT_SPEC §3.2, §4.3; DESIGN_SYSTEM §4–5; HANDOFF §5.

### D-007 · APPROVED 2026-10-10 (owner)
**Decision:** Scanned (no-text) PDFs are rejected as Failed with a reason in MVP; no OCR.
**Reasoning:** Keeps MVP achievable; avoids low-quality extraction silently feeding the RAG.
**Alternatives:** Best-effort OCR (cost/quality risk).
**Affected:** PRODUCT_SPEC assumptions A3; HANDOFF §4.3.

### D-008 · APPROVED 2026-10-10 (owner)
**Decision:** Collection Search is in the **late MVP**, built after upload, RAG Q&A, citations, inspection, comparison and literature review are reliable. It offers natural-language and keyword search across uploaded papers, passage previews, paper title and page references, and basic metadata filters. It **reuses the existing retrieval infrastructure**. External academic web search is excluded.
**Reasoning:** Core RAG workflow reliability comes first; search is a thin layer over retrieval.
**Alternatives:** Separate search system (rejected: unnecessary complexity).
**Affected:** PRODUCT_SPEC §1.5, S10, AC12; HANDOFF M7 and §2.1.

### D-009 · APPROVED 2026-10-10 (owner)
**Decision:** No "general-knowledge answer" fallback when evidence is missing in MVP.
**Affected:** PRODUCT_SPEC §4.2.2 (Q5).

### D-010 · APPROVED 2026-10-10 (owner)
**Decision:** MVP authentication is email + password through a standard authentication provider (no hand-rolled security). Requirements: secure authentication and sessions, password reset, protected projects/documents, server-side authorization, clear sign-out. Google sign-in is desirable if effort permits. University SSO is not in MVP; architecture stays provider-agnostic.
**Reasoning:** Lowest-risk path to a secure public-capable MVP without institutional dependencies.
**Affected:** PRODUCT_SPEC §2.3, S14, AC9; DESIGN_SYSTEM §6.13 (auth card with space for a provider button); HANDOFF M0, §2.1, M1 acceptance #1–2.
**Open:** email verification and auth rate limiting are Claude's assumption (A8), unconfirmed.

### D-011 · APPROVED 2026-10-10 (owner) — values PROVISIONAL
**Decision:** Start with 20 MB per PDF, 20 papers per project, 300 pages per PDF (subject to processing capability); multiple uploads allowed through a controlled queue. Values are **configurable product limits, not architectural constraints**, to be finalized with Antigravity after estimating storage, extraction, embedding, and query costs.
**Affected:** PRODUCT_SPEC §2.2, AC10; HANDOFF §2.2 (cost-estimate deliverable), §4.3, M1 acceptance #4–5.
**Follow-up:** record Antigravity's cost estimate and final values here.

### D-012 · APPROVED 2026-10-10 (owner)
**Decision:** Build a portfolio-quality MVP intended for eventual public access. Baseline privacy/security from the start: privacy notice, transparent storage and deletion, private-by-default projects, authentication and server-side access control, no public sharing by default, third-party AI processing disclosure, caution about sensitive/confidential uploads. No enterprise compliance, SSO, or legal infrastructure before validation; pre-launch legal/privacy requirements are tracked as a checklist.
**Affected:** PRODUCT_SPEC §2.4–2.5, S15, AC11; HANDOFF §6 #7, M1 acceptance #7; Settings → Privacy.

### D-013 · APPROVED 2026-10-10 (owner)
**Decision:** Priority order: (1) upload and processing, (2) multi-document RAG Q&A, (3) verifiable citations, (4) citation inspection, (5) basic comparison, (6) literature review, (7) Collection Search and remaining polish. The goal is a reliable, evidence-grounded assistant, not a visually impressive interface.
**Consequence applied by Claude:** Dashboard and onboarding move to late MVP/polish (landing = Projects list until then); the v0.1 P0/P1/P2 labels and the v0.1 build order are superseded.
**Affected:** PRODUCT_SPEC §1.5, §3.2, §3.4; HANDOFF §1–2.

### D-014 · PROPOSED
**Decision:** Limit values (file size, pages, papers per project) shown in UI copy come from configuration, never hardcoded in components.
**Reasoning:** Implements D-011 so limits can change without redesign.
**Affected:** DESIGN_SYSTEM §6.13; HANDOFF §2.2, §4.3.

### D-015 · PROPOSED
**Decision:** Show a short upload-point caution about confidential/unpublished/sensitive material and external AI processing, linked to the privacy notice.
**Reasoning:** Claude's interpretation of the owner's guidance in D-012; final wording needs review before public launch.
**Affected:** PRODUCT_SPEC §2.4; DESIGN_SYSTEM §6.13; HANDOFF §4.3.

### Status note (2026-10-10)
Owner approved D-001 to D-007 and D-009 ("approve all"). D-009 resolves open question Q5: no general-knowledge fallback in MVP.
Still PROPOSED (not covered by that approval, awaiting owner): **D-014** (limit values from configuration) and **D-015** (upload-point privacy caution).
Still open: Q6 (email verification/rate limiting assumption, Google sign-in scheduling), Q8 (hosting/jurisdiction of user data).

### Document-consistency review checklist (run at each phase end)
- [ ] Token names identical across DESIGN_SYSTEM and HANDOFF
- [ ] Status and claim vocabularies unchanged
- [ ] Routes in SPEC match HANDOFF
- [ ] Every `[ASSUMPTION]` either confirmed, changed, or still listed
- [ ] All open questions have an owner and status
