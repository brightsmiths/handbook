# Architecture decision records

System-level decisions for Brightsmiths. Decisions about a single service's internals live in
that service's repository.

| ADR | Title | Status |
| --- | --- | --- |
| [0001](ADR-0001-github-is-the-system-of-record.md) | GitHub is the only system of record | Proposed |
| [0002](ADR-0002-cogworks-is-the-deterministic-foreman.md) | CogWorks is the deterministic foreman and selects agent profiles per stage | Proposed |
| [0003](ADR-0003-trust-zones-and-separate-oracle-authorship.md) | Trust zones; oracle authored separately and locked | Proposed |
| [0004](ADR-0004-gate-warden-is-the-sole-source-of-verdicts.md) | gate-warden is the only source of pass/fail verdicts | Proposed |
| [0005](ADR-0005-microvm-isolation-per-stage-attempt.md) | Strong isolation: one microVM per stage attempt | Proposed |
| [0006](ADR-0006-criticality-tiers-per-module.md) | Criticality tiers assigned per module | Proposed |
| [0007](ADR-0007-discipline-packs.md) | Discipline-neutral core; discipline packs | Proposed |
| [0008](ADR-0008-small-services-stable-purposes.md) | Small services with stable purposes | Proposed |

New ADRs use [ADR_TEMPLATE.md](ADR_TEMPLATE.md).
