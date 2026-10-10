# DESIGN_SYSTEM.md — Research Intelligence RAG

> Status: Draft v0.1.1 (Phase 3 proposal; adds notices pattern for privacy/limits). Token names are **binding vocabulary**; values are proposals pending owner approval.
> Contrast ratios below are **estimates computed by approximation, not tool-verified**. Antigravity must verify all text/background pairs with an automated contrast check (WCAG 2.2 AA) before release and report deviations.

## 1. Visual principles
1. **Evidence over ornament.** Nothing decorative competes with source text.
2. **Calm and focused.** Neutral surfaces, one restrained accent.
3. **Provenance is visible.** Original text, generated text, and hypotheses each have a distinct, consistent look.
4. **Density with air.** Efficient spacing; reading text never cramped.
5. **Color is never the only signal.** Status and claim types always include text and/or icon.

Avoid: gradients (beyond none), neon, glassmorphism, oversized radii, hero sections, decorative animation, fake stats.

## 2. Color tokens (light theme)

### Neutrals
| Token | Hex | Use |
|---|---|---|
| `--bg-app` | `#F7F7F9` | App background |
| `--bg-surface` | `#FFFFFF` | Cards, panels, inputs |
| `--bg-subtle` | `#F1F2F5` | Hover rows, code, table headers |
| `--bg-inset` | `#ECEEF2` | Pressed, inset areas |
| `--border-default` | `#E3E5EB` | Dividers, card borders |
| `--border-strong` | `#C9CDD6` | Input borders, emphasized dividers |
| `--text-primary` | `#1B1D24` | Body, headings (charcoal) |
| `--text-secondary` | `#555B6A` | Secondary text |
| `--text-muted` | `#6B7080` | Meta, placeholders (≥ AA on white; est. ~4.9:1) |
| `--text-disabled` | `#9AA0AE` | Disabled only (exempt from contrast, still legible) |

### Accent (indigo)
| Token | Hex | Use |
|---|---|---|
| `--accent-50` | `#EEF0FF` | Selected rows, soft backgrounds |
| `--accent-100` | `#DDE1FF` | Hover on soft |
| `--accent-500` | `#4F5BD5` | Icons, focus ring color |
| `--accent-600` | `#4048B8` | Primary button bg (white text est. ~6:1) |
| `--accent-700` | `#343B96` | Primary hover; link text |
| `--accent-800` | `#292F78` | Primary active |

### Semantic
| Role | Fg (text/icon) | Bg | Border |
|---|---|---|---|
| Success | `#1E6B3A` | `#E8F5EC` | `#B7DFC4` |
| Warning / Conflicting | `#8A5300` | `#FFF4E0` | `#F0D199` |
| Error | `#B3261E` | `#FDECEA` | `#F2B8B5` |
| Info | `#1F5FA8` | `#E8F1FB` | `#B5D0EE` |

### Provenance tokens (core to product)
| Token | Value | Use |
|---|---|---|
| `--source-bg` | `#FBFAF5` (warm paper tint) | "Original text" excerpts |
| `--source-border` | `#E4DFC9` | Left border of original text |
| `--source-highlight` | `#FFF1A8` | Cited span highlight (with underline, not color alone) |
| `--synthesis-border` | `--accent-500` | Synthesis block left rule |
| `--conflict-bg/border` | Warning pair | Conflicting block |
| `--hypothesis-border` | `--border-strong` dashed | Hypothesis block |

**Dark theme:** tokens are semantic and theme-swappable; dark values are **not** defined in MVP.

## 3. Typography
- **UI / sans:** `Inter, "Inter Variable", system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif`.
- **Original-text (source excerpts):** `"Source Serif 4", "Newsreader", Georgia, "Times New Roman", serif` — signals "this is quoted source text".
- **Mono:** `"JetBrains Mono", ui-monospace, SFMono-Regular, Menlo, Consolas, monospace`.
- Fonts self-hosted or system fallback; no layout shift on load (`font-display: swap` with size-adjust fallback).

| Token | Size/Line | Weight | Use |
|---|---|---|---|
| `text-display` | 28/36 | 600 | Page titles (rare) |
| `text-h1` | 22/30 | 600 | Page headings |
| `text-h2` | 18/26 | 600 | Section headings |
| `text-h3` | 15/22 | 600 | Card titles |
| `text-body` | 15/24 | 400 | Chat answers, body |
| `text-body-sm` | 14/20 | 400 | UI default |
| `text-caption` | 12/16 | 500 | Meta, chips, table headers |
| `text-source` | 15/24 serif | 400 | Original excerpts |

