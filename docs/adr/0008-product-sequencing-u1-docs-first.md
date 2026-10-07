# ADR-0008: U1 Docs first, and deeply

- Status: Accepted
- Date: 2026-10-07

## Context

UnsoftOne has four products. There is a strong pull to establish the shared
architecture first and then build all four on top of it — the ecosystem framing
makes this feel obvious.

The failure mode of that plan is **four shallow applications**. Each product
gets 20% of the attention, none reaches usefulness, and the shared abstractions
are guessed rather than earned. Guessed abstractions are wrong; when they are
also load-bearing, correcting them later is expensive and disruptive.

There is a second, subtler failure: designing the shared layer before hitting
the real problems means designing for the problems we *imagine*. The genuinely
hard shared surface — undo, clipboard, fonts, accessibility, atomic save — only
becomes obvious once one product has been built far enough to feel them.

## Decision

**Build U1 Docs first, to genuine quality. Begin the other three only after.**

Sequence:

1. Spike and build U1 Docs until it genuinely opens, edits, saves and exports the
   DOCX files people actually have.
2. During that build, the shared surface is discovered rather than designed.
3. Extract shared code into a `u1-core` repository **only** for needs proven by
   real code, per [ADR-0004](0004-repository-model.md).
4. Start U1 Sheets, U1 Slides and U1 Notes against that proven foundation.

When each additional product starts is a judgement call made at the time, not a
schedule fixed now. The rule is that only **one product is under active
development at a time.**

## Consequences

**Easier:** U1 Docs gets the full depth it needs to be credible. The hard
problems — IME, layout, undo, accessibility, lossless OOXML — are solved once,
properly, and become the foundation for everything after. Shared abstractions are
extracted from working code rather than speculation, so they are the right ones.
Adoption follows credibility: a real U1 Docs is worth more to the project than
four half-working applications.

**Harder:** the ecosystem looks lopsided for a long time. Sheets, Slides and
Notes are visibly waiting. Temptation to start something small in parallel — a
Markdown editor, a quick "Notes" — is high, and that temptation is exactly what
produces four shallow products. Stated here so that when it happens, it is a
recognised failure rather than a reasonable-sounding plan.

**Forecloses:** demonstrating four products at once. Accepted.

## What "deeply" means

Not "many features." It means:

- it opens the DOCX files people actually have, and does not damage them
  ([ADR-0005](0005-ooxml-lossless-layer.md))
- typing is smooth in a large document
- text renders correctly in every script, including bidirectional and CJK
- it saves files other software can open
- it exports usable PDF
- it works with the network switched off
- it is accessible enough to be usable with a screen reader

That list is the definition of done for U1 Docs v1. Everything else is
afterwards.

## Alternatives considered

**Establish shared architecture first, then build all four.** Rejected: this
guesses at the shared surface and produces four shallow products. The shared
surface is discovered, not designed.

**Build the easiest product first (U1 Notes) to learn the stack.** Tempting, and
still rejected. Notes teaches us about rich text but almost nothing about page
layout, pagination, tables or OOXML — which is where a word processor actually
is difficult. We would learn the easy parts and postpone the hard ones, then run
out of runway at exactly the wrong moment.

**Build four products in parallel at reduced scope.** Rejected: four shallow
products.

**Build all four as one application with four modes.** Rejected: contradicts the
independence requirement in [ADR-0004](0004-repository-model.md).
