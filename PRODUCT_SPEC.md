# PRODUCT_SPEC.md — Research Intelligence RAG


---

## 0. Document governance (Deliverable 5)

| File | Answers | Owner of edits | Read by |
|---|---|---|---|
| `PRODUCT_SPEC.md` | *What* and *why* | Claude (owner approves) | Everyone |
| `DESIGN_SYSTEM.md` | *How it looks and behaves* | Claude | Antigravity before any UI work |
| `IMPLEMENTATION_HANDOFF.md` | *What to build, in what order, with what acceptance criteria* | Claude (Antigravity annotates status/blockers) | Antigravity |
| `DECISIONS_LOG.md` | *Why we changed or chose something* | Whoever makes the decision | Everyone |

**Precedence when documents conflict:** `DECISIONS_LOG.md` (latest approved entry) > `PRODUCT_SPEC.md` > `DESIGN_SYSTEM.md` > `IMPLEMENTATION_HANDOFF.md`. A conflict must be logged and resolved in the document that is lower in precedence *before* implementation continues.

**Sync rules**
1. One change = one commit touching the authoritative doc **and** a `DECISIONS_LOG.md` entry (if it is a decision, not a typo).
2. Every decision entry lists **Affected docs/modules**. The author updates all of them in the same commit or marks them `STALE: <reason>` at the top of the section.
3. Antigravity reads the relevant spec + design-system sections before starting each module, and records in the handoff doc: `Module → spec version read → status`.
4. Antigravity may *propose* changes (token tweaks, technical constraints) by adding a `PROPOSED` entry in the log; they become binding only after owner approval.
5. At the end of each phase, Claude runs a consistency review (token names, component names, status vocabulary, route names) and logs the result.
6. Claude does not claim files in the repo were updated unless it actually performed the write. Until repo access exists, the owner pastes these files into the repo.
7. Shared vocabulary (do not rename in only one doc): document statuses `Uploaded · Processing · Ready · Failed`; claim labels `Source-supported · Synthesis · Conflicting · Hypothesis`; component names as in `DESIGN_SYSTEM.md`.

---

## 1. Product definition (Deliverable 1)

### 1.1 Description
Research Intelligence RAG is a multi-document academic research workspace. Users upload PDFs into **Research Projects**, ask questions across them, and receive answers where every substantive claim is tied to an inspectable passage (paper, page, excerpt). The product also compares papers side by side and drafts evidence-backed literature reviews that the user edits and verifies.

### 1.2 Value proposition
> Ask questions across your papers and *check every claim against the source* — without re-reading everything.

Differentiators (the product is judged on these, not on fluency):
- **Verifiability first:** citations open the actual passage, not just a title.
- **Honest uncertainty:** "no evidence found" and "not reported in the source" are first-class states.
- **Structured research outputs:** comparison tables and literature matrices, not only chat prose.
- **Visible disagreement:** conflicting findings are surfaced, never averaged away.

### 1.3 Target users and personas
| Persona | Context | Primary need |
|---|---|---|
| **Thesis student** (final-year CSE, postgrad) | 20–60 papers, deadlines, limited domain depth | Understand fast, cite correctly, avoid misreading |
| **Research assistant** | Builds literature tables for a supervisor | Accurate extraction into comparison matrices |
| **Academic researcher / faculty** | Knows the field, skeptical of AI | Traceable evidence; quick screening of new papers |

**[ASSUMPTION]** Primary design persona is the thesis student; the skeptical researcher sets the trust bar.

### 1.4 Core problems
1. Reading many papers manually is slow.
2. Evidence is scattered across PDFs.
3. Comparing methods, datasets, metrics, and results is tedious and error-prone.
4. Contradictions between studies are easy to miss.
5. Tracking page-level sources is manual.
6. Literature review tables are built by hand.
7. AI tools blur verified evidence with plausible-sounding invention.

### 1.5 MVP scope and priority order (owner-approved 2026-10-10)

**Guiding objective:** a reliable, evidence-grounded research assistant, not a visually impressive interface. When time is short, protect items 1–4 and cut from the bottom.