Reading width: chat answers `max-width: 72ch` (≈720–760px). Minimum body text 14px; no text below 12px.

## 4. Spacing, layout, sizing
- **Spacing scale (4px base):** 0, 4, 8, 12, 16, 20, 24, 32, 40, 48, 64. Tokens `--space-1…--space-16`.
- **Radii:** `--radius-sm 4`, `--radius-md 8` (default), `--radius-lg 12` (dialogs), `--radius-full` (chips/avatars). No larger.
- **Borders:** 1px `--border-default`. **Shadows:** `--shadow-1: 0 1px 2px rgba(20,24,40,.06)` (cards), `--shadow-2: 0 4px 12px rgba(20,24,40,.10)` (popovers), `--shadow-3: 0 12px 32px rgba(20,24,40,.16)` (dialogs). Prefer borders over shadows.
- **Grid:** 12 columns, 24px gutters ≥1024; 16px below.
- **Page widths:** content pages `max-width 1120`; settings `720`; chat thread `720–760`; tables full width with horizontal scroll container.
- **Shell:** global sidebar 248 (collapsed 64); conversations list 240; Source Inspector 400 (resizable 320–560); top bar 56.
- **Icons:** one set (Lucide recommended **[ASSUMPTION]**), stroke 1.5, sizes 16 / 20 / 24. Icon-only buttons need accessible names and tooltips.
- **Hit targets:** ≥ 32px desktop, ≥ 44px touch.

## 5. Breakpoints
| Name | Range | Behavior |
|---|---|---|
| `sm` | < 640 | Single column; bottom tab bar (Projects, Chat, Papers, More); inspector = full-screen sheet |
| `md` | 640–1023 | Sidebar collapsed to icons; conversations list in a drawer; inspector = overlay drawer |
| `lg` | 1024–1439 | Sidebar + thread; inspector docked, collapsible |
| `xl` | ≥ 1440 | Full 4-region chat layout |

Mobile is designed deliberately: comparison tables transpose to per-paper cards with an attribute switcher; upload uses the system file picker as primary; composer sticks to the bottom above the keyboard.

## 6. Component specifications
All components: visible focus (`2px` ring `--accent-500` with `2px` offset, ≥3:1 against adjacent colors), keyboard operable, ARIA per WAI-ARIA Authoring Practices.

**Standard state matrix** (applies to buttons, inputs, rows, chips unless noted)
| State | Treatment |
|---|---|
| Default | Per spec |
| Hover | One step darker/tinted bg (`--bg-subtle` / `--accent-700`) |
| Focus-visible | Focus ring as above |
| Active | `--bg-inset` / `--accent-800` |
| Disabled | `--text-disabled`, no hover, `aria-disabled`; explain why via tooltip where non-obvious |
| Loading | Spinner replaces leading icon, label kept, width stable, `aria-busy` |
| Success | Brief check icon (≤2s) + toast if non-obvious |
| Error | `--error` border + message below with icon; `aria-describedby` |

### 6.1 Buttons
Primary (accent-600, white text), Secondary (surface, border-strong), Ghost (no border), Destructive (error-fg; confirm dialog always). Heights 32 (sm) / 36 (md) / 40 (lg). One Primary per view region.

### 6.2 Inputs
Text, textarea, select, checkbox, radio, switch, file input. Height 36; label above, helper below; error replaces helper. Search input has clear button and `/` shortcut hint.

### 6.3 Dialogs and drawers
Dialog `radius-lg`, width 480 (confirm) / 640 (form); focus trap; Esc closes (unless destructive in progress); returns focus to trigger. Drawers right-side 400–560, same rules. Destructive confirms name the object ("Remove *Smith2021.pdf*? Its citations in 3 conversations will become unavailable.") — **[ASSUMPTION]** backend can report affected counts.

### 6.4 Dropdowns, tooltips
Menus: roving tabindex, type-ahead. Tooltips on hover **and** focus, 500ms delay, never the only carrier of essential info.

### 6.5 Tabs
Underlined tabs for project nav; `role=tablist`, arrow-key navigation, selected has 2px accent underline + `--text-primary`.

### 6.6 Tables
Header `--bg-subtle`, caption-size, sticky header and first column in comparison. Row height 44 (default) / 56 (with metadata). Zebra off; hover `--bg-subtle`. Cell overflow: clamp 4 lines + "Show more". Use semantic `<table>`, `scope` attributes. Missing value = "Not reported in the source" (`--text-muted`, italic).

