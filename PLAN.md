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

---

## Roadmap

### Phase 0 — Spike (current)

Risky-assumption validation. Throwaway. See above.

### Phase 1 — Core foundations (`u1-docs`)

- `u1-font` — system font enumeration, matching, fallback, cache
- `u1-layout` — inline text layout via `parley`, then block/flow/page layout
- `u1-doc` — document model: blocks, inlines, formatting, sections
- Incremental layout: typing stays at 60fps in a 500-page document

### Phase 2 — OOXML (`u1-docs`)

- OPC container layer (`zip` + relationships + content types)
- WordprocessingML reader producing **model + preserved original tree**
- Writer that splices edits back into the preserved tree — lossless
- Headers/footers, sections, page setup, styles, numbering, tables

See [ADR-0005](docs/adr/0005-ooxml-lossless-layer.md). The lossless property is
non-negotiable; it is what stops us corrupting user files.

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
