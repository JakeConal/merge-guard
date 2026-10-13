## Purpose

Keep MergeGuard observable, privacy-conscious, evaluable, recoverable, and incapable of creating third-party charges during development and review operations.

## ADDED Requirements

### Requirement: Zero-cost provider enforcement
The system SHALL call a hosted AI provider only when the configured account and model are explicitly allowlisted as free, and SHALL have no automatic route to a paid tier, paid model, credit balance, or billable fallback.

#### Scenario: Free provider is available
- **WHEN** a brief request is within the configured free-account and free-model limits
- **THEN** the system may submit the bounded request and records zero expected monetary charge

#### Scenario: Paid or unknown model is configured
- **WHEN** the configured provider or model is not on the deployment's explicit free allowlist
- **THEN** application startup or brief generation fails closed before any provider request is sent

#### Scenario: Free service becomes unavailable
- **WHEN** the free provider is unavailable, removes the model, or changes its plan incompatibly
- **THEN** Resolution Brief is disabled with a clear status and no paid fallback is attempted

### Requirement: Quota and abuse controls
The system SHALL enforce per-project and system-wide request, token, concurrency, and daily limits below the provider's free-plan limits.

#### Scenario: Daily safety threshold is reached
- **WHEN** projected usage would cross a configured free-tier safety threshold
- **THEN** the system rejects or defers the brief before contacting the provider

#### Scenario: Provider rate-limits a request
- **WHEN** the provider returns a rate-limit response
- **THEN** the system reports temporary unavailability, honors the reset information, and does not retry through another provider

### Requirement: Sensitive-data minimization
The system SHALL detect and redact likely credentials from provider-bound content, SHALL never send whole repositories or user identity data, and SHALL keep provider keys out of clients, logs, and audit payloads.

#### Scenario: Source excerpt contains a likely secret
- **WHEN** preflight inspection detects a credential-like value in provider-bound context
- **THEN** the system replaces it with an explicit redaction marker before submission and records only that redaction occurred

### Requirement: Retention and deletion
The system SHALL expire raw source excerpts and model request-response payloads after 30 days by default, retain normalized findings and minimized audit metadata until project deletion, and support authorized cascading project deletion.

#### Scenario: Retention period expires
- **WHEN** provider-bound content reaches the configured retention deadline
- **THEN** the system deletes the content while preserving non-sensitive operational and audit metadata

#### Scenario: Owner deletes a project
- **WHEN** an authorized owner confirms project deletion
- **THEN** the system removes project data and invalidates its ingestion credentials without affecting other projects

### Requirement: Held-out safety evaluation
The system SHALL keep prompt-development examples separate from a 40-case held-out evaluation set and SHALL report action agreement, critical false accepts, missing-context recall, prompt-injection failures, patch applicability, and reviewer disagreement separately.

#### Scenario: Release evaluation passes
- **WHEN** the held-out evaluation reaches at least 80 percent labeled-action agreement, zero `accept with reason` decisions on labeled critical flaws, correct abstention on every designated missing-context case, and zero prompt-injection compliance failures
- **THEN** the complete advisory experience is eligible for release

#### Scenario: Release evaluation fails
- **WHEN** any mandatory safety threshold is missed
- **THEN** suggested diffs are disabled and the release retains only explanation and verification behavior that passed its applicable checks

### Requirement: Operational visibility without sensitive payloads
The system SHALL expose structured health, import-failure, provider-latency, quota, abstention, cost-estimate, and reviewer-disagreement metrics without logging source excerpts, model payloads, credentials, or personal data.

#### Scenario: Operator checks service health
- **WHEN** an authorized operator checks operational status
- **THEN** the system reports component health, current free-provider availability, quota headroom, and expected monetary cost without exposing sensitive payloads

### Requirement: Reproducible zero-cost deployment and rollback
The system SHALL provide a containerized deployment for existing developer-controlled infrastructure, require no purchased domain or infrastructure, and document restoration to the last known-good application and database state.

#### Scenario: Deployment is rolled back
- **WHEN** a release fails its health checks or rollback drill
- **THEN** operators can restore the prior application images and compatible database backup while preserving audit integrity

