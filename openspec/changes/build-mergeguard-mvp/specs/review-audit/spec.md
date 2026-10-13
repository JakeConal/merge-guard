## Purpose

Preserve accountable human review decisions and an immutable history explaining how findings and deterministic gate results changed over time.

## ADDED Requirements

### Requirement: Attributed reviewer decisions
The system SHALL record every disposition with the reviewer identity, timestamp, selected action, rationale, finding revision, and any brief consulted.

#### Scenario: Reviewer accepts with reason
- **WHEN** an authorized reviewer records `accepted with reason`
- **THEN** the system requires a non-empty rationale and stores the decision with its exact finding and source revision

#### Scenario: Reviewer disagrees with a brief
- **WHEN** a reviewer chooses an action different from the latest brief recommendation
- **THEN** the system records the disagreement for evaluation without preventing the human decision

### Requirement: Append-only audit history
The system SHALL append tamper-evident audit events for ingestion, policy changes, credential lifecycle, brief generation, reviewer decisions, authorization denials, and gate transitions.

#### Scenario: Existing decision is superseded
- **WHEN** an authorized reviewer changes a prior disposition
- **THEN** the system appends a new decision event and preserves the complete prior history

#### Scenario: Member views finding history
- **WHEN** an authorized project member opens a finding's audit history
- **THEN** the system presents events in chronological order with actor, event type, revision, and resulting gate state

### Requirement: Audit isolation and minimization
The system SHALL restrict audit access to project members and SHALL exclude secrets, authentication tokens, and unnecessary source content from audit records.

#### Scenario: Audit event contains provider failure
- **WHEN** a provider request fails
- **THEN** the audit record contains status and correlation metadata but no API key or unredacted request payload

### Requirement: Reproducible gate transitions
The system SHALL associate every gate transition with the run, policy version, relevant disposition events, and deterministic result inputs.

#### Scenario: Reviewer investigates a changed gate
- **WHEN** a run changes from `FAIL` to `PASS`
- **THEN** the audit history identifies the exact accepted dispositions or subsequent-run resolutions responsible for the transition

