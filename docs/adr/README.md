# Architecture Decision Records

We use ADRs to record **why** the project is shaped the way it is. Code shows
what we do; ADRs record what we rejected and what it would have cost.

## The rules

1. **ADRs are immutable.** Never edit an accepted ADR to change its mind.
2. **To change your mind, write a new ADR** that explicitly supersedes an old
   one. The old record stays. The reasoning trail is the point.
3. **A significant architectural change arrives as an ADR first**, not as a
   commit. Implement only after the record exists.
4. **Disagreement is recorded too.** If an ADR was close, say so. Future
   maintainers need to know which decisions were fragile.

## Format

```markdown
# ADR-NNNN: Title

- Status: Proposed | Accepted | Superseded by ADR-NNNN
- Date: YYYY-MM-DD

## Context
What forced the decision. Constraints, not opinions.

## Decision
What we are doing, stated plainly and specifically.

## Consequences
What this makes easy. What it makes hard. What it forecloses.

## Alternatives considered
Each with the actual reason it lost.
```

## Index

| ADR                                             | Title                                | Status  |
| ----------------------------------------------- | ------------------------------------ | ------- |
| [0001](0001-rust-first-architecture.md)         | Rust-first architecture              | Accepted |
| [0002](0002-text-layout-parley.md)              | `parley` over `cosmic-text`          | Accepted |
| [0003](0003-desktop-ui-stack.md)                | Desktop UI stack (pending spike)     | Accepted |
| [0004](0004-repository-model.md)                | Repository model                     | Accepted |
| [0005](0005-ooxml-lossless-layer.md)            | Lossless OOXML layer, self-built     | Accepted |
| [0006](0006-license-gpl-3.md)                   | GPL-3.0                              | Accepted |
| [0007](0007-local-first-is-not-negotiable.md)   | Local-first is not negotiable        | Accepted |
| [0008](0008-product-sequencing-u1-docs-first.md)| U1 Docs first                        | Accepted |
