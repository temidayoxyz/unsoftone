# Contributing

Thank you for wanting to help. This project is early, and honest help is worth
more than large contributions.

---

## Before you write code

1. **Read [`PLAN.md`](PLAN.md).** If your change belongs in "Not now", it does
   not belong in a pull request. That table is not a wishlist; it is the main
   defence of the project against scope creep.
2. **Read [`docs/adr/`](docs/adr/).** Several decisions here are non-obvious and
   were made for reasons you will not guess. If you disagree with one, say so
   *before* implementing — disagreement is welcome, silent divergence is not.
3. **Check [`docs/compatibility.md`](docs/compatibility.md)** to see whether the
   area you want to work on has already been designed.

---

## Where to contribute

| Repository   | Contents                | You need this to build it |
| ------------ | ----------------------- | ------------------------- |
| `unsoftone`  | docs only               | nothing                   |
| `u1-docs`    | U1 Docs                 | Rust + a C++ toolchain    |
| `u1-sheets`  | U1 Sheets               | Rust                      |
| `u1-slides`  | U1 Slides               | Rust                      |
| `u1-notes`   | U1 Notes                | Rust                      |

Architecture changes that affect more than one product belong in the
`unsoftone` repository as an ADR. Code changes belong in the product repository.

---

## Contribution types, in order of usefulness right now

1. **Real-world files.** The round-trip corpus is the most valuable thing anyone
   can contribute and it requires no coding skills. Messy, hand-edited documents
   that break things are gold. **Do not send documents containing anything you
   do not have permission to share** — redact them first.
2. **Compatibility bug reports.** "This file opened but lost my footnotes" with
   an attached sample is the single most actionable report we can receive.
3. **Layout correctness tests.** Expected rendering for a specific script,
   direction or font combination. [ADR-0002](docs/adr/0002-text-layout-parley.md)
   makes these mechanical to produce.
4. **Code.** Genuinely welcome — see the constraints below.

---

## Hard constraints

These are architectural requirements, not style preferences. A contribution that
violates one of them will not be merged, however good it otherwise is.

1. **No network calls in core code paths.** No accounts, no telemetry, no
   activation, no phone-home. [ADR-0007](docs/adr/0007-local-first-is-not-negotiable.md)
2. **Never destroy content we do not understand.** If a file feature is not
   implemented, pass it through untouched. Silently dropping content is the worst
   bug this project can have. [ADR-0005](docs/adr/0005-ooxml-lossless-layer.md)
3. **Respect the layering.** Foundations know nothing about the document model;
   the document model knows nothing about file formats or the UI.
4. **Accessibility is part of the model,** not a wrapper applied at the end.
5. **Rust core.** [ADR-0001](docs/adr/0001-rust-first-architecture.md)

---

## Changing an architecture decision

Do not edit an accepted ADR. Write a new one that supersedes it and say plainly
which evidence changed your mind. The old record stays — the trail is the point.

```bash
# in the unsoftone repository
cp docs/adr/README.md /tmp/  # read the format first
# create docs/adr/00NN-your-decision.md
```

PRs that change architecture should modify ADRs **and** the code together. An ADR
without the implementation is a proposal, not a decision.

---

## Code standards

- `cargo fmt` and `cargo clippy -- -D warnings` must be clean. No exceptions.
- Unsafe code requires a comment explaining why it is sound.
- Public items get doc comments; non-obvious algorithms get a comment explaining
  *why*, not *what*.
- Tests are expected for anything with a correctness surface. For layout, golden
  images; for file I/O, round-trip; for undo, property tests.
- Commits should be small and reviewable. Do not mix refactors with behaviour
  changes.

---

## Commit messages

Explain **why**, not what. The diff already says what.

```
Good:
  keep footnote refs when the footnote body is unrecognised

  Round-trip tests showed footnotes silently dropped when the body used a
  run property we do not model yet. Preserve the original element instead.

Bad:
  fix footnotes
  update code
  misc changes
```

---

## Reporting bugs

A useful report contains:

1. **What you did** and **what happened instead**
2. **Version**, operating system
3. **A sample file**, if you can share one — this is the part that matters most
4. **Whether the round trip lost content** — if content disappeared on
   open → save → open, say so loudly; it is treated as the highest severity

---

## Licensing

Contributions are accepted under [GPL-3.0](LICENSE), matching the rest of the
ecosystem. See [ADR-0006](docs/adr/0006-license-gpl-3.md) for why.

By contributing you confirm you have the right to license the work under GPL-3.0.
Do not contribute code you copied from a project under an incompatible license,
including proprietary code. When in doubt, ask.

---

## Not looking for

- Feature requests for things in [`PLAN.md`](PLAN.md) "Not now"
- Contributions that require an account, a network call, or telemetry
- Cloud, sync or collaboration features for the core products
- Large refactors with no behavioural goal attached
- Abstractions designed "for the future" — we extract shared code when two
  products have actually needed it
