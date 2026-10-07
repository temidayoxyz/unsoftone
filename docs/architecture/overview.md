# Architecture overview

How the ecosystem fits together. Product-specific architecture lives in each
product repository; this document covers only what is genuinely shared.

Read the [ADRs](../adr/) for *why*. This document covers *what*.

---

## The shape

```
┌──────────────────────────────────────────────────────────────┐
│  UnsoftOne ecosystem                                         │
│                                                              │
│  Shared: direction, design language, proven infrastructure    │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │ U1 Docs  │  │U1 Sheets │  │ U1 Slides│  │ U1 Notes │     │
│  │          │  │          │  │          │  │          │     │
│  │ own repo │  │ own repo │  │ own repo │  │ own repo │     │
│  │ own rel. │  │ own rel. │  │ own rel. │  │ own rel. │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
└──────────────────────────────────────────────────────────────┘
```

Independent applications. Independent releases. One direction.

---

## Layers within a product

Every U1 product has the same five layers, in this order. They are layered so
that the expensive, hard-to-change decisions sit at the bottom.

```
┌─────────────────────────────────────────────┐
│ 5. Shell                                    │  windows, menus, chrome,
│    (UI stack — ADR-0003)                    │  IME anchor, a11y plumbing
├─────────────────────────────────────────────┤
│ 4. Editing surface                          │  caret, selection, commands,
│                                             │  undo/redo, clipboard
├─────────────────────────────────────────────┤
│ 3. File I/O                                 │  DOCX / XLSX / PPTX, CSV,
│    (ADR-0005)                               │  PDF import, PDF export
├─────────────────────────────────────────────┤
│ 2. Document model + Layout engine           │  blocks, inlines, sections,
│    (ADR-0002)                               │  inline layout → page layout
├─────────────────────────────────────────────┤
│ 1. Foundations                              │  fonts, shaping, geometry,
│                                             │  undo engine, atomic save
└─────────────────────────────────────────────┘
```

**The rule:** lower layers never depend on upper ones. Layout does not know
about the shell. The document model does not know about DOCX. This is what makes
the core testable without a GUI and replaceable without a rewrite.

The hard part is layer 2, and specifically the **block/flow/page layout** above
`parley`. That layer is ours, and it is the largest single engineering item in the
project.

---

## What gets shared, and when

Extracted into a `u1-core` repository **only after two products need the same
thing for real**. Per [ADR-0004](../adr/0004-repository-model.md) and
[ADR-0008](../adr/0008-product-sequencing-u1-docs-first.md).

Candidates, in rough order of how likely they are to be genuinely shared:

| Component             | Why it is a real candidate                          |
| --------------------- | --------------------------------------------------- |
| `u1-font`             | identical need; enormous complexity; hard to get right |
| `u1-layout`           | Docs, Slides and Notes all lay out rich text        |
| `u1-ooxml` (OPC)      | zip + content types + relationships; Docs/Sheets/Slides |
| `u1-pdf`              | PDF export in three products                        |
| `u1-undo`             | the *engine* only — never the commands              |
| `u1-a11y`             | accessibility plumbing                              |
| `u1-atomic-save`      | same correctness problem everywhere                 |

**Explicitly not shared:** UI components across applications. Design *language*
is shared; chrome code is not. Product chrome diverges, and forcing it into a
shared component library is how ecosystems become slow to change.

---

## The Rust stack

Verified against crates.io on 2026-10-07.

| Concern            | Crate                              | Notes                                |
| ------------------ | ---------------------------------- | ------------------------------------ |
| Inline text layout | `parley` v0.11                     | ADR-0002. WPT-validated              |
| Font access        | fontique, skrifa (via parley)      |                                      |
| Shaping            | harfrust (via parley)              | HarfBuzz port                        |
| OOXML container    | `zip` v8.6                         | OPC package layer                    |
| OOXML parsing      | `quick-xml` v0.42                  |                                      |
| Rendering          | platform webview (ADR-0003)        | pending spike                        |
| PDF writing        | `pdf-writer` v0.15                 | plus font embedding                  |
| Clipboard          | `arboard` v3.6                     |                                      |
| Accessibility      | `accesskit` v0.25                  | if the native path is ever taken     |
| Windowing          | `tao` v0.37                        |                                      |

These are *starting positions*, re-verified at implementation time — not pins.
Version numbers drift; the reasoning in the ADRs does not.

---

## Testing strategy

A layered architecture demands a layered test strategy.

| Layer              | How we test it                                        |
| ------------------ | ----------------------------------------------------- |
| Foundations (1)    | unit tests; property tests for atomic save and undo    |
| Layout (2)         | golden-image tests; Web Platform Tests via `parley`   |
| File I/O (3)       | round-trip corpus — **the critical suite**             |
| Editing surface (4)| integration tests over synthetic documents             |
| Shell (5)          | minimal; manual, plus the spike                        |

### The round-trip corpus is the most important test suite in the project

It is a corpus of real-world documents — messy, hand-edited, produced by many
versions of Word and LibreOffice — where the assertion is:

> open(x) → save → open again, and the second document is semantically
> identical to the first, with no lost content.

This is the direct test of [ADR-0005](../adr/0005-ooxml-lossless-layer.md), and it
is what stops silent data loss. It should grow as a living collection of real
files, and it is the single best argument a bug reporter can make when submitting
a document.

---

## Cross-cutting constraints

These apply to every product and are enforced in review:

1. **No network in core code paths.** [ADR-0007](../adr/0007-local-first-is-not-negotiable.md)
2. **No account, telemetry or activation.** Same ADR.
3. **Lossless file handling.** [ADR-0005](../adr/0005-ooxml-lossless-layer.md)
4. **Layering is one-directional.** Lower layers never depend on upper ones.
5. **Accessibility is not a post-process.** Semantics are part of the model, not
   something bolted onto the shell at the end.