| Order | Feature | Tier |
|---|---|---|
| 1 | PDF upload and document processing (multi-PDF, controlled queue, statuses, retry, duplicates, removal) | Core |
| 2 | Multi-document RAG question answering (streaming, scope control, no-evidence handling) | Core |
| 3 | Accurate, verifiable citations (real stored passages, page numbers, claim-type labels) | Core |
| 4 | Citation inspection (Source Inspector panel, passage highlight, PDF page link if feasible) | Core |
| 5 | Basic paper comparison (structured table, citations, "Not reported in the source") | Core |
| 6 | Literature review generation (editable, Markdown/CSV export, verification notice) | Core |
| 7 | Collection Search (natural-language + keyword, passage previews, title + page refs, basic metadata filters) | **Late MVP** |
| 7 | Remaining polish: Dashboard, onboarding, Settings refinements, empty/loading/error-state pass | **Late MVP** |

**Foundations required before item 1** (not user-facing features, but blocking): email/password authentication with server-side authorization (§2.3) and research project create/edit/delete/search/sort.

**Scope decisions**
- **Collection Search (approved):** late MVP. It must **reuse the existing retrieval infrastructure** from item 2; no separate search system. External academic web search is out of scope.
- **Dashboard:** because it is polish, until it is built the post-login landing page is the Projects list. This does not change the Dashboard spec in §4.1.
- **Dark theme:** tokens are dark-ready, only light ships in MVP.

### 1.6 Non-goals (MVP)
Autonomous research agents · knowledge graphs · voice · team collaboration · billing · enterprise administration and enterprise compliance · university SSO · public document sharing · external web/academic-database search · analytics dashboards · fake statistics · reference-manager sync (Zotero/Mendeley; roadmap) · non-PDF formats · OCR for scanned PDFs (see A3). Google sign-in is **not** a requirement; it is a desirable enhancement (§2.3).

---

## 2. Owner decisions, limits, security and privacy, assumptions

### 2.1 Resolved questions (2026-10-10)
| Was | Resolution | Log |
|---|---|---|
| Q1 Collection Search in MVP? | Yes, late MVP, reusing retrieval | D-008 |
| Q2 Auth method | Email + password via standard provider; Google if effort permits; no SSO | D-010 |
| Q3 Upload limits | Provisional configurable limits (§2.2), finalized after cost estimates | D-011 |
| Q4 Audience | Portfolio-quality MVP intended for eventual public access | D-012 |
| Q5 General-knowledge fallback? | No, not in MVP | D-009 |

### 2.2 Upload and project limits (provisional, configurable)
| Limit | Starting value | Notes |
|---|---|---|
| Max PDF size | 20 MB per file | |
| Max papers per project | 20 | |
| Max pages per PDF | 300 | "Subject to processing capability": Antigravity may lower it |
| Concurrency | Multiple uploads allowed; processing through a **controlled queue** | Queued files show `Uploaded`; see DESIGN_SYSTEM §6.8 |

Rules:
- Limits are **configuration**, not architecture. UI copy must read values from configuration (never hardcode "20 MB" in components).
- **Finalization gate:** Antigravity estimates storage, extraction, embedding, and query (LLM) costs at these limits and proposes adjustments *before* the limits are treated as final. Until then, the numbers are provisional and may change without a redesign.
- Rejections must be helpful and specific: unsupported type, corrupted file, password-protected file, over size, over page limit, project paper limit reached, scanned/no extractable text.
- Heads-up: 20 papers × up to 300 pages can be very large for whole-project features (comparison, literature review). Those features must work from retrieved/extracted material, not by sending whole papers to the model; Antigravity to confirm feasibility.

### 2.3 Authentication and access control (MVP requirements)
- Email + password sign-up/sign-in using a **standard authentication provider or well-established library**; no custom cryptography or hand-rolled session security.
- Secure session management; clear **sign-out** control (user menu).
- **Password reset** flow (request, emailed link, set new password, confirmation, expired-link handling).
- Research projects and documents are **private by default**; **server-side authorization checks on every request** (project, paper, conversation, citation passage, export). UI hiding is never sufficient.
- No public document sharing by default; no share links in MVP.
- **Google sign-in:** desirable if effort permits; not blocking. Design the sign-in screen so a second provider button can be added without a redesign.
- **University SSO:** not in MVP; keep the auth architecture provider-agnostic so it can be added later.
- **[ASSUMPTION]** Email verification at sign-up, and basic rate limiting on auth endpoints. Not stated by the owner; included as a standard baseline. Confirm or drop.

