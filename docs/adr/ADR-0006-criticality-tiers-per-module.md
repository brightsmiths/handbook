# ADR-0006: Criticality tiers (T0–T3) assigned per module

**Status:** Proposed
**Date:** 2026-11-08

## Context

The same engine must serve throwaway tools and security-critical components without burdening
the former or under-protecting the latter. Risk varies within a single repository.

## Decision

- Each repository has a human-owned `risk-map.yaml` mapping path globs to tiers T0–T3.
- The tier of a piece of work is the highest tier among the paths it touches. CogWorks proposes
  it, can only raise it, and a human confirms for T2+.
- The tier selects the stage graph, thresholds, oracle depth, human gates and release approval
  (see [workflow.md](../architecture/workflow.md)).

## Consequences

- Replaces CogWorks' safety-critical registry with a single mechanism.
- Thresholds are starting points to calibrate with data.
- Changing the risk map is a human decision, and the map itself is a protected file.

## Alternatives considered

- **A tier per repository:** too coarse. An auth module inside an ordinary service would be
  under-protected.
- **Leaving the tier entirely to an LLM:** not deterministic, and can be manipulated through the
  issue text.
