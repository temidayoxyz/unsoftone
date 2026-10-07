# ADR-0007: Local-first is not negotiable

- Status: Accepted
- Date: 2026-10-07

## Context

Almost every modern productivity application requires an account and a network
connection before you can type a word. That is not a neutral default — it is a
business model. It means:

- the product can be discontinued or acquired
- files can be held hostage, or rendered inaccessible when a company folds
- work is subject to someone else's uptime, bandwidth prices and policy changes
- the software cannot be used on an airliner, in a hospital air-gapped network,
  or during an outage
- data is collected by default, because it is valuable to the vendor

Microsoft Office is the clearest example of the failure this ADR exists to
prevent: subscribing to it means the software stops working if the subscription
stops.

## Decision

**The core UnsoftOne applications must function completely offline, with no
account, no sign-in, no subscription, no activation and no network access.**

Specifically:

1. **No core code path may require the network.** Not "no network by default" —
   no network at all.
2. **No account may be required** to create, open, edit, save or export a
   document.
3. **Users own their files.** Saving writes to the local filesystem, in standard
   formats, readable by other software. Our formats are not a lock-in mechanism;
   any proprietary extension must be additive and optional.
4. **No telemetry.** None, by default, anywhere in the core products.
5. **No online activation or licence check.** A licence is a file, or there is no
   licence to check.
6. **Cloud, sync, collaboration and accounts may only ever be optional
   add-ons** that are useful when present and never required when absent.

This is an architectural constraint on the codebase, not a policy preference or a
business decision. Code that violates it does not merge.

## Consequences

**Easier:** trust. Users can verify the absence of network calls. Adoption in
regulated, air-gapped and offline environments. No compliance surface for data
protection law. No accounts to secure, breach or delete. No server operating
cost, which is what makes "free and open source" sustainable rather than a
promise funded by ads.

**Harder:** no cloud collaboration features in the core; no usage analytics to
guide prioritisation, so user feedback must come from voluntary channels;
syncing, if ever built, is a separate product surface that must not contaminate
the core; some conveniences users expect from connected software will be absent,
permanently.

**Forecloses:** any business model based on subscription revenue from the core
products. Accepted and intended.

## Enforcement

- Any dependency that phones home is a bug, not a feature request.
- Code review checks for network-capable paths in core.
- The Phase 0 spike and Phase 5 packaging both include a check that the built
  application runs correctly with the network disabled.

## Alternatives considered

**Optional accounts from launch, to learn about users.** Rejected. It is the
exact pattern we are opposing, and it is remarkably difficult to remove later —
once the code depends on an identity, the "optional" path decays into the
default.

**Telemetry, opt-out.** Rejected. Privacy-respecting defaults exist precisely
because opt-out does not work.

**License-key phone-home activation.** Rejected: converts a free local
application into a network dependency, which is the failure mode above.
