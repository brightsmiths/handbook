# ADR-0001: GitHub is the only system of record

**Status:** Proposed
**Date:** 2026-11-08

## Context

A deterministic orchestrator needs somewhere to keep run state, including runs that wait days
for a human decision. Common choices are a protected branch, a database or a workflow engine
(Step Functions, Temporal). Brightsmiths services already keep all durable state in GitHub.
CogWorks is designed as a step function that reads GitHub state.

## Decision

All work, state, decisions and approvals live in GitHub:

- issues, labels and signed comments
- pull requests and reviews
- check runs, commits and releases

No service owns workflow state outside GitHub. Attestation stores and telemetry are *evidence*:
they can be lost without blocking or changing any workflow.

## Consequences

- Every service can be restarted and rebuild its view from GitHub.
- Waiting on a human for days needs no workflow engine. It is a state with no pending action.
- Throughput is limited by GitHub API rate limits; services must be economical with reads.
- Every decision is visible to anyone with repository access.

## Alternatives considered

- **A workflow engine (Step Functions, Temporal):** durable waits, but a second source of truth,
  and cloud lock-in or another platform to run.
- **A database for run state:** faster queries, but breaks restart-from-GitHub and hides state
  from humans.
