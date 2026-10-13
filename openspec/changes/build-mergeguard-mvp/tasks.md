## 1. Repository and Runtime Foundation

- [ ] 1.1 Create the `web`, `api`, `ci`, `eval`, and `infra` project structure with pinned Node, Java 21, and build-tool versions; verify clean frontend and backend builds succeed from documented commands.
- [ ] 1.2 Add Dockerfiles and a development Docker Compose stack for Next.js, Spring Boot, and PostgreSQL; verify all containers start and report healthy without a purchased service.
- [ ] 1.3 Define typed runtime configuration for database, OAuth, encryption, retention, and zero-cost AI settings; verify startup rejects missing secrets, unknown providers, paid models, and configured fallbacks before external traffic occurs.
- [ ] 1.4 Add formatting, linting, type-checking, backend unit tests, database integration tests, and Playwright jobs to CI; verify a deliberately failing check blocks the workflow.
- [ ] 1.5 Configure Spring OpenAPI generation and a generated TypeScript client; verify CI detects an outdated generated client when the API contract changes.

## 2. Persistence, Identity, and Project Access

- [ ] 2.1 Add database migrations for users, projects, memberships, ingestion credentials, runs, findings, source contexts, policy versions, dispositions, briefs, usage counters, and audit events; verify migrations apply to an empty database and schema constraints reject invalid cross-project references.
- [ ] 2.2 Implement application-layer encryption for raw SARIF, source excerpts, and provider payloads plus keyed hashing for ingestion credentials; verify round-trip encryption and non-recoverable token-storage tests pass.
- [ ] 2.3 Implement identity-only GitHub OAuth, secure server sessions, logout, expiry, and rotation; verify OAuth callback and anonymous-access integration tests pass without requesting repository-content scopes.
- [ ] 2.4 Add the disabled-by-default test identity adapter for automated tests; verify production configuration cannot start with the adapter enabled.
- [ ] 2.5 Implement project creation, membership management, and `owner`/`reviewer`/`member` authorization; verify the role matrix and cross-project isolation integration tests pass.
- [ ] 2.6 Implement one-time ingestion credential creation, rotation, revocation, and endpoint scoping; verify revoked credentials fail immediately and cannot call interactive APIs.
- [ ] 2.7 Build sign-in, project list, membership, and ingestion-credential screens; verify Playwright covers owner administration and an unauthorized member rejection.

## 3. Enriched SARIF Ingestion

- [ ] 3.1 Define and publish the versioned enriched-bundle schema with SARIF 2.1.0, immutable revision metadata, analyzer identity, excerpts, line ranges, file hashes, source-run identifier, and bundle digest; verify valid and invalid contract fixtures.
- [ ] 3.2 Implement the CI packaging utility that reads ESLint and Semgrep SARIF and emits at most 30 surrounding lines per supported finding; verify fixtures contain normalized relative paths, correct line ranges, matching SHA-256 hashes, and no unrelated files.
- [ ] 3.3 Add packaging-time secret detection and redaction markers without changing line numbering; verify credential fixtures are redacted and ordinary source remains unchanged.
- [ ] 3.4 Implement bounded request parsing for content type, byte size, nesting depth, run count, result count, path traversal, line ranges, and required provenance; verify malformed or oversized bundles create no partial database state.
- [ ] 3.5 Implement transactional raw-bundle storage and normalized ESLint/Semgrep JavaScript/TypeScript findings; verify supported fields are preserved and unsupported results appear in an explicit skipped summary.
- [ ] 3.6 Implement versioned stable fingerprints using SARIF partial fingerprints with the documented deterministic fallback; verify repeated and line-shift fixtures produce the expected fingerprint behavior.
- [ ] 3.7 Implement ingestion idempotency on project, revision, analyzer, and bundle digest; verify an identical retry returns the original run without duplicate findings or audit events.
- [ ] 3.8 Implement manual bundle upload and CI ingestion endpoints using the same validator; verify both routes produce equivalent normalized runs.
- [ ] 3.9 Build run-history and import-status screens showing revision, analyzer, counts, skipped results, state, and gate; verify Playwright covers successful, failed, and processing runs.

## 4. Deterministic Quality Gate and Finding Review

