# ADR-0004: Repository model — docs-only meta-repository plus four independent product repositories

- Status: Accepted
- Date: 2026-10-07

## Context

UnsoftOne is one brand with four products. They must:

- be independently buildable, testable, versioned and released
- share a direction, a design language and a plan
- never become one inseparable application
- not require shared code that does not yet exist

There is a real tension here. Fixing repository boundaries **before** discovering
what is genuinely shared locks in guesses. Committing to four repos on day one
maximises the coordination tax on code we do not yet have, while risking that
the shared direction lives in a file with no URL and no history.

The failure mode we most want to avoid is four disconnected applications that
become impossible to maintain together. The second most serious failure mode is
four products designed in parallel, none of them deep.

## Decision

**One docs-only meta-repository, plus four fully independent product
repositories.**

```
UnsoftOne/          →  unsoftone   (docs only; product folders are gitignored)
├── U1Docs/         →  u1-docs     (independent repo)
├── U1Sheets/       →  u1-sheets   (independent repo)
├── U1Slides/       →  u1-slides   (independent repo)
└── U1Notes/        →  u1-notes    (independent repo)
```

### Rules

1. **This repository contains no product source code, ever.** The four product
   folders are listed in `.gitignore`. This is what prevents the meta-repo from
   trying to manage nested repositories.
2. **Each product repository is autonomous.** Nothing in `unsoftone` gates a
   product build, release or CI run.
3. **Shared code is not shared by default.** When a shared need appears, it goes
   into its own repository (`u1-core`) consumed as a git dependency — never by
   copying files between repos.
4. **Shared *design* is immediate; shared *code* is earned.** Design tokens,
   iconography, keyboard model and interaction patterns are specified here from
   day one. Code sharing waits until the same hard problem has actually been
   solved twice.

### Why not a Cargo workspace monorepo

Not because it is wrong — it is the right answer *later* — but because **today
three of the four products are empty directories.** A workspace with three empty
members buys coordination we do not need and costs nothing we do not currently
need to avoid. When product #2 starts and shared code genuinely exists, the
question becomes real and we will answer it with evidence rather than a guess.

Splitting later is cheap (`git filter-repo`, subtree split). Merging is
expensive. We have the asymmetry in the right order.

## Consequences

**Easier:** the plan, ADRs and compatibility matrix have a URL and a real
history, so contributors and future maintainers can read the reasoning;
products cannot be coupled by accident; a product can be abandoned without
killing the ecosystem; releases are genuinely independent.

**Harder:** shared infrastructure requires cross-repository version management
when it arrives; identical bug fixes are duplicated until the shared crate
exists; cross-cutting changes are not atomic.

**Forecloses:** a single `cargo build` for the whole ecosystem. Accepted, with
the mitigation that the ecosystem has one product in active development at a
time.

## When to revisit

Write a superseding ADR when **all** of these are true:

- a second product is under active development, **and**
- at least one shared Rust crate exists with a demonstrated, product-agnostic
  API, **and**
- the cross-repository version cost has actually been felt.

Not before.

## Alternatives considered

**Cargo workspace monorepo.** Rejected *for now*. Deferred with a specific,
written revisit trigger rather than rejected on principle — this is very likely
the right answer once shared code exists.

**Four repos on day one with no shared-documentation repo.** Rejected: this was
the previous plan. It leaves the shared plan and ADRs living in an unversioned,
unreachable local file, which is precisely how the "why" evaporates over a
multi-year project.

**A single repository where all four products live as directories.** Rejected:
invites the four-products-shallowly failure mode and makes per-product releases
painful.

**One monolithic application with four modes.** Rejected outright: directly
contradicts the requirement that products be independent applications.
