# ADR-0003: Agent boundaries follow trust zones; the oracle is authored separately and locked

**Status:** Proposed
**Date:** 2026-11-08

## Context

If the agent that writes the code can also see or edit the tests, it can make tests and code
agree without meeting the spec. Splitting agents by job title adds hand-off loss without
necessarily creating a barrier.

## Decision

- Agents are grouped into trust zones: A intent/design, B oracle, C build, D assurance,
  E health, Q quarantine. Agents are split only across zones.
- The oracle (tests, acceptance checks, holdouts) is written by a zone B identity, locked by a
  signed manifest before implementation, and is never writable by zone C.
- The generating agent disagrees with the oracle through a dispute, never an edit.
- At T2+, a hidden subset of the oracle (holdouts) is stored privately and only added at gate
  time.

## Consequences

- Separate GitHub App identities, sandboxes and path policies per zone.
- Each discipline pack must define what "locked oracle" means for it.
- Locked tests make seams costly to change. Seam changes become design changes (test change
  requests).

## Alternatives considered

- **One workspace for Tester and Coder with a "clean context break":** no enforced barrier.
- **Hiding all tests from the implementer:** prevents gaming, but reduces legitimate task
  success. A visible set is kept.
