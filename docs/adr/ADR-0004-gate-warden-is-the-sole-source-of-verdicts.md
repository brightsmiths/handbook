# ADR-0004: gate-warden is the only source of pass/fail verdicts

**Status:** Proposed
**Date:** 2026-11-08

## Context

Until now, Brightsmiths deliberately shipped no process referee: nothing enforced that state
transitions were valid or that required approvals existed. CogWorks' design ran a Verifier agent
and its own review gate inside the same job that produced the work. An agent (or
the job that hosts it) certifying its own output is not evidence.

## Decision

- A new service, gate-warden, re-runs every gate from a commit SHA, in a clean sandbox, under
  its own GitHub App.
- It publishes results as check runs and is the **only** App allowed as a required-check source.
- It also verifies that human-only transitions (labels, approvals) were made by authorised
  humans.
- Checks are discipline-pack commands emitting a standard result format.

## Consequences

- No agent claim of success is trusted. Agent-side hooks are fast feedback only.
- The "no process referee" scope gap closes.
- Gate runs cost compute. Mutation and similar checks run on changed code only.

## Alternatives considered

- **Gates inside CogWorks:** the orchestrator and the judge become the same component.
- **An LLM Verifier:** verification is not a task for a probabilistic component.
