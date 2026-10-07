# File-format compatibility

The compatibility target, tracked honestly.

"Compatible with Office" is not a claim we can make or verify. This document
converts it into something we can actually measure and improve, one row at a
time.

---

## Legend

| Symbol | Meaning                                                       |
| ------ | ------------------------------------------------------------- |
| ●      | supported, with the behaviour users expect                     |
| ◐      | partial — common cases work; edge cases are lossy or missing  |
| ○      | read-only                                                     |
| ·      | not started                                                   |

**Lossless** means: open → save → open yields the same document with no content
lost. This is the binding requirement, not the highest level of feature support.
See [ADR-0005](adr/0005-ooxml-lossless-layer.md).

---

## U1 Docs

| Format | Read | Write | Lossless | Notes                                  |
| ------ | ---- | ----- | -------- | -------------------------------------- |
| `.docx` | ·   | ·     | ·        | Primary target. The whole project.      |
| `.pdf`  | ·   | ·     | n/a      | Export is the realistic goal            |
| `.txt`  | ·   | ·     | n/a      | Trivial                                |
| `.md`   | ·   | ·     | n/a      | Not a substitute for `.docx`            |
| `.rtf`  | ·   | ·     | ·        | Only if Word compatibility demands it   |
| `.odt`  | ·   | ·     | ·        | After DOCX is solid                     |
| `.doc`  | ·   | ·     | ·        | Binary format. Very large surface.      |
| `.html` | ·   | ·     | n/a      | Import only, realistically              |

### `.docx` feature targets

Ordered by how often they matter to a real user. This ordering is deliberate:
it is the definition of "deeply" in
[ADR-0008](adr/0008-product-sequencing-u1-docs-first.md).

| Feature                                | Target | Why it matters                                  |
| -------------------------------------- | ------ | ----------------------------------------------- |
| Paragraphs, runs, character formatting | ●      | The core of the product                         |
| Fonts, sizes, bold/italic/underline     | ●      | The core of the product                         |
| Paragraph alignment, indentation        | ●      | Every document                                   |
| Lists and numbering                     | ●      | Every document                                   |
| Styles (named + linked)                 | ●      | Every real document uses them                   |
| Tables                                  | ●      | Ubiquitous in real documents                     |
| Page setup, margins, sections           | ●      | Required for pagination                          |
| Headers and footers                     | ●      | Extremely common                                |
| Images (inline + floating)              | ●      | Extremely common                                |
| Hyperlinks                              | ●      | Common                                          |
| Comments                                | ◐      | Common in collaborative documents                |
| Track changes                           | ○      | Must not destroy; editing them is much later    |
| Footnotes / endnotes                    | ○      | Common in academic and legal documents          |
| Text boxes and shapes                   | ○      | Must not destroy                                |
| Equations                               | ○      | Must not destroy                                |
| Content controls                        | ○      | Must not destroy                                |
| `w:compat` legacy layout flags         | ◐      | Silent layout changes; must be preserved verbatim |

The "must not destroy" rows matter more than they look. A feature we cannot
*edit* is a missing feature. A feature we cannot edit but *do destroy* is a data
loss bug — the worst category of problem this project can have.

---

## U1 Sheets

Not started. Prior research only — see below.

| Format   | Read | Write | Lossless | Notes                                          |
| -------- | ---- | ----- | -------- | ---------------------------------------------- |
| `.xlsx`  | ·   | ·     | ·        | Primary target                                 |
| `.csv`   | ·   | ·     | n/a      | Easy; encoding detection is the fiddly part    |
| `.tsv`   | ·   | ·     | n/a      |                                                |
| `.ods`   | ·   | ·     | ·        | After XLSX                                     |
| `.xls`   | ·   | ·     | ·        | Binary format                                  |

### Prior research (not commitments)

The ecosystem is more helpful here than for DOCX:

- `calamine` v0.36 — reads XLSX and ODS
- `rust_xlsxwriter` v0.99 — writes XLSX
- `umya-spreadsheet` v3.1 — reads and writes XLSX

None is a lossless round-trip editor. Two problems dominate and both are
projects in their own right:

1. **Formula recalculation on open.** Excel's function set is enormous, and
   files routinely store cached values rather than recalculated ones. Deciding
   what a spreadsheet does with a stale cache is a correctness problem before it
   is an engineering problem.
2. **Number formats and dates.** Serial-date systems, custom format codes and
   locale interaction. A deep, low-glamour rabbit hole that users notice
   immediately when it is wrong.

---

## U1 Slides

Not started.

| Format   | Read | Write | Lossless | Notes                        |
| -------- | ---- | ----- | -------- | ---------------------------- |
| `.pptx`  | ·   | ·     | ·        | Primary target               |
| `.pdf`   | ·   | ·     | n/a      | Import realistic; export useful |
| `.odp`   | ·   | ·     | ·        | After PPTX                   |
| `.ppt`   | ·   | ·     | ·        | Binary format                |

**PPTX is expected to be the hardest of the three.** It is not primarily a text
problem — it is mostly floating shapes at absolute coordinates, z-order,
autofit text behaviour and theme inheritance. This is why
[PLAN.md](PLAN.md) does not commit to a date for U1 Slides.

---

## U1 Notes

Not started.

| Format  | Read | Write | Lossless | Notes                                     |
| ------- | ---- | ----- | -------- | ----------------------------------------- |
| `.md`   | ·   | ·     | n/a      | Likely the native format                   |
| `.html` | ·   | ·     | n/a      | Markdown-compatible notes are portable    |
| `.docx` | ·   | ·     | ·        | Only if notes ever need Word interchange  |

U1 Notes exists to be the lightest member of the family. It should be the
product most likely to become genuinely good quickly, and the least likely to
need the parts of the architecture that are hardest to build.

---

## How this document is maintained

Update the markers when a format's status genuinely changes — not when work
starts. A row that says ● before it is true is worse than a row that says ·,
because it erodes the only honest signal this document has.

The **round-trip corpus** (see [architecture/overview.md](architecture/overview.md))
is what turns these markers from claims into facts. When it exists, this table
becomes its own evidence.
