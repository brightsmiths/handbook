# ADR-0008: Small services with stable purposes

**Status:** Proposed
**Date:** 2026-11-08

## Context

Adopting the factory design could tempt us to repurpose existing services to fill new roles, for
example turning an issue scoping service into an architecture agent, or an ordering service into
a budget enforcer. That would blur each service's purpose and make the services harder to
reason about.

## Decision

- Each Brightsmiths service keeps its existing purpose. Its functionality may be tweaked to fit
  the factory, but not drastically changed.
- New responsibilities go into new small services (gate-warden, sandbox-runner,
  release-ratifier, chain-chronicler, oracle-guard, ratchet) or into agent profiles in the
  harness.
- Services hand work to each other only through GitHub state and queue-keeper events, never
  through direct calls.
- This handbook holds system-level documentation and architecture only. Each service's design
  lives in its own repository.

## Consequences

- More services to deploy, each with a clear, testable boundary.
- Responsibilities that look adjacent to an existing service are deliberately kept out of it.
  Examples: Scope-Sage does issue scoping, not architecture; cadence-commander orders issues,
  not tasks; release-regent manages release PRs, not the release gate.

## Alternatives considered

- **Folding new roles into the nearest existing service:** fewer deployables, but purposes
  drift and boundaries erode.