### 2.4 Privacy and security baseline (MVP)
- **Clear privacy notice**, linked from sign-up, upload area, and Settings → Privacy.
- **Transparent storage and deletion:** user can see what is stored (papers, conversations, generated reviews), and delete a paper, a project, or their account. Deletion must remove files, extracted text, chunks, and embeddings; the UI states what deletion covers, and any backup/retention delay if one exists. **[Antigravity to report actual behavior so the copy is true.]**
- **Third-party AI disclosure:** plain-language statement that document excerpts and questions are sent to external AI service providers to generate answers and embeddings, naming the providers once chosen. Do **not** state anything about provider training or retention practices unless verified against the provider's current terms.
- **Upload-time caution** **[my interpretation of the owner's "avoid uploading sensitive or confidential research materials without appropriate safeguards"]:** a short notice at the upload point: avoid uploading confidential, unpublished, or sensitive material unless comfortable with external AI processing. Wording to be reviewed before public launch.
- Do not overbuild: no enterprise compliance, SSO, or legal infrastructure before the product is validated.

### 2.5 Requirements to resolve before a real public launch (checklist, not MVP work)
This is a product checklist, **not legal advice**; items marked (L) need review by someone qualified.
- (L) Privacy policy and terms of use drafted/reviewed for the jurisdictions where you and your users are located.
- (L) Copyright and licensing: users will upload papers they may not have the right to redistribute; terms must cover storage, processing, and no public sharing.
- (L) Data-protection obligations (consent, access/deletion rights, breach notification) under applicable law. Which laws apply is unconfirmed.
- Data-retention policy: how long files, embeddings, chats, and backups persist; deletion SLAs.
- Subprocessor list (hosting, storage, AI providers) and their data-processing terms.
- Security review: authorization tests (cross-user access attempts), upload hardening (file validation, size/page limits, malware scanning decision), secrets management, backups.
- Prompt-injection risk: PDF text is untrusted input to the LLM; treat as a known risk with mitigations and tests.
- Abuse and cost controls: per-user quotas, rate limits, usage monitoring.
- Account lifecycle: email verification, deletion, support contact.
- Accessibility check against WCAG AA.

### 2.6 Assumptions
- A1. Single-user accounts; projects are private (confirmed in spirit by D-012).
- A2. PDF-only, text-extractable papers; English-language primary.
- A3. Scanned/image-only PDFs are detected and flagged as Failed with a reason; no OCR in MVP.
- A4. Page numbers are the PDF's physical page index; printed page labels shown additionally only if extractable. Citations state which.
- A5. Retrieval scope default = whole active project; user can narrow to selected papers.
- A6. Claim-type labels come from the backend (structured output), validated; the UI never infers them.
- A7. Duplicate detection by content hash within a project.
- A8. Email verification and auth rate limiting (see §2.3).

### 2.7 Open questions
- **Q6.** Confirm A8 (email verification, rate limiting) and whether Google sign-in should be scheduled or left opportunistic.
- **Q7 (Antigravity).** Cost estimates and final limits (§2.2); PDF page viewer feasibility; actual deletion/backup behavior.
- **Q8.** Hosting/jurisdiction for user data, which feeds the §2.5 legal items.

---

## 3. Information architecture (Deliverable 2)

### 3.1 Key structural decision
**The Project is the container of everything** (papers, chats, comparisons, reviews, search). Global "Document Library" and "Research Chat" items from the suggested nav would force users to repeatedly choose which papers they mean and invite cross-project mixing. So the nav is project-scoped.

### 3.2 Sitemap
```
/login /signup /reset-password   Auth (public)
/privacy                  Privacy notice (public)
/                         Dashboard (late MVP; until built, redirects to /projects)
/projects                 Projects list
/projects/:id             Project workspace (default tab: Chat)
   ├─ /chat               Research Chat (+ /chat/:conversationId)
   ├─ /library            Papers (upload, status, metadata)
   ├─ /compare            Paper Comparison
   ├─ /review             Literature Review Builder
   └─ /search             Collection Search
/settings                 Account · Appearance · Documents & data · Privacy
/onboarding               First-run (modal flow, not a separate marketing page)
```
Paper detail is a **drawer/panel**, not a route, to keep reading context. Optional deep link: `/projects/:id/library?paper=:paperId`.

### 3.3 Navigation structure
**Global sidebar (collapsible, 248px / 64px):** Dashboard · Projects · Settings · (user menu at bottom).
**Project switcher** at top of sidebar when inside a project.
**Project tab bar (inside project):** Chat · Papers · Compare · Literature Review · Search. Chat is the default landing.
**Right panel (contextual):** Source Inspector (chat, review, compare), Paper details (library). Collapsible; remembered per user.

### 3.4 Screen inventory
| ID | Screen | Tier (build order per §1.5) |
|---|---|---|
| S1 | Dashboard (new-user and returning states) | Late MVP |
| S2 | Projects list | Foundation |
| S3 | Project create/edit dialog | Foundation |
| S4 | Papers (library + upload) | Core (1) |
| S5 | Paper detail drawer | Core (1) |
| S6 | Research Chat (conversation list, thread, composer) | Core (2–3) |
| S7 | Source Inspector panel | Core (4) |
| S8 | Paper Comparison | Core (5) |
| S9 | Literature Review Builder | Core (6) |
| S10 | Collection Search | Late MVP (7) |
| S11 | Settings | Foundation (account, privacy, delete) + polish |
| S12 | Onboarding | Late MVP |
| S13 | Global error / 404 / offline | Polish |
| S14 | Auth: sign up, sign in, password reset, sign out | Foundation |
| S15 | Privacy notice page | Foundation |

### 3.5 Primary user journeys (summary; full behavior in §5)
1. **First run:** sign up → onboarding → create project → upload papers → wait for Ready → first question → inspect citation.
2. **Daily research:** open project → pick conversation → ask → read answer → open citations → follow-up → copy/export.
3. **Comparison:** Papers → select 2–6 Ready papers → Compare → choose attributes → review table → open cell citations → export CSV.
4. **Literature review:** enter research question → select papers → generate → verify per-section evidence → edit → export Markdown/CSV.
5. **Recovery:** failed upload → see reason → retry or replace; network drop mid-stream → keep partial answer, offer retry.

---

## 4. UX strategy (Deliverable 3)

### 4.1 Dashboard (S1)
Purpose: get back to work in one click. No fake statistics or decorative charts.
- **Returning user:** greeting line → primary button "New project" → *Recent projects* (up to 6 cards: title, paper count, last modified) → *Recently uploaded papers* (title, project, status chip) → *Recent conversations* (question preview, project, time).
- **New user:** single onboarding card with 3 real steps (Create project · Upload papers · Ask a question), each marking complete from real state. No sample/fake data.
- States: skeleton rows, per-section error with retry, empty per-section (one line + CTA).

### 4.2 Research Chat layout (S6) — highest priority
Three regions inside the project workspace:

```
| Sidebar | Conversations (240) | Thread (flex, reading width 720–760) | Source Inspector (400, collapsible) |
```
- **Scope bar** above the thread: "Searching 12 of 14 papers ▾" — always visible; click to narrow papers. Prevents silent scope errors.
- **Thread:** user question; assistant answer as *structured blocks* (see 4.2.1); sources strip beneath each answer; actions row (copy, export, 👍/👎, regenerate); suggested follow-ups (generated only from retrieved content).
- **Composer:** multiline, send on Enter, Shift+Enter newline, stop button during streaming, disabled with explanation if no Ready papers.
- **Streaming:** text streams; citation markers appear as soon as resolvable; unresolved markers are never shown as final. If the stream drops, the partial answer is kept and labelled "Incomplete".

#### 4.2.1 Answer anatomy and claim types
Each answer is a list of **claim blocks**, each carrying one label:

| Label | Meaning | Visual treatment |
|---|---|---|
| **Source-supported** | Directly stated in cited passage(s) | Default text; citation chips required |
| **Synthesis** | AI combines multiple sources | Subtle left-border + "Synthesis" tag; ≥2 citations required |
| **Conflicting** | Sources disagree | Amber-tinted block listing each side with its citations side by side |
| **Hypothesis** | Possible gap/inference, unverified | Dashed border + "Unverified hypothesis" tag; never styled as fact |

Rules: a block without citations may only be labelled Hypothesis or be a transitional sentence; the UI must not render an uncited factual claim in default style. **[ASSUMPTION A6]** backend provides label + citation IDs per block.

#### 4.2.2 No-evidence state
Replace the answer with a distinct card: "No supporting evidence found in the selected papers." Show what was searched (scope), offer: broaden scope, rephrase, upload more papers. Do **not** show a general-knowledge answer styled like a cited one. **[RESOLVED, D-009]** No general-knowledge fallback in MVP, including as a user-initiated action.

### 4.3 Citation inspection experience (S7)
- **Inline chip** `[3]`-style numeric marker + short label on hover/focus (e.g., "Smith 2021, p. 7").
- **Citation card** (under answer): paper title, authors/year if known, page, 2–3 line excerpt, "Open in inspector".
- **Source Inspector (right panel):** header (paper title, year, page) → *Retrieved passage* shown as verbatim text with the cited span highlighted and surrounding context (±1 paragraph) → "Open PDF at page" (opens PDF viewer at that page; **dependency: PDF page rendering**) → list of all citations in this answer with prev/next → "Copy citation".
- Verbatim extracted text is visually distinct (serif/quote style, bordered "Original text") from generated text (sans, no border) so the user always knows which is which.
- Page-number caveat: if only physical index is known, label "PDF p. 7".
- If the passage can't be located: show "Passage unavailable — citation could not be verified" and flag the claim as unverified. Never hide it.

### 4.4 Multi-paper comparison workflow (S8)
1. Entry: from Papers (select 2–6 Ready papers → "Compare") or Compare tab.
2. Attribute set (default all): Objective · Methodology · Models/algorithms · Datasets · Metrics · Reported results · Limitations · Future work. User can toggle rows.
3. Generate → table with papers as columns, attributes as rows (transposable on narrow screens). Each cell: concise text + citation chip(s). Empty = **"Not reported in the source"** (muted, italic; distinct from "Not found by extraction" if the backend reports low confidence).
4. Cell click → Source Inspector. Cells are not editable in MVP **[ASSUMPTION]**; export CSV (includes citation columns).
5. Limits: warn above 6 papers (table readability); cell text truncated at ~280 chars with expand.
6. Loading: per-column skeletons, cells fill as available. Failure: per-cell error with "Retry cell".

### 4.5 Literature review workflow (S9)
1. Inputs: research question (required), papers (default: all Ready; multi-select), optional structure (themes / chronological / methodological — **[ASSUMPTION]** MVP offers these three).
2. Generate → document-style editor: sections with claim-level citations, plus a comparison matrix section.
3. Persistent banner: "AI-generated draft. Verify every claim against its sources before use."
4. Each citation opens the Source Inspector. Hypotheses/gaps appear in a clearly separate "Possible research gaps (unverified)" section.
5. Editing: rich-text basics; edits don't re-verify. Edited text loses its "verified-by-source" styling — **[ASSUMPTION]** edited paragraphs get a subtle "edited" marker so provenance is honest.
6. Export: Markdown (with references list) and CSV (matrix). Export dialog repeats the verification notice.
7. Save as versions? **Not MVP**; autosave latest draft only.

---

## 5. Workflow specifications (summary table)

Full per-workflow detail (entry, action, response, loading, success, failure, recovery, a11y) is maintained in `IMPLEMENTATION_HANDOFF.md §4`. Workflows covered: onboarding · create project · multi-PDF upload · processing monitoring · cross-paper Q&A · citation inspection · comparison · literature review · export · insufficient evidence · failed upload/network recovery.

## 6. Functional acceptance criteria (product level)
- AC1. Every cited claim opens a source passage in ≤2 interactions.
- AC2. No answer renders factual claims without citation unless labelled Hypothesis.
- AC3. Comparison cells without source data say "Not reported in the source", never a guess.
- AC4. Document status is always one of Uploaded/Processing/Ready/Failed with a visible reason on Failed.
- AC5. Chat is blocked, with explanation, when scope has no Ready papers.
- AC6. Conflicting findings display both sides with citations.
- AC7. All primary flows complete using keyboard only.
- AC8. Generated literature reviews always display the verification notice.
- AC9. A user can sign up, sign in, reset a forgotten password, and sign out; every data request is authorized server-side (a user cannot read or modify another user's projects, papers, conversations, or passages).
- AC10. Upload enforces configured limits and shows specific reasons for every rejection; limit values shown in the UI come from configuration.
- AC11. Privacy notice, third-party AI disclosure, and deletion behavior are reachable from sign-up, upload, and Settings, and describe what the system actually does.
- AC12. Collection Search reuses the chat retrieval layer and shows paper title and page reference for every result.

## 7. User stories (excerpt)
- As a thesis student, I upload 20 PDFs so I can ask questions across them.
- As a researcher, I click a citation to read the exact passage so I can trust or reject the claim.
- As a RA, I compare 5 papers' datasets and metrics in one table so I can build a review matrix.
- As a student, I see when evidence is missing so I don't cite something that isn't there.
- As a user, I retry a failed upload without losing the rest of the batch.

## 8. Roadmap (post-MVP, not committed)
Google sign-in (if not done in MVP) · university SSO · dark theme · reference-manager import/export · OCR · saved comparisons/review versions · highlights/notes on PDFs · BibTeX export · team sharing · external academic search.