- [ ] 4.1 Implement the pure gate engine for severity thresholds, rule overrides, and dispositions; verify table-driven tests cover pass, fail, overridden rules, and accepted findings.
- [ ] 4.2 Implement immutable gate-policy versions with a default error-level threshold and owner-only updates; verify policy changes are attributed and deterministically recalculate affected runs.
- [ ] 4.3 Implement `unresolved`, `fix required`, and `accepted with reason` dispositions with required rationale and role checks; verify only acceptance removes a current finding from the unresolved blocking count.
- [ ] 4.4 Reconcile stable fingerprints across later runs so fixed findings disappear without permitting a manual false resolution; verify a later clean run resolves `fix required` while the original run remains historically reproducible.
- [ ] 4.5 Implement finding search and combined filters for severity, rule, path, support status, and review state; verify API pagination and combined-filter integration tests.
- [ ] 4.6 Build the dashboard, run gate summary, finding table, and finding-detail view with provenance and gate contribution; verify Playwright covers a 38-finding fixture and retains active filters.
- [ ] 4.7 Add an invariant test proving stored or newly generated Resolution Brief actions cannot change gate inputs or results.

## 5. Review Decisions and Audit History

- [ ] 5.1 Implement the per-project hash-chained audit-event writer and verifier; verify mutation, deletion, and reordering of fixture events are detected.
- [ ] 5.2 Integrate audit writes transactionally with ingestion, policy, credential, brief, disposition, authorization-denial, and gate-transition operations; verify a failed domain transaction leaves no orphan audit event.
- [ ] 5.3 Record reviewer disagreement when a human disposition differs from the latest brief while preserving human authority; verify disagreement aggregation tests.
- [ ] 5.4 Implement project and finding audit-history endpoints and views with actor, revision, transition, and correlation metadata; verify non-members cannot infer audit existence.
- [ ] 5.5 Add audit sanitization tests covering provider errors, tokens, excerpts, model payloads, and personal data; verify sensitive fixture values never appear in serialized events or logs.

## 6. Free-Provider Resolution Brief

- [ ] 6.1 Define the strict Resolution Brief JSON schema and domain types for action, confidence, cited explanation, nullable minimal diff, exactly two verification steps, and missing context; verify representative valid and invalid payloads.
- [ ] 6.2 Implement an AI provider interface and the Groq Free Plan adapter for allowlisted `openai/gpt-oss-20b` requests with tools disabled, provider storage disabled where supported, and no fallback; verify recorded HTTP contract tests.
- [ ] 6.3 Implement configurable request, token, concurrency, and daily safety limits below the free-plan limits plus rate-limit-header tracking; verify projected overages and HTTP 429 responses stop without retrying through another provider.
- [ ] 6.4 Build the versioned prompt from one supported finding, verified excerpt, rule metadata, and approved guidance only; verify snapshots contain no user identity, unrelated file, repository credential, or whole-repository content.
- [ ] 6.5 Implement provider-bound credential detection and typed redaction markers; verify a secret corpus is removed before mocked outbound submission and the plaintext never reaches logs.
- [ ] 6.6 Implement semantic validation for citation ranges, action-dependent diff presence, path/hash binding, excerpt-local patch applicability, whole-file rewrites, and approved verification commands; verify each invalid category is rejected.
- [ ] 6.7 Implement brief orchestration with eligibility checks, encrypted payload retention, prompt/model/provider versioning, one quota-safe transport retry, and no retry for schema or safety failures; verify failure-path integration tests.
- [ ] 6.8 Implement server-sent lifecycle events for queued, redacting, requesting, validating, complete, and unavailable states; verify no model token or partial brief is emitted before validation.
- [ ] 6.9 Build the finding-detail Resolution Brief panel with advisory labels, evidence links, stale-diff state, verification guidance, provider/version metadata, and independent reviewer controls; verify the brief alone never changes the disposition or gate in Playwright.
- [ ] 6.10 Add adversarial tests for instructions hidden in comments, filenames, rule text, guidance, and malformed provider responses; verify no test enables tools, leaks protected context, alters the schema, or bypasses semantic validation.

## 7. Privacy, Retention, and Project Deletion

