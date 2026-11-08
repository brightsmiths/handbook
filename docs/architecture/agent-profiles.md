# Agent profiles and discipline packs

CogWorks has no built-in agent modes. Every stage that needs an LLM runs an **agent profile**,
which CogWorks selects from the harness according to the needs of the work.

## What a profile is

A profile is a versioned package in the harness:

| Part | Content |
| --- | --- |
| Persona | The role prompt |
| Skills | Guidance documents loaded for this profile, such as conventions or test patterns |
| Tool allowlist | The tools and MCP servers available inside the sandbox |
| Stage contract | Inputs (the context-bundle allowlist), output schema, writable paths, network profile, budget, retry limit |
| Identity role | The GitHub App role its commits are pushed under |
| Trust zone | Which zone it belongs to (see below), which decides what it may see |

The agent never sees its own stage contract. CogWorks, agent-gate and sandbox-runner enforce
it.

```yaml
# harness: disciplines/software/profiles/implementer/stage-contract.yaml (illustrative)
stage_type: implement
zone: C
identity_role: builder
inputs:
  - spec://contracts/**
  - spec://plan/{intent}/slices/{slice}.yaml
  - impl://src/**
  - impl://tests/locked/** : read-only
  - gate://results/{run}/visible-summary.json
outputs:
  writable_paths: ["src/**", "tests/own/**"]
  schema: outputs/implement.schema.json
sandbox: { network: [model-gateway, dependency-proxy], wall_clock_min: 45 }
budget: { max_turns: 80, max_tokens: 2000000 }
retries: { max_attempts: 3 }
gates: [G6]
```

## Selection

For each stage in the graph, CogWorks resolves a profile from:

```text
(stage type, discipline, language/toolchain, tier)  →  profile@version
```

- **Stage type:** the kind of work, for example `specify`, `architect`, `contract`, `plan`,
  `acceptance-oracle`, `unit-oracle`, `implement`, `refactor`, `review-semantic`,
  `review-security`, `drift-scan` or `debt-fix`.
- **Discipline:** chosen by the repository's `.factory/pin.yaml`, or per path.
- **Language/toolchain:** detected deterministically, or declared in the pin.
- **Tier:** can select a stricter profile variant, for example extended fuzzing at T3.

The selection is deterministic and recorded in the stage attestation. No LLM chooses a profile.

## Trust zones

Profiles are grouped by the information barrier they need, not by job title.

| Zone | Stage types (software pack) | Reads | Writes | Never sees |
| --- | --- | --- | --- | --- |
| **A. Intent and design** | specify, architect, contract, plan | Confirmed intent, spec, architecture, lore-keeper | Spec paths only | Implementation tests, holdouts |
| **B. Oracle** | acceptance-oracle, unit-oracle | Spec, contracts, plan slice | Acceptance repo / locked test paths / holdouts | Implementation source |
| **C. Build** | implement, refactor, debt-fix | Contracts, slice, locked tests (read-only), coarse gate results | `src/`, `tests/own/` | Holdouts, acceptance source (T2+), harness |
| **D. Assurance** | review-semantic, review-security | Diff, spec, evidence | Nothing (structured findings only) | Builder transcripts and PR narrative, holdouts, secrets |
| **E. Health** | drift-scan | Read-only snapshot, metrics | Issue drafts only | Secrets |
| **Q. Quarantine** | intake summarisation (inside Entry Sentinel) | Raw untrusted text | Fixed-schema JSON | Everything else; no tools |

Profiles in the same zone may share context. Hand-offs across zones carry structured artifacts
(contracts, plan, criteria, findings), never prose memory.

## Visibility and write matrix (software pack)

agent-gate holds this policy. sandbox-runner mounts follow it, and gate-warden's scope check
enforces it per commit.

| Identity role | spec | contracts | `src/` | locked tests | `tests/own/` | acceptance (visible) | holdouts | harness | `.factory/` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| designer (zone A) | **W** | **W** | R (snapshot) | — | — | — | — | — | — |
| oracle (zone B) | R | R | — | **W** | — | **W** | **W** | — | — |
| builder (zone C) | R (slice) | R | **W** | R | **W** | results only (T2+) | — | — | — |
| reviewer (zone D) | R | R | R | R | R | R | — | — | — |
| health (zone E) | R | R | R | R | R | R | — | — | — |
| gate-warden | R | R | R | R | R | R | **R** | R | **W** (baselines) |
| humans | by role | by role | by role | by role | by role | by role | T3 verification owner | maintainers **W** | tech lead **W** |

"Locked tests" covers every locked path: test projects, colocated locked test files, fixtures,
snapshots and test-affecting configuration. See [oracles.md](oracles.md).

**Model diversity** (a hypothesis, not yet proven): using a different model family for zone B
than for zone C may reduce correlated misreadings of the spec. Measure the visible-vs-holdout
gap before relying on it.

## Discipline packs

Brightsmiths' core (CogWorks, gate-warden, sandbox-runner, agent-gate, chain-chronicler) knows
nothing about any one engineering discipline. A **discipline pack** supplies everything
discipline-specific:

| A pack provides | Software pack example | Hypothetical electrical pack |
| --- | --- | --- |
| Profiles per stage type | implementer, unit-oracle… | schematic designer, rule-check author… |
| Skills | `rust-conventions`, `property-test-patterns` | `kicad-conventions`, `derating-rules` |
| Stage graphs per tier | `disciplines/software/graphs/t2.yaml` | its own graphs |
| Domain service (toolchain access via CogWorks' Extension API) | Rust domain service | KiCad domain service |
| Oracle conventions (what "locked" means, how it is addressed) | locked test paths, manifest | locked ERC/DRC rule sets, simulation testbenches |
| Gate checks (commands emitting the standard result format) | oracle-guard, ratchet, mutation, clippy, Semgrep | ERC, DRC, SPICE testbench runner |
| Path conventions for agent-gate | `src/`, `tests/locked/` | `hw/schematics/`, `hw/checks/locked/` |

The architectural invariants hold across every pack:

- The oracle is written by a different identity from the design artifact.
- The oracle is locked before generation starts.
- gate-warden recomputes the verdict.

How each pack satisfies those invariants is up to the pack.

Packs live in the harness under `disciplines/<name>/`, or in separate repositories that the
harness references by version. Brightsmiths maintains only `disciplines/software/`. Its first
toolchain is Rust; C# and TypeScript mappings are documented but not yet built.
