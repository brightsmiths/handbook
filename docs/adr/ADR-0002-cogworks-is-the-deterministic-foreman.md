# ADR-0002: CogWorks is the deterministic foreman and selects agent profiles per stage

**Status:** Proposed
**Date:** 2026-11-08

## Context

CogWorks' current design builds agent modes into the product (Architect, Planner, Coder, Tester,
QA, Security Reviewer, Tech Lead) and runs tests and implementation in one container. The
architecture needs a deterministic orchestrator that decides what runs, while the agents
themselves vary by the work's needs and engineering discipline.

## Decision

- CogWorks is the deterministic foreman: a step function that owns the stage graph, retries,
  escalation and context bundles.
- CogWorks has **no built-in agent modes**. A pipeline node is a *stage type*. For each stage,
  CogWorks deterministically picks an agent profile from the pinned harness by (stage type,
  discipline, language, tier).
- CogWorks does not execute agents itself (sandbox-runner does) and does not decide pass/fail
  (gate-warden does).

## Consequences

- New disciplines and languages are added in the harness, not in CogWorks.
- The remaining node work (#58–#62) and the in-flight architecture node (#59) must be reshaped
  around stage types.
- Profile selection is recorded in each stage attestation, for attribution.

## Alternatives considered

- **An LLM conductor routing work (as in AWS AI-DLC):** no information barrier between stages,
  and routing becomes probabilistic.
- **Keeping built-in modes:** ties CogWorks to software engineering and to one way of dividing
  roles.
