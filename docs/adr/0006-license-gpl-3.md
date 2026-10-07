# ADR-0006: GPL-3.0

- Status: Accepted
- Date: 2026-10-07

## Context

UnsoftOne is open source and intends to remain so. The license is the primary
legal mechanism that enforces that intent, and it is a decision that is
expensive to change later.

The relevant tensions:

- **Adoption.** UnsoftOne competes with Microsoft Office. A restrictive license
  reduces the number of people and organisations who will build on it, package
  it, or contribute to it.
- **Permanence.** A permissively licensed project can be forked into a closed
  source product and the user community gains nothing. We would have built the
  ecosystem for a competitor.
- **Cloud capture.** The business model we are opposing is software delivered
  over a network and controlled by a vendor.

## Decision

**GPL-3.0**, for all repositories in the UnsoftOne ecosystem.

## Consequences

**Easier:** any distributed derivative must also be GPL and source-available.
The core ecosystem cannot be quietly closed. Distributors — Fedora, Debian, Arch,
Homebrew, Flathub, winget, the major Linux distributions' own builds — can ship
it without renegotiating. Contributors retain copyright. The license is
unambiguously accepted by the FSF and understood by every major distribution.

**Harder:** some commercial organisations will not contribute. Companies
considering an internal fork must open-source it. If UnsoftOne later gains a
commercial or hosted offering, GPL-3.0 imposes real obligations on how that
offering interacts with the GPL codebase.

**Forecloses:** relicensing to a permissive license without relicensing every
contribution. Note: MIT/Apache relicensing is effectively irreversible in
practice once outside contributors have participated, because their patches
cannot be retro-permitted.

## Consequences for the plugin question

GPL-3.0 has specific implications for a future plugin/extensibility system
([PLAN.md](../PLAN.md) "Not now"). A plugin mechanism designed so that plugins
can be proprietary while linking against GPL libraries is legally delicate and
depends on how linking occurs. **This is a further reason not to design a plugin
ABI before the core API has stabilised** — and a good reason to involve a
lawyer when we do.

## Alternatives considered

**MIT / Apache-2.0.** Rejected. Maximum adoption and zero friction, but anyone
can fork it closed and large companies can build proprietary versions with no
obligation to contribute. For a project whose entire premise is being an open
alternative to a proprietary incumbent, this defeats the purpose.

**MPL-2.0.** A serious contender, and the closest runner-up. File-level copyleft
keeps improvements open while allowing proprietary combination. Rejected because
GPL-3.0's guarantee is stronger and the adoption difference turned out not to be
decisive — major distributions ship GPL happily.

**AGPL-3.0.** Rejected. Its network clause targets hosted services, which we do
not operate and do not intend to. GPL-3.0 already covers distribution, which is
how a desktop application reaches users.

**LGPL-3.0.** Rejected. Designed for libraries that must be linkable into
proprietary applications. We are building applications, and the weaker guarantee
does not match the intent.

## Note

This ADR is a decision of intent, not legal advice. Before the plugin system is
designed, and before any commercial model is considered, get a qualified lawyer
involved. Record the outcome in a new ADR.
