# ADR-0005: Strong isolation, one microVM per stage attempt

**Status:** Proposed
**Date:** 2026-11-08

## Context

Agents will sometimes be successfully prompt-injected. Their blast radius must be limited by the
platform, not by agent behaviour. The system must run both on-prem and in the cloud.

## Decision

- Every agent stage attempt, and every gate-warden run, executes in a fresh microVM that is
  destroyed afterwards.
- Inside the microVM: read-only root filesystem, non-root user, no secrets, and deny-by-default
  egress through a proxy that allows only the model gateway and the dependency proxy.
- A new service, sandbox-runner, owns this. It applies agent-gate's mount plan and pushes
  commits on the agent's behalf using short-lived, role-scoped GitHub App tokens.
- The reference provider is microsandbox (libkrun). Firecracker and Kata Containers are
  supported through CogWorks' provider-neutral runtime interface. gVisor is allowed as a T0/T1
  fallback only.

## Consequences

- Hosts must support KVM: bare metal on-prem, metal or nested-virtualisation instances in the
  cloud.
- Per-attempt startup cost. Images and dependency caches must be read-only and pre-warmed.
- The orchestrator never holds virtualisation privileges.

## Alternatives considered

- **Plain containers:** a shared kernel is too weak a boundary for untrusted agent actions.
- **Agent-side permission hooks as the main control:** they run inside the agent process, and
  bypasses have been demonstrated.
