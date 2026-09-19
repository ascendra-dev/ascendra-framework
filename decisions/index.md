# Decision Index

Framework-level architecture decision records (ADRs). Each `FW-{NNN}` file is a permanent, immutable ID assigned at the time the decision was made — IDs are never reused or renumbered, so gaps in the sequence are expected as decisions get superseded and removed rather than a sign something is missing. Use `decisions/TEMPLATE.md` when writing a new one.

| ID | Title | Status |
|----|-------|--------|
| FW-001 | [Document Versioning Rule](FW-001-document-versioning.md) | Decided |
| FW-002 | [Related Artifacts Table (brief.md Section 9) Maintenance Strategy](FW-002-related-artifacts-maintenance.md) | Decided — implemented |
| FW-003 | [Template and Example File Conventions](FW-003-template-and-example-conventions.md) | Decided |
| FW-004 | [\[AI Guide\] Label Convention](FW-004-ai-guide-label.md) | Decided |
| FW-005 | [Brief as Intake State](FW-005-brief-as-intake-state.md) | Decided |
| FW-006 | project.json Is Identity Only | Superseded by FW-027 — removed 2026-07-14, see git history |
| FW-007 | [Artifact Ownership Boundaries](FW-007-artifact-ownership-boundaries.md) | Decided |
| FW-008 | [Project Code Naming Convention](FW-008-project-code-naming.md) | Decided |
| FW-009 | [Two-Option Presentation Pattern](FW-009-two-option-presentation.md) | Decided |
| FW-010 | Project Composition Model | Superseded by FW-016 — removed 2026-07-14, see git history |
| FW-011 | [Agent-Based Implementation Architecture](FW-011-agent-based-implementation.md) | Parked — reconsidered; command/chat mode retained until the framework matures |
| FW-012 | [Artifact Co-location](FW-012-artifact-colocation.md) | Decided — implemented |
| FW-013 | [Domain Discovery Architecture](FW-013-domain-discovery-architecture.md) | Decided — implemented |
| FW-014 | [Sub-Domain Decomposition](FW-014-subdomain-decomposition.md) | Decided |
| FW-015 | [Framework Design Strategy](FW-015-framework-design-strategy.md) | Decided |
| FW-016 | [Brief-Driven Extension Model](FW-016-brief-driven-extension-model.md) | Decided — supersedes FW-010, amends FW-006 |
| FW-017 | [Estimation References Model](FW-017-estimation-references-model.md) | Decided — implemented |
| FW-018 | Architecture References Model | Superseded by FW-025 — removed 2026-07-14, see git history |
| FW-019 | [Review Model Redesign](FW-019-review-model-redesign.md) | Decided — implemented |
| FW-020 | [Command Conventions](FW-020-command-conventions.md) | Decided |
| FW-021 | [Template Conventions](FW-021-template-conventions.md) | Decided |
| FW-022 | [Problem Domain / Solution Domain Boundary](FW-022-problem-solution-domain-boundary.md) | Decided |
| FW-023 | [PO-Facing Guidance Voice](FW-023-po-facing-guidance-voice.md) | Decided |
| FW-024 | [Parking Lot for Undiscovered Deferred Capabilities](FW-024-parking-lot-deferred-capabilities.md) | Decided |
| FW-025 | [On-the-Fly Architecture and Standards Generation](FW-025-on-the-fly-architecture-generation.md) | Decided — supersedes FW-018 |
| FW-026 | [Screen Design Phase](FW-026-screen-design-phase.md) | Decided — implementation in progress |
| FW-027 | [Retire `project.json`](FW-027-retire-project-json.md) | Decided — supersedes FW-006 |
| FW-028 | [Artifact Naming Conventions](FW-028-artifact-naming-conventions.md) | Decided |
| FW-029 | [Stack-Agnostic Implementation Commands](FW-029-stack-agnostic-implementation-commands.md) | Decided — extends FW-025 |
| FW-030 | [Project-Agnostic Implementation Commands and Deterministic Story Targeting](FW-030-project-agnostic-implementation-commands.md) | Decided — extends FW-029 |
| FW-031 | [Story Planning Phase Separation](FW-031-story-planning-phase-separation.md) | Decided — implemented, extends FW-029/FW-030 |
| FW-032 | [Screen Design Coverage in /assess-change](FW-032-assess-change-screen-design-coverage.md) | Decided — implemented, amends FW-026 |
| FW-033 | [Standards Compliance in /implement-story](FW-033-implement-story-standards-compliance.md) | Decided — implemented |
| FW-034 | [Architecture Generation Completeness Gaps](FW-034-architecture-generation-completeness-gaps.md) | Decided — implemented |
| FW-035 | [Audit Column Symmetry (updated_by/deleted_by)](FW-035-audit-column-symmetry.md) | Decided — implemented, column shape amended by FW-037 |
| FW-036 | [Audit Column Population Mechanism](FW-036-audit-column-population-mechanism.md) | Decided — implemented, amends FW-035, population contract extended by FW-037 |
| FW-037 | [Audit Column Actor-Label Design](FW-037-audit-column-actor-label-design.md) | Decided — implemented, amends FW-035/FW-036 |
| FW-038 | [Audit Column Shape Discovery in /gen-architecture](FW-038-audit-column-shape-discovery.md) | Decided — implemented, generalizes FW-037 into the framework |
| FW-039 | [Domain-Specific Actor Attribution Symmetry](FW-039-domain-actor-attribution-symmetry.md) | Decided — implemented, project + framework (gen-architecture.md, review-architecture.md check 20a) |
| FW-040 | [Ascendra UI Hard Instructions Registry](FW-040-ascendra-ui-hard-instructions.md) | Decided — implemented |
| FW-041 | [Logging and Exception Handling Completeness](FW-041-logging-and-exception-handling.md) | Decided — implemented, amends `FW-033`/`FW-034` |
| FW-042 | [Client↔Server API Contract](FW-042-client-server-api-contract.md) | Decided — implemented, amends `FW-041` |
| FW-043 | [Frontend Auth Session Module Generation](FW-043-frontend-auth-session-module-generation.md) | Decided — implemented, amends `FW-042` |
| FW-044 | [Frontend Data-Layer Domain Organization](FW-044-frontend-data-layer-domain-organization.md) | Decided — implemented, amends `FW-042`/`FW-043` |
| FW-045 | [Optimistic Concurrency API Contract](FW-045-optimistic-concurrency-api-contract.md) | Decided — implemented, amends `FW-042` |
| FW-046 | [Optimistic Concurrency Client Propagation](FW-046-optimistic-concurrency-client-propagation.md) | Decided — implemented, amends `FW-045` |
| FW-047 | [Optimistic Concurrency Generation Completeness](FW-047-optimistic-concurrency-generation-completeness.md) | Decided — implemented, amends `FW-045`/`FW-046` |
| FW-048 | [Walking Skeleton Milestone](FW-048-walking-skeleton-milestone.md) | Decided — implemented |
| FW-049 | [Anchor Project and Priming Session](FW-049-anchor-project-and-priming-session.md) | Decided — implemented, pending validation |
| FW-050 | [Project Journal and Framework Learning](FW-050-project-journal-and-framework-learning.md) | Decided — implemented, pending validation, amends `FW-049` |
| FW-051 | [Code Priming Session](FW-051-code-priming-session.md) | Decided — implemented, pending validation, depends on `FW-049` |

## Removed decisions

FW-006, FW-010, and FW-018 were superseded and removed from the working tree on 2026-07-14 — each is fully superseded by a later decision (FW-027, FW-016, and FW-025 respectively) with no forward-looking value of its own. Every commit up to and including their removal is preserved in git history; retrieve a removed file with:

```
git log --all --oneline -- 'decisions/*FW-006*'
git show <commit>:decisions/FW-006-project-json-identity-only.md
```
