# cargo-axi post-v0.1 roadmap

> **Fork-only planning document.** This file is normally excluded through this
> clone's `.git/info/exclude`. It may be committed to an `AG9898/axi` planning
> branch, but must never be added to a `kunchenguid/axi` pull request.

## Purpose

`cargo-axi` v0.1 is intentionally a small AXI: agents can orient in a Rust
workspace, inspect packages and features, run constrained `check` or Clippy
validation, and receive compact session context through opt-in integrations.

This roadmap records possible additions without expanding the released v0.1
contract by implication. Each item must be separately approved, designed,
tested, documented, released from `AG9898/cargo-axi`, and only then reflected
in the AXI catalog description if the public scope materially changes.

## Alignment snapshot

The released tool already covers AXI's intended agent surfaces:

- **CLI:** Cargo-subcommand installation and TOON-only workspace inspection and validation.
- **Agent discovery:** an installable Agent Skill checked against canonical command metadata.
- **Ambient context:** explicit, idempotent hooks for Claude Code, Codex, and OpenCode.
- **Distribution:** crates.io and the public `AG9898/cargo-axi` repository.

Its remaining gap is **proven OS portability**, not an intended platform claim:
the code handles Windows executable/path quoting, but CI has so far exercised
Ubuntu only. The cross-platform lane below should precede any claim of broad
platform support.

## Roadmap order

### R1 — release hardening and cross-platform proof

**Outcome:** establish support evidence for Rust/Cargo environments on Linux,
macOS, and Windows before adding more surface area.

- Add CI jobs for the supported MSRV and stable toolchains on the three OSes.
- Exercise read commands, structured error output, `check`, and Clippy where
  the component is available.
- Add focused hook-path tests for Windows quoting and project/user locations;
  do not install real user hooks in CI.
- Document the supported-platform policy and retain Cargo installation as the
  primary delivery path.

**Acceptance:** all supported OS/toolchain jobs pass and the README makes only
claims that the matrix proves.

### R2 — complete the read/validate Rust loop

**Outcome:** let an agent format-check and test a selected package or workspace
using the same strict, compact protocol as `check` and `clippy`.

1. Add `cargo axi fmt --check` with normalized status and an explicit note
   that it never rewrites source.
2. Consider `cargo axi test` with the existing package/workspace and feature
   selectors, TOON test summaries, normalized failing test locations, and a
   `--full` escape hatch for rendered output.

**Decision required before implementation:** tests execute workspace test code.
Although this does not edit source, manifests, or the lockfile, it has a
larger execution side effect than v0.1's validation commands. Define the
execution policy, timeout/cancellation behavior, and any explicit confirmation
or safe-default scope before exposing it.

**Acceptance:** no arbitrary Cargo forwarding; `fmt --check` is read-only;
test failures are agent-actionable; success, failure, empty, and timeout cases
have strict-TOON process-boundary tests.

### R3 — toolchain and workspace preflight

**Outcome:** eliminate avoidable failed turns caused by a mismatched local
environment.

Potential command: `cargo axi doctor` (or `cargo axi toolchain`) reporting
only the information an agent needs to decide its next command:

- installed `cargo`/`rustc` version versus a package's declared `rust-version`;
- presence of `Cargo.lock` and whether the requested `--locked` validation is
  viable;
- availability of `clippy` and `rustfmt`; and
- a compact package/default-member summary.

**Safety:** inspect local tool state only; no component installation, toolchain
switching, lockfile update, or network access.

**Acceptance:** structured remedies name the next `cargo axi` action or the
specific external prerequisite, and tests cover absent components and version
mismatch without relying on a developer's machine state.

### R4 — dependency and change-impact navigation

**Outcome:** help an agent determine which package or feature is affected
before it edits manifests or source.

- `cargo axi deps <package>`: compact direct dependency summary, defaulting to
  package name, source/type, and optional/feature status.
- `cargo axi why <crate>`: concise reverse-dependency path(s) explaining why a
  crate is present.
- Optional later `cargo axi impact`: map an explicitly supplied changed package
  or path to affected workspace packages. Keep Git integration read-only and
  opt-in if this is pursued.

**Safety:** derive data from Cargo metadata / a locked dependency view. Never
run update, add, remove, fix, or publish commands.

**Acceptance:** default responses remain bounded and show total counts; large
graphs use limits/truncation plus complete next-step commands; feature and
target-specific behavior is documented.

### R5 — richer, still-private session context

**Outcome:** make later sessions more useful without weakening the approved
summary-only privacy model.

Candidate additions:

- `cargo axi session inspect` to show exactly what is currently retained;
- a compact home-view status such as the last validation kind/status/duration;
- explicit retention controls, including `clear`, rather than accumulating a
  history by default.

**Guardrails:** keep the one-way workspace key; retain no prompts, transcripts,
source text, file lists, raw diagnostics, or workspace path. Any new retained
field, cross-session history, or external telemetry requires a separate
maintainer privacy decision.

**Acceptance:** state-file tests prove the negative privacy guarantees as well
as the positive summary behavior.

### R6 — ecosystem integrations and packaging

**Outcome:** reduce setup friction once core CLI behavior and platform support
are mature.

- Keep the current Claude Code, Codex, OpenCode hooks as the supported primary
  integrations; add other agent hosts only when they expose a stable,
  maintainable lifecycle interface.
- Consider a documented, opt-in CI example that emits a compact validation
  artifact. It must not silently upload source or diagnostics.
- Consider signed/prebuilt binaries only if Cargo installation speed becomes a
  demonstrated adoption problem. Cargo/crates.io remains the source of truth.

**Acceptance:** every integration is explicit, idempotent, removable, scoped,
and verified on its stated platform. It must not turn `cargo-axi` into a
general-purpose CI runner or remote service.

## Deliberately out of scope

Do not add these features without a new charter and explicit authorization:

- arbitrary Cargo argument passthrough;
- source, manifest, lockfile, dependency, Git, registry, or publishing
  mutations;
- automatic compiler-error fixes or dependency upgrades;
- prompt/transcript/source capture, telemetry, or workspace-path persistence;
- a dynamic plugin system or a replacement Cargo frontend.

## Suggested next implementation decision

After R1, choose between **R2 formatting/test support** and **R3 preflight**.
R3 preserves the v0.1 read-only safety model and is the lower-risk next slice.
R2 creates the most complete agent validation loop, but `cargo axi test`
requires the explicit execution-policy decision recorded above.
