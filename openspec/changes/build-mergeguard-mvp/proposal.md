## Why

Developers and software teams need a fast, consistent way to decide whether static-analysis findings should block a pull request without treating every warning equally or trusting an ungrounded chatbot. MergeGuard provides a deterministic quality gate and a narrowly scoped, advisory Resolution Brief so reviewers can make defensible decisions under delivery pressure while keeping all third-party service costs at zero.

## What Changes

- Add team sign-in, project membership, and revocable per-project ingestion credentials.
- Accept bounded SARIF 2.1.0 ingestion bundles from GitHub Actions and manual upload, preserve the raw run, normalize supported ESLint and Semgrep findings, and record immutable source provenance.
- Add run history, finding details and filters, plus a deterministic pass/fail gate based only on policy and recorded human dispositions.
- Add an advisory Resolution Brief for one JavaScript or TypeScript finding at a time, using bounded source excerpts and team-approved guidance.
- Use a hosted AI provider only while its account remains on a genuinely free plan. The application must never route to a paid model, create a billable request, or silently degrade to a paid fallback.
- Validate every model response against a fixed schema containing an action, confidence, evidence-linked explanation, minimal diff, verification steps, and explicit missing context.
- Add reviewer decisions, reasons, immutable audit events, rate limits, quota handling, prompt-injection defenses, secret redaction, and retention controls.
- Add a held-out evaluation harness, structured operational metrics, zero-cost deployment assets, rollback documentation, and fail-closed behavior when free quotas or dependencies are unavailable.
- Exclude repository execution, automatic commits, automatic dismissal, LLM-controlled gate decisions, repository-wide chat, multi-finding briefs, unsupported languages or analyzers, billing, and claims that the product proves security.

## Capabilities

### New Capabilities

- `team-project-access`: Team authentication, project membership, authorization boundaries, and ingestion credential lifecycle.
- `analysis-ingestion`: Validated SARIF ingestion, bounded source-context packaging, provenance, normalization, idempotency, and run history.
- `quality-gate`: Deterministic finding classification, project gate evaluation, filtering, and reviewer-controlled dispositions.
- `resolution-brief`: Safe generation and presentation of one structured, advisory Resolution Brief through a free hosted AI provider.
- `review-audit`: Reviewer decisions and append-only audit history linking people, findings, model output, source revisions, and gate transitions.
- `safe-operations`: Zero-cost enforcement, quotas, evaluation, privacy controls, observability, deployment, and rollback behavior.

### Modified Capabilities

None. This project has no existing product capability specifications.

## Impact

- Introduces a Next.js and TypeScript web client, a Java 21 Spring Boot API, PostgreSQL storage, and Docker-based local/deployment environments.
- Adds API contracts for authentication, projects, ingestion, findings, gate status, briefs, decisions, audit history, and operational health.
- Integrates GitHub Actions as an ingestion client and a hosted free AI provider behind a replaceable server-side adapter.
- Adds security-sensitive handling for source excerpts, SARIF payloads, project credentials, reviewer identity, and AI-provider keys.
- Establishes CI, evaluation fixtures, prompt-injection tests, structured logs, metrics, backup/restore, and rollback procedures.
