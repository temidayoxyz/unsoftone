# ADR-0002: `parley` over `cosmic-text` for text layout

- Status: Accepted
- Date: 2026-10-07

## Context

A word processor is, before it is anything else, a **text layout engine**. The
dominant cost — and the dominant source of bugs — is:

- font enumeration and font fallback across three platforms
- shaping (the HarfBuzz algorithm) for Arabic, Indic and other complex scripts
- line breaking, justification and hyphenation
- bidirectional text
- inline baseline alignment
- incremental re-layout so typing stays at 60fps in a 500-page document

No off-the-shelf crate does **page** layout for a document. But inline text
layout — the shaping and line-breaking foundation — must come from somewhere,
and there are exactly two serious pure-Rust candidates.

## Decision

**Use [`parley`](https://github.com/linebender/parley) (Linebender) for inline
text layout.**

We will build the block/flow/page layer ourselves. That layer is ours; the
inline layer is not.

### Why parley

| | `parley` | `cosmic-text` |
| --- | --- | --- |
| Latest release | v0.11.1 (Aug 2026) | v0.19.0 (Apr 2026) |
| Backing | Linebender — the Servo/Stylo organisation | System76 / COSMIC |
| Correctness | validated against the **Web Platform Tests** | own test suite |
| Stack | fontique, harfrust, skrifa, icu4x 2.0 | fontdb, harfrust, swash |
| Roadmap | active; floats and advanced layout in progress | visibly stalled |

The deciding factors:

1. **WPT validation.** parley is tested against the browser text-layout test
   suite. For a word processor that must render Arabic, Indic and CJK
   correctly, that is the strongest correctness signal available in Rust.
2. **Governance.** Linebender is a multi-project organisation with a long
   runway. cosmic-text's roadmap is stalled and its maintainers are visibly
   drifting toward parley.
3. **Optionality.** parley's dependency set — `fontique`, `harfrust`, `skrifa`,
   Vello, WGPU — is the same one Servo's rendering stack uses. If we later want
   a GPU rasteriser, that door stays open. That door is closed with
   cosmic-text.
4. **ICU4X 2.0** gives better Unicode and locale handling than the alternative.

## Consequences

**Easier:** correctness on complex scripts comes with a maintained library rather
than our own bug reports; Unicode handling is genuinely solid; a credible path to
GPU rendering later.

**Harder:** we must build the entire block/flow/page layer ourselves —
paragraphs, list numbering, tables, page breaking, floats, widows/orphans,
keep-with-next. parley stops at the inline level. This is the real cost, and it
is the single largest engineering item in the project.

**Forecloses:** nothing important. The inline API surface is comparable, so
swapping in `cosmic-text` remains possible if parley stalls.

## Alternatives considered

**`cosmic-text`.** Rejected. Still viable and worth knowing, but its roadmap is
incomplete and its ecosystem is consolidating onto parley. Betting against the
consolidation is a bet we do not need to take.

**Hand-rolled shaping on `rustybuzz`/`harfrust`.** Rejected: line breaking with
correct UAX #14 semantics is a deep, well-specified problem with a deep, ugly
test suite. We would be reimplementing a solved problem badly.

**CSS layout (`stylo`).** Rejected for now. Attractive because it is a
production CSS engine, but a word processor is not a web page — the page model,
box model and formatting context we actually need are a poor fit, and pulling in
a full CSS cascade is a large commitment for a poor abstraction.

**Delegate layout to the platform** (WebKit/Chromium text rendering). See
[ADR-0003](0003-desktop-ui-stack.md) — this is the main argument for the
webview, and the main risk in the spike.