- [ ] 7.1 Implement 30-day default expiry timestamps and a scheduled deletion job for raw excerpts and provider request-response payloads; verify expired plaintext is removed while minimized audit metadata remains.
- [ ] 7.2 Implement owner-confirmed project deletion that revokes credentials before cascading project data; verify another project's data remains unchanged and deleted credentials cannot ingest.
- [ ] 7.3 Add project controls to disable Resolution Brief independently of ingestion and gating; verify disabled projects make no provider request.
- [ ] 7.4 Document the external-provider data boundary and required free-account data-control settings; verify the release checklist fails when the settings have not been reviewed.

## 8. Evaluation and Release Guardrails

- [ ] 8.1 Define an immutable evaluation case format, labeling rubric, dataset manifest, and integrity hashes that separate development examples from held-out cases; verify the harness rejects overlap or modified held-out data.
- [ ] 8.2 Create and review development fixtures for prompt iteration, including supported actions, insufficient context, stale source, and prompt injection; verify every fixture has two-person label agreement or an adjudication record.
- [ ] 8.3 Create the first held-out batch covering critical authorization and injection flaws plus benign findings; verify labels, provenance, and expected evidence are complete without using the cases for prompt tuning.
- [ ] 8.4 Complete the held-out set to 40 cases with missing-context, adversarial-comment, patch-applicability, ESLint, and Semgrep coverage; verify the manifest count, category balance, and integrity hashes.
- [ ] 8.5 Implement the batched evaluation runner with quota-aware pauses and exact provider/model/prompt recording; verify it can resume without duplicating completed cases or exceeding configured daily limits.
- [ ] 8.6 Implement reports for macro action agreement, critical false accepts, missing-context recall, injection compliance failures, patch applicability, latency, token use, and disagreement; verify calculations against a hand-checked result fixture.
- [ ] 8.7 Implement release feature flags that enable full briefs only after mandatory thresholds pass and otherwise suppress suggested diffs; verify both pass and fallback profiles through integration tests.

## 9. Operations, Deployment, and Recovery

- [ ] 9.1 Add structured JSON logging with correlation IDs and explicit payload-denial filters; verify source excerpts, credentials, model payloads, and personal data are absent from captured logs.
- [ ] 9.2 Add health and metrics for database state, import failures, provider latency and availability, quota headroom, abstention, disagreement, and expected cost; verify operational endpoints expose no sensitive payloads.
- [ ] 9.3 Add production Compose configuration for an existing developer-controlled machine with persistent PostgreSQL volumes and documented secret injection; verify a clean machine can start the stack without paid infrastructure.
- [ ] 9.4 Add versioned database backup and restore scripts plus compatibility checks; verify a backup restores runs, dispositions, policies, and a valid audit chain into a clean environment.
- [ ] 9.5 Document and execute a rollback drill using the previous container images and compatible database state; verify ingestion, gate calculation, and audit inspection work after rollback.
- [ ] 9.6 Add a provider-policy release check confirming the account remains free, has no payment method or paid fallback, uses an allowlisted model, and has reviewed data controls; verify a failed check blocks brief enablement.

## 10. End-to-End Acceptance

- [ ] 10.1 Add an end-to-end flow from enriched CI bundle to stored run, filtered finding, failing gate, reviewer disposition, and updated deterministic gate; verify the complete flow without enabling AI.
- [ ] 10.2 Add an end-to-end Resolution Brief flow using a mocked provider, including lifecycle events, citations, advisory diff, verification steps, human disagreement, and audit history; verify all state transitions remain human-controlled.
- [ ] 10.3 Run the authorization, ingestion-fuzzing, path-traversal, secret-redaction, prompt-injection, and audit-integrity suites; verify all security checks pass before deployment.
- [ ] 10.4 Run load checks for configured maximum bundle size, a representative large run, concurrent reviewers, and free-provider concurrency caps; verify bounded resource use and deterministic rate-limit behavior.
- [ ] 10.5 Publish setup, CI integration, reviewer workflow, privacy boundary, zero-cost constraint, backup, rollback, and troubleshooting documentation; verify a developer unfamiliar with the project can complete the fixture-based walkthrough.
- [ ] 10.6 Execute the final held-out evaluation, deployment smoke test, zero-cost provider check, backup/restore drill, and rollback drill; verify every mandatory release criterion has linked evidence or the documented fallback is active.
