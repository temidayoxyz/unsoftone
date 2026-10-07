# ADR-0001: Rust-first architecture

- Status: Accepted
- Date: 2026-10-07

## Context

UnsoftOne must ship four native desktop applications for Windows, macOS and
Linux. They need to be fast, low in memory, fully functional offline, and
maintainable by a small number of people over many years.

The requirements that actually bind:

- **Offline-first** — no core code path may require the network.
- **Native performance** — a large document must stay responsive while editing.
- **Cross-platform** — one codebase, three desktop platforms.
- **Low resource use** — a word processor should not feel like a browser.
- **Long-lived** — file-format compatibility must be maintainable for decades.

We also have a standing instruction that must *not* be violated: Rust does not
imply Tauri. The UI framework choice has to be argued on merit.

## Decision

**Build the core in Rust.** Document models, file-format I/O, text layout, undo,
clipboard, PDF generation and the rendering core are all Rust. Platform
integration is Rust via the ecosystem's windowing and graphics crates.

**Rust is the core. It is not, by itself, a UI decision.** The UI stack is
decided separately in [ADR-0003](0003-desktop-ui-stack.md) and must be validated
by a spike before it is committed to.

Where a product genuinely needs a non-Rust layer (e.g. a webview-hosted UI), that
is a deliberate exception, recorded as an ADR, not an accident.

## Consequences

**Easier:** one language across the whole stack; excellent memory safety for
parsing untrusted, malformed files (which is the normal case in file-format
work); first-class concurrency; a large mature ecosystem for exactly the
hard parts we need (`zip`, `quick-xml`, `parley`, `pdf-writer`).

**Harder:** a smaller talent pool than TypeScript/C++; a young ecosystem for
desktop UI, which is why the UI decision needs a spike; longer compile times;
ffi friction where we must reach platform APIs.

**Forecloses:** reusing the enormous body of existing JavaScript office tooling.
Accepted — most of it is web-only, and the offline-native requirement is not
negotiable.

## Alternatives considered

**Electron / full Node.js desktop.** Rejected: heavy memory footprint directly
contradicts the low-resource goal; large attack surface; the offline-native
story would be weaker.

**C++ / Qt (LibreOffice's approach).** Rejected: this is exactly the path that
produces a 10M-line codebase. Memory safety and modern tooling are decisive for
a small team.

**Go.** Rejected: a good fit for tooling and services, but the desktop GUI and
text-shaping ecosystem in Rust is substantially more mature.

**Rust + JavaScript everywhere (Electron-style).** Rejected: see Electron.
