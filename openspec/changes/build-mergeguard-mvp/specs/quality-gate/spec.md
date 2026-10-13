## Purpose

Give reviewers a transparent and reproducible merge decision based on configured policy and human dispositions rather than model judgment.

## ADDED Requirements

### Requirement: Deterministic run gate
The system SHALL calculate a run's gate solely from normalized findings, project policy, and recorded reviewer dispositions; Resolution Brief content SHALL have no direct or indirect gate effect.

#### Scenario: Blocking finding remains unresolved
- **WHEN** a complete run contains at least one policy-blocking finding without an accepted disposition
- **THEN** the run gate is `FAIL` and identifies the unresolved blocking count

#### Scenario: No blocking finding remains
- **WHEN** a complete run contains no unresolved policy-blocking findings
- **THEN** the run gate is `PASS`

#### Scenario: Brief recommends acceptance
- **WHEN** a Resolution Brief recommends `accept with reason` but no authorized reviewer records that disposition
- **THEN** the gate remains unchanged

### Requirement: Explicit gate policy
The system SHALL expose a project policy composed of a normalized blocking severity threshold and optional rule-specific overrides, with a safe default that blocks unresolved error-level findings.

#### Scenario: Rule override blocks a lower-severity finding
- **WHEN** a project policy marks a specific lower-severity rule as blocking
- **THEN** an unresolved occurrence of that rule causes the run gate to fail

#### Scenario: Policy changes after a run
- **WHEN** an authorized project owner changes gate policy
- **THEN** the system records the change and recalculates affected gate results deterministically

### Requirement: Reviewer disposition semantics
The system SHALL support `unresolved`, `fix required`, and `accepted with reason` review states; only `accepted with reason` removes an existing blocking finding from the unresolved blocking count.

#### Scenario: Reviewer requires a fix
- **WHEN** an authorized reviewer records `fix required`
- **THEN** the finding remains blocking until a later run no longer contains its stable fingerprint

#### Scenario: Reviewer accepts a finding
- **WHEN** an authorized reviewer records `accepted with reason` with a non-empty rationale
- **THEN** the finding no longer blocks that run and the gate is recalculated

### Requirement: Finding exploration
The system SHALL let authorized members filter findings by severity, rule, file, support status, and review state and SHALL present exact run and source provenance on finding detail.

#### Scenario: Reviewer narrows a large run
- **WHEN** a reviewer combines severity, rule, and file filters
- **THEN** the system returns only matching findings and preserves the active filters in the view

#### Scenario: Reviewer opens a finding
- **WHEN** an authorized reviewer opens finding detail
- **THEN** the system shows rule metadata, message, exact location, verified excerpt status, disposition, gate contribution, and linked audit history

