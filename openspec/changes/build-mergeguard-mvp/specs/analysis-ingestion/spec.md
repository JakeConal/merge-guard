## Purpose

Import trustworthy, bounded static-analysis evidence with enough immutable source context to support deterministic gating and safe advisory review.

## ADDED Requirements

### Requirement: Bounded analysis bundle ingestion
The system SHALL accept SARIF 2.1.0 bundles from authenticated CI and manual uploads only when required run metadata, source provenance, and configured size limits are satisfied.

#### Scenario: Valid CI bundle is accepted
- **WHEN** an active project credential submits valid SARIF with repository identity, commit SHA, analyzer metadata, and bounded source excerpts
- **THEN** the system creates a run for that exact revision and returns its stable identifier

#### Scenario: Invalid or oversized bundle is rejected
- **WHEN** an uploaded bundle is malformed, exceeds a configured limit, or omits required provenance
- **THEN** the system rejects it atomically with field-level validation errors and creates no partial run

### Requirement: Bounded and verified source context
The system SHALL accept at most 30 surrounding source lines per finding, associate the excerpt with a normalized repository-relative path and line range, and verify it against the supplied file hash.

#### Scenario: Source excerpt matches its provenance
- **WHEN** a finding includes a bounded excerpt whose content and location match the declared file hash and commit metadata
- **THEN** the system marks the finding context as verified for that revision

#### Scenario: Source context is absent or inconsistent
- **WHEN** an otherwise valid finding lacks a usable excerpt or its excerpt does not match the declared provenance
- **THEN** the system imports the finding, marks its context unavailable, and prevents a patch-bearing Resolution Brief

### Requirement: Raw preservation and normalized findings
The system SHALL preserve the accepted raw SARIF run and SHALL normalize supported ESLint and Semgrep results for JavaScript and TypeScript into stable findings without discarding analyzer-specific fields.

#### Scenario: Supported finding is normalized
- **WHEN** a valid run contains an ESLint or Semgrep result for a JavaScript or TypeScript file
- **THEN** the system stores its rule, normalized severity, message, location, fingerprint, provenance, and original analyzer data

#### Scenario: Unsupported result is present
- **WHEN** a valid run contains an unsupported analyzer or file type
- **THEN** the system records the result as skipped, reports the skipped count, and does not silently treat it as an evaluated passing result

### Requirement: Idempotent ingestion
The system SHALL treat repeated uploads of the same project, immutable revision, analyzer identity, and bundle digest as one run.

#### Scenario: CI retries an accepted upload
- **WHEN** CI submits a bundle identical to an already accepted bundle
- **THEN** the system returns the existing run identifier without duplicating findings or audit events

### Requirement: Run lifecycle and history
The system SHALL expose each project's runs in reverse chronological order with processing state, revision, analyzer summary, import counts, skip counts, and gate result when available.

#### Scenario: Member reviews run history
- **WHEN** an authorized project member opens run history
- **THEN** the system shows complete, failed, and processing runs without exposing another project's data