### 6.7 Citation components (most important)
1. **Citation chip (inline):** `[n]` numeric, `--accent-50` bg, `--accent-700` text, radius-sm, 20px tall, min width 24. Hover/focus: tooltip "Author Year · p. N". Click: opens Source Inspector on that citation. `aria-label="Citation 3: Smith 2021, page 7"`. Multiple per claim allowed (`[2][5]`).
2. **Citation card:** surface, border, padding 12. Title (h3, 1-line clamp), meta (authors · year · "p. 7"), excerpt (`text-source`, 3-line clamp, on `--source-bg`), actions: Open in inspector, Copy citation. Status: *Verified passage* / *Passage unavailable* (warning).
3. **Original-text block:** `--source-bg`, 3px left border `--source-border`, serif, label "Original text" (caption, muted, with quote icon). Cited span highlighted with `--source-highlight` + underline.
4. **Claim block labels:** tag chips — `Source-supported` (check icon, success tone, shown subtly), `Synthesis` (layers icon, accent), `Conflicting` (alert icon, warning, block tinted), `Hypothesis — unverified` (flask icon, neutral, dashed border). Always icon + text.
5. **Source Inspector:** header (title, authors, year, close/collapse), passage viewer, citation list navigator ("3 of 5", prev/next, `J/K` shortcuts **[ASSUMPTION]**), footer actions (Open PDF at page, Copy citation). Collapsible; state persists. On `sm` becomes full-screen sheet.

### 6.8 Document status indicators
| Status | Icon | Tone | Text | Extra |
|---|---|---|---|---|
| Uploaded | upload-cloud | neutral | "Uploaded" | Queued, not yet processing |
| Processing | spinner (reduced-motion: static dots) | info | "Processing" + stage (e.g., "Extracting text") if backend supplies it | Determinate progress only if real |
| Ready | check-circle | success | "Ready" | |
| Failed | alert-circle | error | "Failed" | Reason + "Retry" / "Replace" / "Remove" |
Also: a **Duplicate** notice (info) on upload with choices "Skip" / "Keep both" — **[ASSUMPTION]** duplicate detection by content hash. Upload progress (client→server) is separate from processing (server pipeline); show both distinctly.

### 6.9 Toasts
Bottom-left, max 3 stacked, 5s auto-dismiss (errors persist until dismissed), `role=status` / `role=alert` for errors, with Undo where possible.

### 6.10 Loading skeletons
Match final layout, subtle pulse (disabled under `prefers-reduced-motion`). Streaming answers show a text caret, not a skeleton.

### 6.11 Empty states
Icon (24px, muted) + one-sentence explanation + one primary action. No illustrations required. Copy states what's missing and the next step.

### 6.12 Error states
Inline (field), section (card with retry), page (full area with retry + support text), offline banner (top, persistent). Always say what happened, whether data is safe, and what to do next. No raw error codes in primary text (available under "Details").

### 6.13 Notices (privacy, limits, verification)
Inline notice component: tone Info or Warning, icon + text, optional link, optional dismiss (persistent notices are not dismissible). Used for: upload privacy caution and limits hint (shown at the upload area), third-party AI processing notice, and the literature-review verification banner (not dismissible). **Limit values (size, pages, papers per project) are never hardcoded; they come from configuration** (DECISIONS_LOG D-014). Auth screens: single centered card (max 400), email/password fields, provider buttons area reserved for a future Google button, links to privacy notice.

## 7. Interaction patterns
- Motion: 120–200ms ease-out for panel open/close and hover; none for content updates; respect `prefers-reduced-motion`.
- Streaming: append-only, scroll anchoring pauses if user scrolls up ("Jump to latest" pill).
- Destructive actions: confirm dialog; Undo toast for reversible ones (e.g., chat delete).
- Selection: checkboxes on rows; sticky action bar appears for bulk actions.
- Keyboard: `/` focus search, `Cmd/Ctrl+K` command palette (post-MVP **[ASSUMPTION]**), `Esc` closes panels, Enter sends / Shift+Enter newline.

## 8. Accessibility standards
- WCAG 2.2 AA target. Text contrast ≥ 4.5:1; large text and UI components ≥ 3:1.
- Full keyboard operation; logical focus order; focus not trapped except in modals.
- Landmarks: `nav`, `main`, `aside` (inspector), skip-to-content link.
- Streaming answers: live region `aria-live="polite"` announces completion, not each token.
- Claim types and statuses conveyed via text/icon, not color alone.
- Respect reduced motion, 200% zoom, and text-resize without loss; reflow at 320px.
- Tables: proper headers; transposed mobile view keeps header associations.
- Drag-and-drop upload always has a keyboard-accessible button equivalent.

## 9. References
No external visual references approved yet. Reference products/screenshots go here once the owner approves a direction; until then this document is authoritative.
