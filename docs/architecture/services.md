# Service catalogue

Each service has one purpose, and the architecture keeps it that way. Where the factory needs
something new, it gets a new service or an agent profile rather than stretching an existing
service. The **Changes** lines summarise what each service must adjust to fit this
architecture. Design detail, and the issues that track the work, belong in the service's own
repository.

Status key: **exists** (repository with spec and/or code), **planned** (named, no repository
yet), **new** (introduced by this architecture; name is a working name).

## Summary

| Service | Status | Repository | Purpose |
| --- | --- | --- | --- |
| queue-keeper | exists | [brightsmiths/queue-keeper](https://github.com/brightsmiths/queue-keeper) | Webhook intake and routing to queues |
| webhook-watchman | exists (spec) | [brightsmiths/webhook-watchman](https://github.com/brightsmiths/webhook-watchman) | GitHub webhook-delivery metrics |
| Entry Sentinel | exists | [brightsmiths/entry-sentinel](https://github.com/brightsmiths/entry-sentinel) | Intake safety gate for new issues |
| Triage Titan | exists | [brightsmiths/triage-titan](https://github.com/brightsmiths/triage-titan) | Issue triage: priority, type, information requests |
| Scope-Sage | exists | [brightsmiths/scope-sage](https://github.com/brightsmiths/scope-sage) | Issue scoping assessment |
| cadence-commander | planned | — | Cross-repository issue ordering |
| task-tactician | exists | [brightsmiths/task-tactician](https://github.com/brightsmiths/task-tactician) | Issue workflow automation |
| glitch-guardian | exists | [brightsmiths/glitch-guardian](https://github.com/brightsmiths/glitch-guardian) | Project, issue and PR maintenance |
| CogWorks | exists | [brightsmiths/cog-works](https://github.com/brightsmiths/cog-works) | Deterministic foreman |
| agent-gate | exists (description) | [brightsmiths/agent-gate](https://github.com/brightsmiths/agent-gate/tree/description) | Role-based path boundaries for agents |
| task-marshal | exists | [brightsmiths/task-marshal](https://github.com/brightsmiths/task-marshal) | Task broker CLI for agents |
| commit-gate | exists (spec) | [brightsmiths/commit-gate](https://github.com/brightsmiths/commit-gate) | Commit quality checks |
| todo-gate | planned | — | Policy for deferred-work markers |
| review-reeve | planned | — | PR evidence report and reviewer assignment |
| release-regent | exists | [pvandervelde/release-regent](https://github.com/pvandervelde/release-regent) | Rolling release PRs |
| lore-keeper | exists | [brightsmiths/lore-keeper](https://github.com/brightsmiths/lore-keeper) | Semantic search over knowledge bases |
| gate-warden | new | — | Independent gate evaluation; the only source of required checks |
| sandbox-runner | new | — | Isolated microVM execution of stage attempts |
| oracle-guard | new (CLI) | — | Integrity checks on locked software test oracles |
| ratchet | new (CLI) | — | No-regression baselines for code health metrics |
| release-ratifier | new | — | Release gate (G9) and release provenance |
| chain-chronicler | new | — | Signed stage attestations and the telemetry pipeline |

## Ingress and events

### queue-keeper

- **Purpose:** the single public ingress for webhooks. It verifies HMAC signatures, normalises
  events, keeps raw payloads for audit and replay, and routes events to NATS queues (and other
  queue back-ends) with ordered delivery where required.
- **Role in the factory:** unchanged. Every other service consumes queues, never HTTP.
- **Changes:**
  - Add an actor-trust field to normalised events (org member, outside collaborator, external)
    so consumers can refuse to start agent work from external events.
  - Update the spec to reflect NATS support.

### webhook-watchman

- **Purpose:** polls the GitHub delivery APIs and exports delivery success, latency and
  redelivery metrics.
- **Role in the factory:** watches the health of the event path.
- **Changes:** none.

## Intake and ordering

### Entry Sentinel

- **Purpose:** decides whether a new or edited issue is safe to enter the pipeline. On approval
  it posts a signed approval comment; on rejection it quarantines the issue.
- **Role in the factory:** the first link in the chain of custody. Every downstream service
  verifies its signature before acting.
- **Changes:**
  - The optional LLM second pass runs quarantined: no tools, no repository access, fixed output
    schema.
  - Issues created by factory identities (for example drift-scout health issues) are scanned
    like any other issue.
  - Ignore events from external actors that would start factory work.
- **Does not:** judge quality, priority or feasibility.

### Triage Titan

- **Purpose:** issue triage. It assigns priority and type labels and asks for missing
  information.
- **Role in the factory:** supplies priority and type for ordering, and tells CogWorks whether
  an issue has enough information to specify.
- **Changes:**
  - Verify Entry Sentinel's attestation before reading content.
  - Emit a signed, machine-readable triage block next to the labels, so downstream services do
    not parse prose.
- **Does not:** assess architecture, scope or tier.

### Scope-Sage

- **Purpose:** an issue scoping assessment, posted as a signed issue comment: strategic
  alignment, affected areas, architecture fit, relevant ADRs, cross-cutting concerns and open
  questions. Runs only when a human applies `state:design`.
- **Role in the factory:** input to the human decision to proceed, and to CogWorks' tier
  proposal.
- **Changes:**
  - Add a machine-readable "affected areas" block (paths and modules) that CogWorks can match
    against the repository's `risk-map.yaml`.
- **Does not:** do architecture work, write code, change state or create sub-issues.
  Architecture is a CogWorks stage. See [workflow.md](workflow.md).

### cadence-commander

- **Purpose:** keeps a WSJF-scored ordering of **issues** across repositories.
- **Role in the factory:** CogWorks takes the next ready issue in cadence-commander's order.
- **Changes (for the initial design):**
  - Reserve a configurable share of the ordering (start at 20%) for issues labelled as health
    work, so health work is not crowded out by features.
- **Does not:** order tasks or slices within an issue (CogWorks does that from the plan's
  dependency graph), enforce budgets, or merge releases.

### task-tactician

- **Purpose:** issue workflow automation: branches, workflow labels, PR/issue linking labels,
  dependency tracking, cycle-time metrics and stale-issue detection.
- **Role in the factory:** keeps workflow labels and dependency tracking consistent for the
  sub-issues that CogWorks' plan stage creates.
- **Changes:**
  - Recognise the factory's label namespaces (see [state-model.md](state-model.md)).
  - Do not create branches for issues CogWorks owns; CogWorks has its own branch scheme.
  - Remove organisation-specific references from the spec.

### glitch-guardian

- **Purpose:** project, issue and PR maintenance.
- **Role in the factory:** unchanged. Its scope still needs writing down; see
  [open-questions.md](../open-questions.md).

## Deterministic control plane

### CogWorks

- **Purpose:** the deterministic foreman. A step function that reads GitHub state, decides the
  next stage, runs it, writes the result back to GitHub and exits.
- **Role in the factory:** owns the stage graph, retries, escalation, profile selection and
  context bundles. Has no built-in agent modes.
- **Changes:**
  - **Remove built-in agent modes** (architect, coder, tester…). A node becomes a *stage type*.
    For each stage, CogWorks picks an agent profile from the harness by stage type, discipline,
    language and tier. See [agent-profiles.md](agent-profiles.md).
  - **Split code generation** into separate *oracle* and *implement* stages that run under
    different identities in different sandboxes.
  - **Build a context bundle** for each stage from the profile's input allowlist, and hash it.
    Context Packs feed into this.
  - **Delegate execution** to `sandbox-runner` through the existing runtime-provider interface.
  - **Delegate pass/fail** to `gate-warden`. CogWorks reads the verdict from check runs and
    never computes it itself.
  - **Verify upstream attestations** (Entry Sentinel, Triage Titan, Scope-Sage) instead of
    repeating their checks. The Intake node's own injection scan is removed.
  - **Derive the tier** from `risk-map.yaml` plus Scope-Sage's affected areas, replacing the
    safety-critical registry. The tier can only be raised, as with the existing safety
    override.
  - **Load prompt templates, output schemas and constitutional rules from the pinned harness
    version** instead of from CogWorks itself.
- **Does not:** merge, release, or decide whether gates pass.

### agent-gate

- **Purpose:** enforces which role may write where, from a single policy file.
- **Role in the factory:** the single source for the visibility and write matrix
  ([agent-profiles.md](agent-profiles.md)). The same policy drives sandbox mounts, the
  commit-level scope check and push rulesets.
- **Changes:**
  - Replace the fixed tester/coder modes with roles declared by the harness's agent profiles.
  - Produce a **mount plan** for sandbox-runner: read-write, read-only or absent per path, plus
    file ownership and sticky directories for locked files that sit next to source files.
  - Provide the commit-range **scope check** as a CLI that gate-warden runs.
  - Hand quality checks (formatting, linting, tests) to commit-gate and gate-warden, so
    agent-gate stays about boundaries.
  - Remove organisation-specific defaults from the example config.

### sandbox-runner *(new)*

- **Purpose:** runs one stage attempt in a short-lived, strongly isolated microVM, then destroys
  it.
- **Responsibilities:**
  - Apply agent-gate's mount plan.
  - Read-only root filesystem, non-root user, no secrets inside the microVM.
  - Deny-by-default egress through a proxy that allows only the model gateway and the dependency
    proxy.
  - Enforce wall-clock, CPU and memory limits.
  - After a scope pre-check, push the agent's commits using a short-lived token for the role's
    GitHub App. The agent never holds that token.
- **Providers:** microsandbox (reference provider), Firecracker, Kata Containers. See
  [isolation-and-security.md](isolation-and-security.md).
- **Why separate from CogWorks:** the orchestrator never holds host or virtualisation
  privileges.

### gate-warden *(new)*

- **Purpose:** decides pass or fail. It re-runs every gate from a commit SHA in a clean sandbox
  and publishes the results as check runs under its own GitHub App. It is the **only** App
  allowed as the source of required checks.
- **Responsibilities:**
  - Run the checks for the gate (G1–G9) as declared in the pinned harness, with thresholds per
    tier.
  - Verify signatures (oracle manifests, upstream attestations).
  - Verify that human-only transitions (labels, approvals) were made by authorised humans.
    This fills the "no process referee" gap.
  - Return structured, coarse feedback to CogWorks.
- **Discipline-neutral:** checks are commands that emit a standard result format. Discipline
  packs supply them.

### chain-chronicler *(new)*

- **Purpose:** evidence and attribution.
- **Responsibilities:**
  - A shared signing library, generalising the Ed25519 signing already used by Entry Sentinel,
    Scope-Sage and lore-keeper, that produces in-toto/DSSE attestations for each stage.
  - An append-only attestation store.
  - The OTel pipeline and event schema for the factory scorecard.
- **Rule:** attestations and telemetry are evidence, not state. Workflow decisions read GitHub
  only.

## Agent plane

### task-marshal

- **Purpose:** gives agents a single, stable CLI for fetching and completing tasks.
- **Role in the factory:** the agent's only interface to its stage, inside the sandbox.
- **Changes:**
  - Add a CogWorks bundle source: `next` returns the stage bundle and `done` submits
    schema-validated output.
  - Add `dispute`, so an agent can raise a test dispute (or test change request) instead of
    editing the oracle.

### commit-gate

- **Purpose:** commit quality checks (format, conventional commits, configured tool pipeline).
  Supports hook mode and git-wrapper mode.
- **Role in the factory:** fast feedback inside the sandbox through git-wrapper mode. This is
  defence in depth only; the authoritative run is gate-warden's.
- **Changes:** add a CI/verify mode that gate-warden can run against a commit range.

## Check tools

### todo-gate *(planned)*

- **Purpose:** compares `TODO`/`FIXME`/`todo!()`/`unimplemented!()` markers against an
  allowlist with two-level fingerprinting, and reports allowlist entries that have gone stale.
- **Role in the factory:** a gate-warden check, plus fast feedback in the sandbox.
- **Changes (for the initial design):**
  - The allowlist is a protected path that no agent can write.
  - New entries count as exceptions in the review evidence report.

### oracle-guard *(new, software pack)*

- **Purpose:** integrity checks on locked software test oracles:
  - `lock`: signed manifest
  - `red-check`: every new oracle test must fail against stubs
  - `inventory`: every expected test ran and passed, none skipped
  - `config-guard`: test-affecting configuration matches the manifest
  - `sniff`: detects test-only behaviour in production code
  - `holdout-inject`: adds hidden tests at gate time
- See [oracles.md](oracles.md).

### ratchet *(new, software pack)*

- **Purpose:** keeps baselines for complexity, duplication, warnings, suppressions, coupling and
  mutation score on changed code. Values may stay flat or improve; any regression must be
  recorded as an explicit exception.
- **Role in the factory:** a gate-warden check. Baselines are written only by gate-warden.

## Review and release

### review-reeve *(planned)*

- **Purpose:** prepares PRs for human review: a structured report, plus reviewer assignment
  from CODEOWNERS and domain signals. It is additive and never blocks.
- **Role in the factory:** builds the human evidence pack.
- **Changes (for the initial design):**
  - Build the report from artifacts and evidence only (requirement-to-test trace, gate results,
    sensor exceptions, mutation survivors, reviewer-agent findings), never from the
    implementing agent's own PR narrative.
  - Assign the human who approved the program design (H3) as the designated reviewer.

### release-regent

- **Purpose:** keeps a rolling release PR against the default branch, recalculates the version
  and release notes on each merge, and tags the commit and creates the GitHub Release when the
  release PR merges.
- **Role in the factory:** unchanged.

### release-ratifier *(new)*

- **Purpose:** the release gate (G9) and release provenance.
- **Responsibilities:**
  - Act as a required check on release-regent's release PR: verify that every included change
    has a complete attestation chain, that an SBOM was generated, and that the approvals
    required by the tier are present (T2: release owner; T3: plus security sign-off).
  - After release-regent publishes, attach the SBOM, SLSA provenance and the attestation bundle
    to the GitHub Release.
- **Does not:** upload artifacts or deploy.

## Knowledge

### lore-keeper

- **Purpose:** semantic search over engineering knowledge bases (ADR corpora across
  repositories), exposed as MCP tools. Verifies signatures before indexing. Does no LLM
  synthesis.
- **Role in the factory:** design-stage profiles may get lore-keeper as a read-only MCP tool,
  and humans use it directly. Results are treated as data.
- **Changes:** index this handbook's ADRs and the factory spec directories.
