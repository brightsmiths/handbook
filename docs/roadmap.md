# Roadmap

This page covers how the system gets from today's services to the architecture in this handbook.
Each phase's output is useful on its own, even if later phases never happen. Track per-service
work as issues in that service's repository and link them here.

## Phase 0: architecture and housekeeping

- [ ] Accept or amend the ADRs in [adr/](adr/).
- [ ] Settle the names of the new services ([open-questions.md](open-questions.md)).
- [ ] Harmonise the label taxonomy across Entry Sentinel, Scope-Sage, task-tactician and CogWorks
      ([state-model.md](architecture/state-model.md)).
- [ ] Create the harness repository with `disciplines/software/`, plus the harness-evals
      repository.
- [ ] Create repositories for review-reeve, cadence-commander and todo-gate.
- [ ] Fix documentation drift in service repositories:
  - [ ] queue-keeper: document NATS support.
  - [ ] task-tactician: remove organisation-specific references.
  - [ ] agent-gate: remove organisation-specific example config.
- [ ] Update the short landing summary in `brightsmiths/.github`.

## Phase 1: trust foundations

*Goal: a dummy agent cannot write outside its paths, reach the internet, see secrets, or forge a
check. Prove it by red-teaming.*

- [ ] **sandbox-runner** with the microsandbox provider, egress proxy, model gateway and
      dependency proxy.
- [ ] **GitHub Apps per role**, with short-lived tokens issued from the KMS.
- [ ] **agent-gate:** roles from the harness, mount plans, a CLI scope check.
- [ ] **gate-warden MVP:** the only required-check source. Runs the agent-gate scope check,
      commit-gate CI mode and todo-gate.
- [ ] **oracle-guard (Rust):** lock, red-check, inventory (cargo-nextest JUnit), config-guard,
      sniff.
- [ ] **CogWorks:**
  - [ ] Replace built-in agent modes with profile selection.
  - [ ] Split the oracle and implement stages.
  - [ ] Build context bundles and delegate execution to sandbox-runner.
  - [ ] Read verdicts from gate-warden.
  - [ ] **Timing:** land these before the remaining node implementations (#58–#62). The
        in-flight architecture-node work (#59) keeps its deterministic constraint validation;
        "architecture" becomes a stage type that runs the architect profile.
- [ ] **task-marshal:** CogWorks bundle source, `dispute`.

## Phase 2: shadow-dark on one T1 Brightsmiths repository

*Goal: the build loop runs dark on a small Rust repository (for example commit-gate), while
humans still merge.*

- [ ] Full intake chain with attestations verified at every hop: Entry Sentinel (quarantined LLM
      pass) → Triage Titan (triage block) → Scope-Sage (affected areas) → CogWorks.
- [ ] Software pack profiles: specify, contract, plan, unit-oracle, implement, refactor.
- [ ] `risk-map.yaml` and `.factory/pin.yaml` in the pilot repository.
- [ ] ratchet and mutation-on-changed-code checks.
- [ ] review-reeve MVP: evidence report, advisory only.
- [ ] chain-chronicler signing library used by CogWorks and gate-warden.

**Exit (decide with data over about 6 weeks):** the rate of approvals that change nothing and the
reasons for changes; zero oracle-integrity bypasses; mutation score stable or rising; cost per
slice known. If humans routinely change agent output, stop at a light factory with better
tooling. That is still a good outcome.

## Phase 3: T2 pilot

- [ ] Spec / impl / acceptance repository split; a private holdouts repository; holdout
      injection.
- [ ] Architect profile and H2/H3 gates through spec PR reviews; designated reviewer assignment
      in review-reeve.
- [ ] **release-ratifier** on release-regent's release PRs; SBOM and provenance attached to
      releases.
- [ ] chain-chronicler attestation store and telemetry pipeline; factory scorecard.
- [ ] Health loop: drift-scan profile, H7 triage, cadence-commander health share, debt-fix
      profile.

**Exit:** the visible-vs-holdout gap is small and stable; escaped defects are no worse than the
human baseline; files changed per change stays flat.

## Phase 4: harden and widen

- [ ] Harness evals with golden tasks and cheat traps; harness pinning and canary rollout.
- [ ] T3 profile: Kani, extended fuzz campaigns, two-person gates, independent verification
      owner.
- [ ] A documented interface for third-party discipline packs.
- [ ] C# and TypeScript toolchain support in the software pack.

**Exit:** every cheat trap is disputed and none passes; no harness release regresses the
scorecard; a walkthrough of the provenance chain from requirement to release succeeds.

## Changes per existing service

| Service | Change | Phase |
| --- | --- | --- |
| queue-keeper | Actor-trust field on normalised events; spec update for NATS | 0–1 |
| Entry Sentinel | Quarantined LLM pass; refuse events from external actors | 2 |
| Triage Titan | Verify Entry Sentinel attestation; signed machine-readable triage block | 2 |
| Scope-Sage | Machine-readable affected-areas block | 2 |
| cadence-commander | (New repo) WSJF issue ordering with a reserved health share | 3 |
| task-tactician | Factory label namespaces; leave CogWorks branches alone | 2 |
| CogWorks | Profiles instead of modes; oracle/implement split; bundles; sandbox-runner; gate-warden verdicts; upstream attestation verification; risk-map tiers; harness-loaded prompts | 1–2 |
| agent-gate | Harness-defined roles; mount plans; scope-check CLI; boundaries only | 1 |
| task-marshal | CogWorks bundle source; `dispute` | 1 |
| commit-gate | CI/verify mode for gate-warden | 1 |
| todo-gate | (New repo) protected allowlist; exceptions reported | 1–2 |
| review-reeve | (New repo) evidence-based report; designated reviewer | 2–3 |
| release-regent | None | — |
| lore-keeper | Index handbook ADRs and factory spec directories | 3 |
| webhook-watchman, glitch-guardian | None | — |
