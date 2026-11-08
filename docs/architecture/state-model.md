# State model

GitHub is the only system of record (ADR-0001). This page lists which GitHub objects carry which
state, so that any service can rebuild its view after a restart.

## GitHub objects and what they mean

| Object | Carries | Written by |
| --- | --- | --- |
| Issue | The work item, its human-readable history, sub-issue hierarchy (plan slices) | Humans; CogWorks (sub-issues); drift-scout profile (health issues) |
| Labels | Lifecycle state, tier, priority, type, holds | Services per namespace (below); some are human-only |
| Hidden signed comments | Attestations: Entry Sentinel approval, triage block, Scope-Sage assessment, CogWorks stage records | The owning service's App |
| Pull requests | Spec PRs (design artifacts), impl PRs (per slice), release PRs | Profiles through sandbox-runner; release-regent |
| PR reviews | Human gate decisions (H1–H3, H5, H6) | Humans only; gate-warden verifies who reviewed |
| Check runs | Gate verdicts G1–G9 | gate-warden only (required-check source) |
| Commits | Artifacts; authorship identifies the role | Role Apps through sandbox-runner |
| Releases | Published outcome, SBOM, provenance | release-regent; release-ratifier attaches evidence |
| Repository files | `risk-map.yaml`, `.factory/pin.yaml`, baselines, spec, manifests | Humans; gate-warden (baselines); profiles within their owned paths |

## Label namespaces

Proposed taxonomy; harmonising it with the labels services already use is a roadmap item.

| Namespace | Examples | Owner | Human-only? |
| --- | --- | --- | --- |
| `process:` | `process:development`, `process:admin` | Entry Sentinel | — |
| `state:` | `state:discovery`, `state:design`, `state:implementation`, `state:quarantine` | Entry Sentinel, humans, CogWorks | `state:design` and the factory trigger are human-only |
| `tier:` | `tier:t0` … `tier:t3` | CogWorks proposes, humans confirm (T2+) | Lowering is human-only |
| `prio:`, `type:` | `prio:high`, `type:bug` | Triage Titan | — |
| `factory:` | `factory:escalated`, `factory:waiting-human`, `factory:security-hold` | CogWorks | Clearing a hold is human-only |
| `health:` | `health:mechanical`, `health:structural` | Drift-scout profile proposes, H7 triage confirms | Confirmation is human-only |

gate-warden verifies that every human-only label event was made by an authorised human.

## Branches

| Pattern | Use |
| --- | --- |
| `cogworks/<issue>/<stage>` | Spec artifacts per stage (e.g. `cogworks/42/spec`, `cogworks/42/contracts`) |
| `cogworks/<issue>/slice-<n>` | Impl work per slice |
| release-regent's branch | Rolling release PR |

## Per-repository factory files

```text
risk-map.yaml             # path globs → tier (human-owned)
.factory/pin.yaml         # harness version, discipline pack, overrides (human-owned)
.factory/baseline/        # ratchet baselines (gate-warden-owned)
spec/                     # T0–T1 only; T2+ uses a separate <app>-spec repository
tests/locked/             # software pack; see oracles.md
CLAUDE.md / AGENTS.md     # human-owned and path-protected; never writable by agents
```

## Restart semantics

- **CogWorks** rebuilds the run position from labels, sub-issues, spec PR merge state and
  gate-warden check runs on the latest SHA. It holds no other durable state.
- **gate-warden** is idempotent per (SHA, gate, harness version). Re-running produces the same
  check runs.
- **chain-chronicler's** attestation store and telemetry are evidence. If they are lost, the
  audit trail suffers, but no workflow is blocked or changed. The signed comments in GitHub
  remain the in-band copy of the chain of custody.
