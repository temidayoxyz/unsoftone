# UnsoftOne

**Download. Install. Work.**

UnsoftOne is a free and open-source desktop productivity suite — a genuine
alternative to the Microsoft Office ecosystem. No account. No subscription. No
sign-in. No internet connection required for anything that matters.

```
UnsoftOne
├── U1 Docs    — word processing      → u1-docs
├── U1 Sheets  — spreadsheets          → u1-sheets
├── U1 Slides  — presentations         → u1-slides
└── U1 Notes   — notes                 → u1-notes
```

**U1** is the product family. **UnsoftOne** is the ecosystem.

Each product is its own application, its own Git repository, its own release
cycle. They share a direction, a design language, and — over time — proven
shared Rust infrastructure. They are not one inseparable program.

---

## Product philosophy

The core experience must never require:

- an UnsoftOne account
- an internet connection
- a subscription or license payment
- cloud storage
- online activation
- a Microsoft account

You download an installer. You install it. You open your file. You save to your
own disk. That is the whole product.

Cloud, sync, collaboration and accounts may be added **later, as strictly
optional** features. They may never become a requirement for the core
applications. This is a hard architectural constraint, not a policy preference
— see [ADR-0007](docs/adr/0007-local-first-is-not-negotiable.md).

---

## Repository model

This repository contains **documentation only**. It is the home of the plan, the
architecture, and the decision records that keep four products moving in one
direction.

| Repository     | Purpose                     | Contains product code? |
| -------------- | --------------------------- | ---------------------- |
| `unsoftone`    | Ecosystem direction         | No                     |
| `u1-docs`      | U1 Docs                     | Yes                    |
| `u1-sheets`    | U1 Sheets                   | Yes                    |
| `u1-slides`    | U1 Slides                   | Yes                    |
| `u1-notes`     | U1 Notes                    | Yes                    |

Each product repository is independently buildable, testable, versioned and
releasable. Nothing in this repository gates a product build.

See [ADR-0004](docs/adr/0004-repository-model.md) for the reasoning, including
why the product folders are gitignored here rather than tracked.

---

## Where things live

```
UnsoftOne/
├── README.md            ← you are here
├── PLAN.md              ← roadmap, priorities, and explicit "not now"
├── CONTRIBUTING.md
├── docs/
│   ├── adr/             ← numbered, immutable decision records
│   ├── architecture/    ← cross-product architecture
│   └── compatibility.md ← file-format fidelity matrix
├── U1Docs/              ← independent repo
├── U1Sheets/            ← independent repo
├── U1Slides/            ← independent repo
└── U1Notes/             ← independent repo
```

---

## Status

**Pre-alpha. Nothing here is usable yet.**

This project is at the very beginning. The honest summary:

- A large, real problem — a local-first, open alternative to Microsoft Office
- A small, concrete next step — see [PLAN.md](PLAN.md)
- No working application, no builds, no users

We are not going to pretend otherwise. Build a genuinely excellent U1 Docs, and
the rest follows from that.

---

## Contributing

Read [PLAN.md](PLAN.md) and `docs/adr/` first. Architectural decisions live in
ADRs; significant changes should arrive as a new ADR that supersedes an old one
rather than as an edit to the code.

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[GPL-3.0](LICENSE) — see [ADR-0006](docs/adr/0006-license-gpl-3.md).
