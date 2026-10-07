# PLAN

The single source of truth for **what is being built, in what order, and what is
deliberately being ignored.**

If a change should not be happening yet, it belongs in [Not now](#not-now).
That section is load-bearing — it is what stops scope creep from quietly
consuming the project.

Last updated: 2026-10-07

---

## The problem

Office is a multi-decade, multi-team undertaking. LibreOffice is ~10M lines of
C++ developed over 25+ years, and its DOCX fidelity is still imperfect. We are
starting from zero in Rust, across four products.

**Calibration:** this is a direction, not a schedule. Building a genuine
alternative to Microsoft Office is a multi-year effort for a team. A solo effort
gets a meaningful subset. We say so plainly rather than promising parity.

---

## The wedge

**U1 Docs that reliably opens, edits and saves the DOCX files people actually
have, and exports clean PDF.**

Not "all of Word." Not every format. The files that show up on real desktops.

---

## Current phase — Phase 0: prove the risky assumptions

The two questions below gate the entire architecture. Both are cheap to answer
and ruinously expensive to answer late.

1. **Can we lay out a real Word document at interactive speed** with correct
   shaping, bidirectional text, and CJK? (~2 weeks)
2. **Can we get a caret, IME, and text selection working** in the chosen UI
   stack? (~2 weeks)

These run as a throwaway spike in `u1-docs`. The spike is deleted afterwards;
only the findings are kept. See [ADR-0003](docs/adr/0003-desktop-ui-stack.md).

**Exit criteria:** a document with RTL, CJK, tables and images scrolls and edits
smoothly, with a blinking caret and working CJK input method.

If (2) is ugly in a webview, we find out now — not in six months.

### Spike A — inline text layout: complete

**[ADR-0002](docs/adr/0002-text-layout-parley.md) holds.** Findings:
[`u1-docs/docs/spike-a-inline-layout.md`](https://github.com/temidayoxyz/u1-docs/blob/main/docs/spike-a-inline-layout.md)

Measured: 212 system font families discovered; bidi correct; worst-case
per-paragraph re-layout **54µs**; a visible page re-breaks in **2.66ms against a
16.7ms frame budget**.

Four requirements came out of it and now shape Phase 1:

| # | Requirement | Why it matters |
| - | ----------- | -------------- |
| 1 | Enable `parley/complex-scripts` (not a default feature) and declare `icu_segmenter` directly with `auto` | Without it, CJK and Thai **cannot break lines at all** — no word spaces means no break opportunities |
| 2 | Ship our own per-script, per-platform fallback policy | Default fallback produced `.notdef` for ordinary Chinese characters. A fallback that silently returns a partial-coverage font is worse than none |
| 3 | Verify glyph coverage and report gaps | Nothing currently tells the user a document opened with missing glyphs |
| 4 | Layout must be lazy and cached | Cold layout is 2.58ms/paragraph; only re-break on the typing, resize and scroll paths |

Criterion 1 (layout) is satisfied for the inline layer. The block/flow/page layer
above it does not exist yet and remains the largest engineering item in U1 Docs.

### Spike B — caret, hit testing, IME: criterion 2 done, criterion 3 not testable here

Findings: [`u1-docs/docs/spike-b-caret-and-ime.md`](https://github.com/temidayoxyz/u1-docs/blob/main/docs/spike-b-caret-and-ime.md)

**Criterion 2 (caret, selection, navigation) passes.** Caret geometry, hit
testing and arrow-key navigation through mixed-direction text are verified
across all 12 corpus samples, and every conclusion is enforced by tests. Bidi is
correct: in RTL text the caret's logical start sits on the *right* of its logical
end, which is what makes typing land where the user expects.

**Criterion 3 (CJK IME) was NOT verified.** This machine has no input method
registered — no CTF TIP entries, US English keyboard layout only — so composing
Japanese is impossible by construction. Everything the IME *depends on* is
confirmed (anchor rects at every position, safe insertion offsets, geometry
stable across re-break). What remains unproven is whether a webview delivers
correct composition, and that can only be settled by hand. An 8-step protocol is
in the findings doc.

Three findings, all of which change Phase 1:

| # | Requirement | Why |
| - | ----------- | --- |
| 1 | `u1-layout` needs a line-offset table (prefix sums), not per-query scanning | `LineMetrics.offset` is a *horizontal* alignment offset. Reading it as y made vertical navigation a silent no-op: every line reported y=0, so Up/Down did nothing, with no error |
| 2 | **Grapheme segmentation via `unicode-segmentation` is mandatory** before any insertion ships | parley clusters are per-*codepoint*: `e` + combining accent is 2 clusters. A caret walk over raw cluster edges puts a stop inside a visible character, and typing there orphans the mark |
| 3 | Caret geometry is a first-class layout output, not a rendering concern | it is what the IME anchor consumes, so it cannot live in the shell |

All three are **now implemented**. Spike B is 16/16 and exits zero:

- `CaretMap::line_tops` — prefix-sum table, O(1) lookup, invalidated on rebreak
- `CaretMap::grapheme_caret_positions` — real UAX #29 via `unicode-segmentation`.
  A ZWJ family emoji went from 4 caret stops to 2; the deliberately-failing GB11
  check is now a passing one, so it cannot be quietly closed by weakening the
  assertion
- `CaretMap` is a public library type, so caret geometry is a layout output by
  construction

### The IME protocol is now runnable

```bash
cargo run -p ime-protocol
```

Opens a window with three samples (ASCII, Arabic-inside-English, Chinese), a live
composition event log, and the 8 steps on screen. Every caret position and anchor
coordinate is computed by the Rust layout engine and injected as plain numbers —
the page never asks the browser where anything is. That is ADR-0003's entire bet,
made inspectable.

The harness is a **separate workspace crate** (`spikes/ime-protocol`), not a
feature flag. A feature flag looked equivalent and was not: `cargo
clippy --all-features` in the default CI job switched the webview back on, so
`u1-docs` stopped being webview-free in practice while still appearing to be. A
separate crate makes the boundary structural, and CI now asserts it.

**Criterion 3 is still open** — only a human with a CJK IME can close it. But it
now takes about five minutes instead of requiring a build first.

**ADR-0003 remains provisional** until the protocol is run and steps 3 and 4 pass.

---

## Roadmap

### Phase 0 — Spike (current)

Risky-assumption validation. Throwaway. See above.

### Phase 1 — Core foundations (`u1-docs`)

- `u1-font` — system font enumeration, matching, fallback, cache
  - **explicit per-script, per-platform fallback policy** (Spike A finding 3)
  - **coverage verification with gap reporting** (finding 3)
- `u1-layout` — inline text layout via `parley`, then block/flow/page layout
  - **incremental by default: re-break cached layouts, never rebuild the
    document on the typing, resize or scroll path** (Spike A finding 4)
  - `complex-scripts` + `icu_segmenter/auto` are mandatory and test-enforced
    (Spike A finding 1)
  - **line-offset table via prefix sums**, not per-query scanning
    (Spike B finding 1)
  - **grapheme segmentation via `unicode-segmentation`**, mandatory before any
    insertion path ships; parley clusters are per-codepoint and a caret stop
    inside a combining sequence produces mojibake (Spike B finding 2)
  - **caret geometry is a first-class layout output**, consumed by the IME
    anchor — not a rendering concern (Spike B finding 3)
- `u1-doc` — document model: blocks, inlines, formatting, sections
- Incremental layout: typing stays at 60fps in a 500-page document

### Phase 2 — OOXML (`u1-docs`)

- ✅ OPC container layer (`zip` + relationships + content types) — **done**
- ✅ Lossless XML tree (hand-written; preserves bytes, not just meaning) — **done**
- ⬜ WordprocessingML semantics: paragraphs, runs, styles, numbering, tables
- ⬜ Headers/footers, sections, page setup
- ⬜ Writer that splices edits back into the preserved tree

See [ADR-0005](docs/adr/0005-ooxml-lossless-layer.md). The lossless property is
non-negotiable; it is what stops us corrupting user files.

#### What Phase 2 step 1 established

`u1-docs` opens a real `.docx`. A document opened and saved with no edits comes
back with **every part byte-for-byte identical**, and repeated round trips are
idempotent — 12 tests, stable across runs.

The bar is byte-identity rather than semantic equivalence on purpose. Re-serialising
XML "canonically" changes bytes without changing meaning, and the moment Word
meets something it does not recognise it silently repairs the file on the user's
next save. So the XML tree keeps each node's original source text and reproduces
it: attribute order, quote style, whitespace, comments, CDATA and namespace
prefixes all survive.

Round-trip tests were written **before** the implementation, so the requirement
is stated independently. A test written afterwards tends to assert whatever the
reader happens to do, which is how a lossy implementation gets certified as
correct.

Still missing, and it is the substantive part: WordprocessingML *semantics* —
paragraphs, runs, styles, numbering, tables. The container and the tree had to be
exactly right first, because everything above assumes a round trip does not touch
what it did not edit.

### Phase 3 — Editing surface (`u1-docs`)

- Caret, selection (mouse + keyboard), navigation
- Undo/redo (engine shared across products later)
- Clipboard: internal + rich text (HTML) + plain text interop
- Find & replace

### Phase 4 — PDF export (`u1-docs`)

High value, moderate cost — we already own the layout output. `pdf-writer` plus
font embedding. PDF is the universal "it just works" format.

### Phase 5 — Packaging

Installers for Windows, macOS, Linux. No account, no activation, no network
calls in any core code path.

### Phase 6 — Hardening

Accessibility (AccessKit or platform-native), i18n, performance, crash
recovery, autosave.

### Phase 7+ — Other products

U1 Sheets, U1 Slides and U1 Notes begin **only after U1 Docs is genuinely
working**, and each is designed against the real lessons from the first.

---

## Not now

Explicitly out of scope until the phases above demand it.

| Item                     | Why not yet                                                   |
| ------------------------ | ------------------------------------------------------------- |
| Accounts, sign-in        | Contradicts the product philosophy. Never a core requirement. |
| Cloud storage / sync     | Same. Optional later, never required.                          |
| Collaboration            | Requires accounts. Revisit after the local product is solid.  |
| Auto-updates             | Requires a network path. Later, and optional.                  |
| Plugin system / ABI      | Cannot design an extension ABI before the core API stabilises. |
| Shared `u1-core` crate   | No shared surface has been *proven* yet. Extract when it hurts. |
| Cross-repo version mgmt  | Nothing is shared across repos today. Premature.               |
| Web build / mobile       | Desktop-first. Not a goal.                                     |
| ODF (`.odt`/`.ods`)      | Real demand, but strictly after OOXML is solid.                |
| Legacy `.doc`/`.xls`     | Binary formats, enormous surface. Much later.                  |
| DOCX macro / VBA support | Security surface. Effectively never.                           |

**Rule:** moving something out of this table requires moving it into the roadmap
and saying why. Silently adding scope is how this project dies.

---

## Sequencing principle

**One product, deeply, before four products shallowly.**

Designing four products in parallel produces four shallow applications. Shared
abstractions should be *earned* by hitting the same real problem twice — not
guessed at up front, when they will be wrong.

---

## Shared infrastructure policy

When code is shared, it will be the things that are both **difficult** and
**genuinely identical** across products:

- font management, shaping, layout
- the OOXML/OPC package layer (shared by Docs, Sheets and Slides)
- PDF generation
- clipboard format negotiation
- the undo/redo *engine* (mechanism, not commands)
- accessibility plumbing, atomic save, autosave/recovery

Design tokens, iconography and keyboard behaviour are shared as a **design
language**, not as code. See [ADR-0004](docs/adr/0004-repository-model.md).

---

## License

GPL-3.0 — see [ADR-0006](docs/adr/0006-license-gpl-3.md).
