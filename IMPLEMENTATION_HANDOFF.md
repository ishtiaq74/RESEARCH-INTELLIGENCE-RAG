# IMPLEMENTATION_HANDOFF.md — Research Intelligence RAG

> For: Antigravity. Status: Draft v0.2 (owner decisions of 2026-10-10 applied). Read `PRODUCT_SPEC.md` and `DESIGN_SYSTEM.md` before each module. Do not invent styles for anything defined there.
> Technical choices (framework, DB, vector store, LLM) are **yours**; this document specifies observable behavior and quality bars. Items marked **[ASSUMPTION]** need confirmation; **[BLOCKER?]** items need your feasibility answer before the dependent module starts.

## 1. Module status tracker (Antigravity maintains)
| Module | Spec version read | Status | Notes |
|---|---|---|---|
| M0 Foundations (repo, tokens, shell, routing) | | Not started | |
| M0 Auth (email/password, reset, sign-out, authz) | | Not started | |
| M1 Projects | | Not started | |
| M2 Papers: upload, queue, ingestion, statuses | | Not started | |
| M3 Retrieval + RAG chat + verifiable citations | | Not started | |
| M4 Source Inspector (citation inspection) | | Not started | |
| M5 Comparison | | Not started | |
| M6 Literature review | | Not started | |
| M7 Collection Search | | Not started | |
| M8 Dashboard, onboarding, Settings polish, state pass | | Not started | |
| Cost estimate and final limits | | Not started | Required deliverable, see §2.2 |

## 2. Implementation order and dependencies (owner priority order)

Priority order set by the owner: 1 upload/processing, 2 multi-doc RAG Q&A, 3 verifiable citations, 4 citation inspection, 5 comparison, 6 literature review, 7 Collection Search and polish. **The goal is a reliable, evidence-grounded assistant, not a polished UI.** Build thin real UI over real pipelines; do not build mock-only screens that imply working backends.

```
M0  Foundations + Auth                       (blocks everything)
M1  Projects (CRUD, search, sort)            ← M0
M2  Papers: upload, controlled queue, ingestion, statuses, retry, dedupe, delete  ← M1
M3  Retrieval + RAG chat + citations         ← M2   [highest-risk; measure faithfulness, §9]
M4  Source Inspector                         ← M3 (may be built alongside M3)
M5  Comparison                               ← M2, M3 citation system
M6  Literature review                        ← M3, M5 extraction
M7  Collection Search                        ← M3 retrieval (REUSE; no separate system)
M8  Dashboard, onboarding, Settings polish, empty/loading/error pass
```
Until M8, post-login landing is the Projects list. Dashboard is not an early milestone.

### 2.1 Dependency list for Antigravity
| Need | Decision/requirement | Blocks |
|---|---|---|
| Auth provider | Standard provider/library; email+password; password reset; secure sessions; sign-out; provider-agnostic so Google/SSO can be added | M0 |
| Server-side authorization | Every endpoint checks ownership (projects, papers, conversations, citations, exports); include automated cross-user access tests | M1 onward |
| Config service for limits | Limits exposed via configuration and readable by the frontend | M2 |
| Controlled processing queue | Bounded concurrency, status updates, retry | M2 |
| Cost estimate | See §2.2 | Finalizing limits |
| Privacy notice and deletion | Real delete (files, text, chunks, embeddings); report backup/retention behavior | M2 (delete), M8 (Settings) |
| Provider disclosure | Names of AI/embedding/hosting providers for the privacy notice | Privacy copy |
| Search reuse | Collection Search implemented on the M3 retrieval layer | M7 |

### 2.2 Provisional limits (configurable) and the cost-estimate deliverable
| Config key (suggested names) | Starting value |
|---|---|
| `maxPdfSizeMB` | 20 |
| `maxPapersPerProject` | 20 |
| `maxPagesPerPdf` | 300 (lower if processing cannot support it) |
| `processingConcurrency` | your choice (bounded queue) |

**Before limits are finalized, please deliver** an estimate of: storage per paper, extraction time, embedding cost, per-query LLM cost (typical and worst case), hosting cost, plus whether comparison and literature review are feasible at 20 papers. Propose adjusted limits if needed. Record the result in `DECISIONS_LOG.md`. Limits must be changeable without UI redesign or code changes to components.

## 3. Reusable components (build once, from DESIGN_SYSTEM.md)
Button, IconButton, Input, Textarea, Select, Checkbox, Switch, Dialog, Drawer, Dropdown, Tooltip, Tabs, Table (+ sticky/overflow), Toast, Skeleton, EmptyState, ErrorState, StatusChip (`Uploaded/Processing/Ready/Failed`), **CitationChip**, **CitationCard**, **OriginalTextBlock**, **ClaimBlock** (4 variants), **SourceInspector**, ScopeSelector, FileDropzone, UploadRow, ProjectCard, PaperRow, ComparisonTable, ReviewEditor, VerificationBanner, AuthForm, PrivacyNotice, LimitsHint (reads configuration).

