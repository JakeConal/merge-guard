## Purpose

Provide authenticated, project-scoped access so developer-team data, reviewer actions, and ingestion credentials remain isolated between organizations and projects.

## ADDED Requirements

### Requirement: Authenticated team access
The system SHALL require an authenticated user session for every interactive project, run, finding, brief, decision, and audit operation.

#### Scenario: Anonymous access is rejected
- **WHEN** a user without a valid session requests project data
- **THEN** the system rejects the request without revealing whether the project exists

#### Scenario: Team member signs in
- **WHEN** a registered team member supplies valid credentials
- **THEN** the system creates a time-limited authenticated session and returns only projects assigned to that member

### Requirement: Project membership authorization
The system SHALL authorize every project-scoped request against the authenticated user's active membership and role.

#### Scenario: Member views an assigned project
- **WHEN** an active project member requests runs or findings for that project
- **THEN** the system returns the requested project data

#### Scenario: Member attempts cross-project access
- **WHEN** a user requests data or an action for a project where the user has no active membership
- **THEN** the system rejects the request and records an authorization-denied event without disclosing project data

### Requirement: Role-restricted administration and review
The system SHALL restrict membership and ingestion-credential administration to project owners and SHALL restrict finding dispositions to project owners and reviewers.

#### Scenario: Reviewer records a disposition
- **WHEN** an active reviewer records a valid decision for a finding in the reviewer's project
- **THEN** the system accepts the decision and attributes it to that reviewer

#### Scenario: Ordinary member attempts a reviewer action
- **WHEN** a project member without reviewer or owner authority attempts to record a disposition
- **THEN** the system rejects the action without changing the finding or gate

### Requirement: Revocable project ingestion credentials
The system SHALL issue project-scoped ingestion credentials that are displayed once, stored only in non-recoverable form, and independently revocable or rotatable.

#### Scenario: Valid ingestion credential is used
- **WHEN** an ingestion request presents an active credential for its target project
- **THEN** the system authenticates the request only for ingestion into that project

#### Scenario: Credential is rotated
- **WHEN** a project owner rotates an ingestion credential
- **THEN** the previous credential stops working immediately and the replacement is displayed once

#### Scenario: Credential cannot access interactive APIs
- **WHEN** an ingestion credential is presented to a project, review, or administration endpoint
- **THEN** the system rejects the request

