## Purpose

Provide a bounded, evidence-linked AI advisory for one supported finding while preventing model output from making decisions or mutating project state.

## ADDED Requirements

### Requirement: Single-finding eligibility
The system SHALL generate a Resolution Brief only for one supported ESLint or Semgrep finding in a JavaScript or TypeScript file with rule metadata and bounded source evidence.

#### Scenario: Eligible finding is submitted
- **WHEN** an authorized reviewer requests a brief for an eligible finding with verified context
- **THEN** the system constructs one bounded request containing only that finding, its rule metadata, at most 30 surrounding lines, and approved project guidance

#### Scenario: Finding lacks sufficient context
- **WHEN** an otherwise supported finding lacks verified context needed for explanation or patching
- **THEN** the system returns `needs more context`, identifies the missing evidence, and produces no diff

### Requirement: Fixed advisory schema
The system SHALL expose a completed brief only after it validates against a fixed schema containing action, confidence, evidence-linked explanation, optional minimal diff, two verification steps, and missing-context details.

#### Scenario: Provider returns a valid brief
- **WHEN** the provider response satisfies the complete schema and safety validation
- **THEN** the system stores and displays it as an advisory result labeled with provider, model, prompt version, and source revision

#### Scenario: Provider returns invalid output
- **WHEN** a provider response is malformed, violates the schema, or fails safety validation
- **THEN** the system discards it, records a non-sensitive failure event, and exposes no partial advisory as a valid brief

### Requirement: Evidence-linked explanation
The system SHALL require every factual claim about the finding's source to cite lines inside the supplied excerpt and SHALL reject citations outside that evidence.

#### Scenario: Explanation cites available evidence
- **WHEN** a brief attributes a root cause to source code
- **THEN** each attribution references one or more supplied source lines that exist in the verified excerpt

### Requirement: Revision-bound minimal diff
The system SHALL permit at most one minimal diff, only for `fix now`, and SHALL bind it to the finding's repository-relative path, commit SHA, and file hash.

#### Scenario: Current revision receives a patch
- **WHEN** a valid `fix now` brief includes a diff that applies within the bounded excerpt and does not rewrite the entire file
- **THEN** the system labels the diff as advisory and displays its bound revision

#### Scenario: Source changes after generation
- **WHEN** the stored file hash no longer matches the revision being reviewed
- **THEN** the system marks the diff stale and prevents it from being presented as applicable

### Requirement: Safe verification guidance
The system SHALL provide exactly two non-mutating verification steps drawn from project-approved commands or expressed as manual checks when no approved command applies.

#### Scenario: Generated command is not approved
- **WHEN** a provider proposes a command outside the project's approved command patterns
- **THEN** the system rejects that command and does not expose it as executable guidance

### Requirement: Prompt-injection resistance
The system SHALL treat rule text, source excerpts, comments, filenames, and guidance attachments as untrusted data that cannot alter system policy, enable tools, request secrets, or change the output schema.

#### Scenario: Source comment contains instructions
- **WHEN** the source excerpt asks the model to ignore policy, disclose secrets, or return a different response
- **THEN** the resulting brief ignores the instruction and continues to satisfy the required schema and evidence rules

### Requirement: Advisory-only behavior
The system SHALL NOT execute provider output, commit code, dismiss findings, record reviewer decisions, or alter a gate as a consequence of generating a brief.

#### Scenario: Brief generation completes
- **WHEN** a brief is successfully stored
- **THEN** the finding disposition and gate are unchanged until an authorized reviewer performs a separate decision action

### Requirement: Observable generation lifecycle
The system SHALL report generation lifecycle states and SHALL expose the final brief only after complete validation.

#### Scenario: Reviewer waits for a brief
- **WHEN** generation progresses through submission, provider processing, and validation
- **THEN** the client receives status updates without receiving an unvalidated partial brief