## 4. Workflow specifications

Format: Entry · Action · Response · Loading · Success · Failure · Recovery · A11y.

**4.1 First-time onboarding** — Entry: first login with zero projects. Action: complete or dismiss 3-step card (create project, upload, ask). Response: steps tick from real state. Loading: skeleton card. Success: card auto-hides when all done; dismiss persists. Failure: state fetch error → hide card, show nothing blocking. Recovery: reopen from Dashboard help link. A11y: card is a labelled region; steps are a list with status text.

**4.2 Create project** — Entry: "New project" (Dashboard/Projects). Action: title (required, ≤120), description (optional, ≤500). Response: dialog → on submit, navigate to project Papers tab with upload prompt. Loading: button loading. Failure: inline validation; server error → dialog-level error, input preserved. Recovery: retry. Duplicate titles allowed. A11y: focus first field; errors announced.

**4.3 Upload multiple PDFs** — Entry: dropzone or "Add papers". Action: drop/select N files. Response: each file becomes an UploadRow immediately. Loading: per-file upload progress (real), then status→Processing. Success: row becomes Ready with extracted title/authors/year/page count when available; fields show "—" otherwise (never guessed). Failure (per file, batch continues): reasons surfaced — not a PDF, over size limit, over page limit, project paper limit reached, corrupted, password-protected, no extractable text (scanned), duplicate, network. Limits shown in the UI come from configuration. Files wait in a controlled queue (shown as Uploaded) rather than all processing at once. The upload area shows the privacy caution notice (DESIGN_SYSTEM §6.13). Recovery: Retry (same file), Replace, Remove. Duplicate: dialog "Already in this project" → Skip / Keep both. A11y: dropzone is a button-equivalent; row status changes announced politely; progress via `role=progressbar`.

**4.4 Monitor processing** — Entry: Papers tab. Action: none (live). Response: statuses update without reload (polling or push — your choice), with stage text if available. Loading: Processing chip. Success: Ready. Failure: Failed + reason + actions. Recovery: Retry reprocesses without re-upload. Persisted across reloads. **[ASSUMPTION]** backend exposes `status`, optional `stage`, `failure_reason`.

**4.5 Ask a question** — Entry: Chat tab composer. Action: submit question; scope default whole project, narrowable. Response: streaming claim blocks with citations. Loading: "Searching papers…" → "Reading passages…" → streaming. Success: complete answer + sources + follow-ups + actions. Failure: (a) no evidence → NoEvidence card (spec §4.2.2); (b) retrieval/LLM error → error card, question preserved in composer; (c) stream interrupted → partial answer kept, "Incomplete", Retry. Recovery: retry, rephrase, broaden scope. Block send if no Ready papers (explain; link to Papers). A11y: live-region announcements on completion; Stop button keyboard reachable.

**4.6 Inspect citation** — Entry: click/Enter on CitationChip or card. Response: Source Inspector opens at that citation with highlighted span and context. Loading: inspector skeleton. Failure: "Passage unavailable — citation could not be verified" and the claim marked unverified in the thread. Recovery: retry load; "Open PDF at page". Esc closes; focus returns to chip.

**4.7 Compare papers** — see PRODUCT_SPEC §4.4. Failure: per-cell retry; whole-job failure shows retry with selection preserved. Missing data = "Not reported in the source".

**4.8 Generate literature review** — see PRODUCT_SPEC §4.5. Loading: sectioned progress. Failure: partial draft preserved if any sections completed, with failed sections marked. VerificationBanner always visible.

**4.9 Export** — Chat answer → Copy (text with numbered refs + reference list) / Markdown. Comparison → CSV. Review → Markdown + CSV. Export includes page numbers and source titles. Failure: toast with retry. Exports must **not** drop citations or labels (e.g., "Hypothesis") from content.

**4.10 Insufficient evidence** — covered in 4.5(a). Acceptance: no uncited factual claims in default style.

**4.11 Failed upload / network errors** — Offline banner; queued uploads pause and resume or show retry; unsent questions retained; no silent data loss.

## 5. Screen requirements (summary; sizing from DESIGN_SYSTEM)

