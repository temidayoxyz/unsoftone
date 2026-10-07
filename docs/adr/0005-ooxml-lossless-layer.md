# ADR-0005: A self-built, lossless OOXML layer

- Status: Accepted
- Date: 2026-10-07

## Context

File-format compatibility is not a feature to be added at the end. It is the
project. "A serious word processor" means opening, editing and saving files that
users already have, without damaging them.

WordprocessingML is genuinely hostile to naive handling:

- it is enormous — thousands of elements, versioned across many revisions
- it is full of legacy compatibility flags (`w:compat`) that silently **change
  layout**, so identical content can render differently depending on settings
  Word wrote years ago
- features are expressed through combinations of properties with implicit
  precedence
- Word silently "repairs" files it considers malformed — meaning anything we
  emit that Word dislikes can be rewritten on the user's next save

The failure mode this ADR exists to prevent: **parse into our own model, then
re-serialise from that model.** That approach looks clean and it corrupts
documents. Any element we did not fully understand is dropped on save, and the
user loses content they never knew was at risk.

We surveyed the ecosystem. The existing crates do not solve this:

- `docx-rs` — a *writer* aimed at templating (Rust/WASM). Not an editor.
- `docx-rust`, `office_oxide` (v0.1.x) — early, and not lossless round-trip.
- `calamine` — reads XLSX, does not write.
- `rust_xlsxwriter`, `umya-spreadsheet` — write XLSX; writers, not round-trip.

**We are building the OOXML layer ourselves.**

## Decision

**Build a lossless OOXML layer directly on `quick-xml` + `zip`.**

The core property is **preservation**:

> Parsing produces (a) our own editing model, and (b) a faithful
> representation of the original part tree.
>
> Saving splices our edits back **into the preserved tree**. Anything we do not
> understand is passed through byte-for-byte unchanged.

The editing model is a *view over* the document, never a replacement for it. A
feature we have not implemented yet is not a feature we destroy — it is content
we carry through untouched.

### Consequences

**Easier:** opening and saving a complex document we only partially support does
not lose the parts we do not support; we degrade gracefully per-feature instead
of failing per-file; `w:compat` settings and unknown extension elements survive
untouched; Word's "repair" prompt never fires because we never rewrite parts we
did not need to.

**Harder:** the model and the tree must be kept in sync deliberately; every edit
path must go through the splice layer; debugging is harder because you are often
looking at XML, not your model; a discipline that must be enforced in review, or
it will be quietly abandoned for something cleaner and lossy.

**Forecloses:** a clean, normalised internal document model as the sole source of
truth. The duplication is a deliberate, permanent tax — it is the price of not
losing user data.

### Non-negotiable

If losslessness is ever traded away for simplicity, that trade is a regression
worse than a missing feature. A missing feature is visible; silent data loss is
not.

## Consequences for sequencing

This is why [PLAN.md](../PLAN.md) Phase 2 builds the OOXML layer **before** the
full editing surface in Phase 3. If the lossless model cannot be built, we need
to know at Phase 2 — not after users have trusted us with their files.

## Alternatives considered

**Use an existing crate.** Rejected: none provides lossless round-trip editing.
Adopting a lossy one would make the most damaging class of bug the default.

**Parse to a model and re-serialise.** Rejected: this is precisely the
corruption failure mode described above.

**Convert to HTML/Markdown as an intermediate.** Rejected: lossy in exactly the
dimensions that matter — pagination, styles, section breaks, numbering, tables.

**Delegate to LibreOffice headless for conversion.** Rejected as a core
dependency: it introduces a C++ dependency, a process boundary, licensing
questions, and a fidelity ceiling we cannot control. Might be a useful optional
import filter much later; it cannot be the foundation.

**Use an external service.** Rejected: violates the product philosophy
([ADR-0007](0007-local-first-is-not-negotiable.md)).
