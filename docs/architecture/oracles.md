# Oracles and anti-faking

The **oracle** is whatever decides whether generated work is correct: acceptance tests, unit
and property tests, rule checks, simulation testbenches. Every discipline pack must keep its
oracle out of the generating agent's reach (principle P6). This page gives the general rules
first, then the software pack's implementation.

## General rules (all disciplines)

1. **Separate authorship.** A zone B identity writes the oracle; a zone C identity writes the
   artifact.
2. **Locked before generation.** The oracle is hashed into a manifest signed by the oracle
   identity before the implement stage starts.
3. **Must fail first.** Every new oracle check must fail against the stubs or empty design
   (red-check). A check that cannot fail asserts nothing.
4. **Verify execution, not just integrity.** The manifest lists the *expected checks*. The gate
   confirms that every one of them ran and passed, and none was skipped.
5. **Hidden subset.** At T2+, part of the oracle (holdouts) is kept in a private repository and
   only added at gate time. Feedback on holdouts is coarse (requirement ID only).
6. **Lock what affects the oracle.** Configuration and shared fixtures that change what the
   oracle checks are owned by the oracle identity and included in the manifest.
7. **Disagree by dispute.** The generating agent never edits the oracle. It raises a dispute.

## Repository layout per tier

| Tier | Layout |
| --- | --- |
| T0–T1 | Single repo with protected zones: spec, locked tests and source in one repository, with paths owned per identity |
| T2 | `<app>-spec`, `<app>`, `<app>-acceptance`. Acceptance tests drive the built artifact as a black box |
| T3 | As T2, with the acceptance repository owned by an independent verification owner |

Holdouts always live outside the implementation repository, in a **private** holdouts
repository per deployment. They are never published in the public Brightsmiths organisation.

## Software pack

### Locked tests must be addressable by path

| Language | Locking convention |
| --- | --- |
| Rust | Tests in sibling files: `#[cfg(test)] #[path = "foo_tests.rs"] mod tests;`. Public-API-only tests in `tests/`. Brightsmiths repositories already follow this convention. |
| C# | `tests/App.Tests.Locked/` as its own project |
| TypeScript | `*.locked.test.ts` for oracle tests; the implementer's own tests keep `*.test.ts` |

The Rust `mod tests;` declaration sits in a file the implementer owns. Deleting it silently
drops the tests from compilation; the inventory check catches this.

### Manifest

```yaml
# tests/locked/MANIFEST (signed by the oracle identity → MANIFEST.sig)
files:
  - path: src/billing/invoice_tests.rs
    sha256: 9f2c...
test_config:
  - path: .cargo/config.toml
    sha256: 77d0...
expected_tests:
  - billing::invoice::tests::rejects_negative_amounts
policy:
  all_expected_must_run: true
  skipped_allowed: false
  extra_locked_tests_allowed: false
```

### What must be locked or banned

| Group | Examples | Control |
| --- | --- | --- |
| Test discovery and configuration | `Cargo.toml` features, `build.rs`, `.cargo/config.toml`; `.runsettings`, test `.csproj`, `Directory.Build.*`; `vitest`/`jest` config, `tsconfig` include/exclude | Owned by the oracle identity or a human; hashed by `oracle-guard config-guard` |
| Test-only behaviour in production code | `#[cfg(test)]` outside the module declaration, test-only features; `#if DEBUG`, `InternalsVisibleTo` changes; `NODE_ENV === 'test'`, `import.meta.vitest` | Banned by `oracle-guard sniff` |
| Shared test infrastructure | Fakes, builders, fixtures, snapshots | Owned by the oracle identity; in the manifest |

Where a file mixes production and test settings, split it, or hash only the test-affecting
section.

### Sandbox enforcement for colocated files

When locked tests share directories with source files, agent-gate's mount plan sets them up as
follows:

- Locked files are owned by a different user from the agent, with mode `0444`.
- Directories use mode `1775` with the sticky bit set.

The agent can create files but cannot modify, delete or rename the locked ones. This is defence
in depth; the gate-warden checks decide.

### Hidden unit tests

Hidden tests are stored in the holdouts repository and run in-tree. `oracle-guard
holdout-inject` copies them into the clean gate workspace before the test run:

| Language | How hidden tests are added |
| --- | --- |
| Rust | Copy files into `tests/`, or replace the content of a locked placeholder file behind `#[cfg(test)] mod __holdout;` |
| C# | Add an `App.Tests.Holdout` project at gate time |
| TypeScript | Copy in `*.holdout.test.ts` files |

The hidden test IDs are added to the inventory at gate time.

### Seams

- **Locked tests target only seams approved at H3:** ports, use cases and public module APIs,
  never private helpers.
- **Changing a seam is a design change** (a test change request).
- **Below the seams the implementer is free.** Its own tests in `tests/own/` never decide
  pass/fail, but they do count toward mutation score.

### Toolchain mapping

| Concern | Rust (first) | C# | TypeScript |
| --- | --- | --- | --- |
| Mutation | cargo-mutants | Stryker.NET | StrykerJS |
| Property tests | proptest | FsCheck / CsCheck | fast-check |
| Fuzzing | cargo-fuzz | SharpFuzz | Jazzer.js |
| Bounded proofs | Kani; Miri for UB | — | — |
| Machine-readable results | JUnit XML via cargo-nextest | TRX | JUnit XML |
| Architecture rules | Workspace crate boundaries, cargo-deny | NetArchTest | dependency-cruiser |
| Cross-cutting | Semgrep, CodeQL, gitleaks, Syft (SBOM), OSV/Grype | | |

## Anti-faking controls

| Faking vector | Control |
| --- | --- |
| Edit tests to match code | Separate identity, signed manifest, read-only mounts, scope check |
| Code only to the visible tests | Holdouts, property tests with fresh seeds, mutation testing |
| Special-case the test environment | Sniff rules; holdouts run against the built artifact elsewhere |
| Make locked tests silently not run | Inventory: every expected test must run, pass and not be skipped; config guard |
| Fix a fixture or snapshot instead of the code | Fixtures and snapshots are in the manifest |
| Weak tests from the implementer | `tests/own/` never decides pass/fail |
| Tests that assert nothing | red-check |
| Forge a green status | Required checks accepted only from the gate-warden App |
| Learn the holdouts through feedback | Coarse feedback, bounded attempts, holdout rotation, visible-vs-holdout gap metric |
| Persuade a reviewer | Reviewers see artifacts and evidence, never builder reasoning |
| Change the rules | Harness, gate definitions, baselines and `.factory/` are not writable by any agent |

**What this does not solve:** if the program design is wrong, the oracle faithfully encodes the
mistake. Human approval of criteria and program design, and black-box acceptance tests, are the
only defences.