| Screen | Key layout | Required data | API/backend dependencies |
|---|---|---|---|
| Auth (sign up/in, reset) | Centered card, reserved space for a future provider button, privacy link | email, password | standard auth provider; reset emails; sessions |
| Dashboard (M8, polish) | Greeting, New project CTA, recent projects/papers/conversations | recent projects (+paper count, updatedAt), recent papers (+status), recent conversations | list endpoints with `limit`, sort by updated |
| Projects | Search, sort (name, updated, paper count), grid/list | project list | CRUD, search, sort |
| Papers | Dropzone + table (title, authors, year, pages, status, added) + filters (status, text) + detail drawer | papers, metadata, status, failure reason | upload (resumable preferred), ingestion pipeline, retry, delete, dedupe |
| Chat | 4-region layout, scope bar, composer | conversations, messages as claim blocks w/ citation IDs, citations (paper, page, excerpt, span offsets) | streaming endpoint, retrieval, citation mapping, claim-label output, feedback endpoint |
| Source Inspector | Panel | citation → paper, page, passage + context, highlight offsets | passage fetch; PDF page viewer **[BLOCKER?]** |
| Compare | Table + attribute toggles | per-paper extracted attributes + citations + confidence | structured extraction job |
| Review | Inputs + editor + matrix | draft sections, citations, matrix | generation job, save draft |
| Search | Query + filters + results | passage hits (paper, year, page, snippet) | hybrid search |
| Settings | Account, Appearance (light only), Documents & data (storage list, delete all), Privacy (data use statement, export/delete account) | user profile | account endpoints |

## 6. Backend contracts the UI depends on (observable behavior)
1. **Citations must be real:** each citation references a stored chunk with paper ID, page, and verbatim text. The UI never receives citation text synthesized by the LLM.
2. **Answer structure:** list of blocks `{text, label ∈ {supported, synthesis, conflicting, hypothesis}, citationIds[]}`. Backend validates that `supported`/`synthesis` blocks have citations (synthesis ≥2 where possible); otherwise downgrade to hypothesis or drop.
3. **Insufficient evidence:** a retrieval-confidence threshold produces a distinct `no_evidence` result.
4. **Comparison cells:** `{value|null, citationIds[], reason?}`; null ⇒ "Not reported in the source".
5. **Page numbers:** include physical page index; printed label if extracted.
6. **Status model:** `uploaded|processing|ready|failed` + `stage?` + `failure_reason?`.
7. **Security/privacy:** per-user isolation of documents, text, chunks, and embeddings with server-side authorization on every request; private by default; no public sharing; deletion removes files, extracted text, chunks, and embeddings (report backup behavior). Do not claim provider training/retention practices in the UI unless verified.

## 7. Responsive, accessibility, performance requirements
- Follow breakpoints in DESIGN_SYSTEM §5; test at 360, 768, 1024, 1440 widths.
- AA contrast verified by automated check; keyboard-only run-through of all Core-tier flows.
- Targets **[ASSUMPTION — validate feasibility]:** first answer token < 3s typical; citation inspector opens < 300ms after click; upload table handles 100 papers without jank.
- Lists > 100 items paginate or virtualize.

## 8. Milestone 1: acceptance criteria
Goal: a user can sign up, create a project, upload PDFs, ask a question, and verify a citation, end to end with real data.
1. Email/password sign-up, sign-in, **password reset**, and **sign-out** work; sessions are secure.
2. **Server-side authorization:** automated test proves user B cannot read or modify user A's projects, papers, conversations, or passages.
3. Project CRUD with search/sort.
4. Upload ≥ 5 PDFs at once through the controlled queue; per-file progress; statuses correct; a deliberately broken file, a password-protected file, an oversize file, and a file exceeding the page limit each fail with a clear, specific reason; Retry works.
5. Limits are read from configuration; changing a config value changes behavior and UI copy without code edits.
6. Duplicate upload triggers the duplicate dialog.
7. Privacy notice and upload caution are visible; deletion of a paper really removes its data.
8. Chat answers questions across ≥ 3 papers with streaming, claim labels, inline citations, citation cards.
9. Clicking any citation shows the verbatim passage with highlight and page number.
10. Nonsense/out-of-scope question yields the NoEvidence card, not a fabricated answer.
11. Stream interruption leaves partial answer + Retry.
12. Tokens/components match DESIGN_SYSTEM (reviewed by Claude with screenshots).
13. Keyboard-only completion of upload and citation-verification flows; no AA contrast failures.

## 9. Spot-check protocol for citation faithfulness (recommended)
Create a small test set (e.g., 10 questions over 3 papers with known answers/pages). For each: confirm every citation's passage actually supports the block text. Record pass rate in this file. Citation accuracy is the product's core quality metric; do not ship M3 without measuring it.

## 10. Open dependencies and unresolved issues
- **[BLOCKER?]** PDF page rendering/viewer for "Open PDF at page": feasible in MVP? If not, Inspector shows extracted text only and the button is hidden (no dead controls).
- Cost estimate and final limits (§2.2).
- Feasibility of comparison/literature review at 20 papers (retrieval/extraction-based, not whole-paper prompts).
- Scanned PDFs: unsupported, flagged as Failed (D-007).
- Auth: provider choice; email verification and auth rate limiting are my assumption (PRODUCT_SPEC A8): confirm.
- Google sign-in: only if effort permits.
- Streaming + structured output for claim blocks (your design; must satisfy §6.2).
- Claim-label reliability: LLM-assigned labels are fallible; report observed accuracy so UI wording isn't overstated.
- Real deletion/backup/retention behavior and provider list, needed to write accurate privacy copy.
- Pre-public-launch items: PRODUCT_SPEC §2.5.
