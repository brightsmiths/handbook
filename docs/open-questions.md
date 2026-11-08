# Open questions

| # | Topic | Status | Notes |
| --- | --- | --- | --- |
| 1 | Names for new services | Open | Working names: `gate-warden`, `sandbox-runner`, `oracle-guard`, `ratchet`, `release-ratifier`, `chain-chronicler`. `trace-keeper` is unavailable (the name is used elsewhere). |
| 2 | Harness repository name and structure | Open | One repository with `disciplines/<name>/`, or a core harness plus one repository per discipline pack? |
| 3 | Label harmonisation | Open | Existing labels: `process:*`, `state:*` (Entry Sentinel, Scope-Sage), `cogworks:run`, `cogworks:security-hold` (CogWorks). Pick a single factory trigger label and holds namespace. |
| 4 | Release PR merge authority at T0/T1 | Open | Auto-merge when G9 is green, or always human? If automated, which service performs the merge? Avoid giving it to a service whose purpose is something else. |
| 5 | Attestation store | Open | Where chain-chronicler stores attestations so that it works both on-prem and in the cloud: an OCI registry, object storage, or GitHub artifact attestations where available. |
| 6 | Holdouts hosting | Open | Private repository per deployment; the access model for the T3 independent verification owner. |
| 7 | Model diversity for oracle vs build | Open | An experiment first, or required at T3? |
| 8 | glitch-guardian scope | Open | Document its responsibilities so overlap with task-tactician is avoided. |
| 9 | Discipline-pack interface | Deferred | Formalise once the software pack is proven (Phase 4). |
| 10 | Non-Rust toolchains | Deferred | C# and TypeScript mappings are documented in [oracles.md](architecture/oracles.md) but not built. |
| 11 | Workflow-engine durability | Closed | Not needed. CogWorks is a step function, and waits are GitHub state (ADR-0001, ADR-0002). |
