## Context

See `proposal.md` for motivation and scope. The repository currently contains planning artifacts but no application code or persisted product data. The product must support developer teams while keeping the first release limited to JavaScript/TypeScript findings from ESLint and Semgrep. It must ingest untrusted SARIF and source text, keep the quality gate independent of AI output, and make no billable third-party request.

The selected implementation stack is Next.js with TypeScript for the browser experience, Java 21 with Spring Boot for the authoritative API and background work, PostgreSQL for transactional and JSON storage, and Docker Compose for repeatable operation on existing developer-controlled infrastructure. Hosted inference uses a free-provider account and must fail closed before it can generate a charge.

## Goals / Non-Goals

**Goals:**

- Keep authorization, ingestion validation, gate calculation, AI safety checks, and audit writes behind one authoritative API boundary.
- Make every gate outcome reproducible from versioned policy, normalized findings, and human dispositions.
- Bind every source-dependent artifact to an immutable commit and file hash.
- Use hosted free inference without storing a payment method, selecting a paid model, or falling back to a billable route.
- Make provider, model, prompt, and policy versions explicit so evaluations are reproducible.
- Support containerized development, deployment, backup, restoration, and rollback without purchased infrastructure.

**Non-Goals:**

- Executing repository code, applying generated patches, or writing to source-control providers.
- Fetching repository contents through broad GitHub permissions; CI supplies the bounded evidence.
- Guaranteeing availability when free quotas or free models are unavailable.
- Supporting arbitrary SARIF producers, file types, repository-wide reasoning, or organization-scale enterprise administration in the first release.

## Decisions

### 1. Use a three-boundary architecture

The system is divided into a Next.js presentation layer, a Spring Boot application/API, and PostgreSQL. The browser never contacts the AI provider or database directly. Spring Boot owns authentication callbacks, authorization, ingestion, gate evaluation, brief orchestration, retention, and audit transactions.

```text
Browser / GitHub Actions
          |
          v
     Spring Boot API  ------> Hosted free AI provider
          |                         (briefs only)
          v
      PostgreSQL

Next.js renders the UI and calls only the Spring Boot API.
```

This keeps security decisions in one service and prevents duplicated authorization logic. A single Next.js full-stack application was considered, but Java 21 and Spring Boot are retained to match the selected stack and provide a clear API boundary. Independent microservices were rejected because their operational overhead does not fit the MVP.

The repository will use top-level `web`, `api`, `ci`, `eval`, and `infra` areas. Spring publishes an OpenAPI document, and the web client uses a generated TypeScript client so API contracts do not drift.

### 2. Use identity-only GitHub OAuth plus project-local roles

Interactive users authenticate with GitHub OAuth using only identity scopes; MergeGuard does not request repository-content access. On first sign-in, a local user record is created. Project owners manage `owner`, `reviewer`, and `member` memberships inside MergeGuard.

This matches a developer-facing product without introducing password reset or email delivery costs. Local password authentication was rejected because it adds credential recovery and storage responsibilities. GitHub App repository access was rejected because source context is supplied by CI and broad repository access would weaken data minimization.

Server sessions use secure, HTTP-only, same-site cookies with expiry and rotation. Development and automated tests use a disabled-by-default test identity adapter rather than bypassing authorization code paths.

### 3. Ingest an enriched, immutable analysis bundle

The CI helper packages:

- SARIF 2.1.0 output;
- repository identity and commit SHA;
- analyzer name and version;
- one bounded excerpt per finding, with repository-relative path, starting line, ending line, and file SHA-256;
- a deterministic bundle digest and source-run identifier.

The helper reads checked-out files but uploads only the relevant excerpt, capped at 30 surrounding lines. Manual upload uses the same bundle contract. Plain SARIF can still be imported for browsing and gating, but findings without verified excerpts cannot receive a patch-bearing brief.

The API validates content type, total size, nesting depth, counts, normalized paths, line ranges, supported tools, and digest before opening a transaction. It stores the raw accepted bundle in encrypted form and writes normalized run/finding records in the same transaction. Duplicate `(project, revision, analyzer, bundle digest)` uploads return the existing run.

Fetching files from GitHub at review time was rejected because it requires repository permissions, creates mutable evidence, and complicates manual uploads. Trusting snippets embedded in arbitrary SARIF without a file hash was rejected because a patch could target different content.

### 4. Keep normalized data transactional and raw evidence encrypted

Core records are `user`, `project`, `membership`, `ingestion_credential`, `analysis_run`, `finding`, `source_context`, `gate_policy_version`, `finding_disposition`, `resolution_brief`, `usage_counter`, and `audit_event`.

Raw SARIF, excerpts, and provider request/response payloads are encrypted at the application boundary with a deployment secret. Queryable metadata and normalized findings remain relational; analyzer-specific non-sensitive fields use JSONB. Ingestion tokens are stored only as keyed hashes, and AI-provider keys exist only as runtime secrets.

PostgreSQL row ownership is enforced in application authorization and reinforced with project identifiers, foreign keys, unique constraints, and integration tests. Database-native row-level security was considered but deferred because it would duplicate the service authorization model in the MVP.

### 5. Calculate gates synchronously from versioned inputs

Each project has an immutable sequence of gate-policy versions containing a severity threshold and rule overrides. Gate calculation is a pure operation over one run, one policy version, and the latest valid disposition for each finding. It runs after successful ingestion, policy changes, and disposition writes inside or immediately after the same transaction.

`accepted with reason` removes a finding from the unresolved blocking count. `fix required` remains blocking until the stable fingerprint disappears from a later run. Resolution Brief actions are not inputs to the calculator.

Stable fingerprints prefer SARIF partial fingerprints. When absent, MergeGuard derives a versioned fingerprint from analyzer, rule, normalized path, normalized region, and stable message fields. The fingerprint algorithm version is stored so later changes do not rewrite history.

Asynchronous gate calculation was rejected because the calculation is small and reviewers need read-after-write consistency. An AI-derived severity or gate was rejected because it would violate the deterministic enforcement boundary.

### 6. Use a strict, non-streaming provider call with streamed lifecycle events

The initial provider adapter targets the Groq Free Plan and allowlists only `openai/gpt-oss-20b`. The adapter uses the provider's OpenAI-compatible chat endpoint, requests strict JSON-schema output, disables tools, sets provider-side storage off where supported, and sends no more than approximately 6,000 input tokens with a 1,000-token output ceiling.

The server calls the provider non-streamingly so the response can use strict structured output. The browser receives server-sent lifecycle events such as `queued`, `redacting`, `requesting`, `validating`, `complete`, and `unavailable`; it never receives unvalidated model tokens. This preserves responsive feedback without exposing partial unsafe output.

The adapter interface remains replaceable, but deployment configuration contains exactly one explicit free-provider/model allowlist and no fallback chain. OpenRouter auto-routing and paid-provider fallback were rejected because either could change the executing model or billing behavior. Local inference was rejected because the user selected a hosted free provider.

Provider limits are configuration with a safety margin rather than permanent constants. The service consumes returned rate-limit headers, maintains conservative daily token/request counters, and refuses work before a projected limit breach. HTTP 429, provider removal, or incompatible plan changes produce a visible unavailable state.

### 7. Validate model input and output at multiple layers

Before a request, the service:

1. selects only the rule, exact finding, verified excerpt, and approved guidance;
2. scans for credential-like values and replaces them with typed redaction markers;
3. serializes untrusted text as data fields under fixed system policy;
4. records a prompt-template version and input digest without logging the plaintext payload.

After a response, the service validates:

- strict JSON shape and enumerated actions;
- required confidence and missing-context fields;
- citations that fall within supplied line ranges;
- one diff at most, only for `fix now`, matching the path and excerpt hash;
- a hunk that applies to the supplied excerpt and does not replace the whole file;
- exactly two verification steps;
- commands against project-approved non-mutating command patterns.

Any failure discards the candidate response. A single retry is allowed only for transient transport failure when the quota ledger confirms it is safe; schema or safety failures are not blindly retried. The provider receives no tools, repository credentials, user identity, unrelated files, or whole repository.

### 8. Make audit writes append-only and transactionally linked

Domain mutations write an audit event in the same database transaction. Events include actor or credential identity, project, event type, target identifiers, policy/model/prompt versions, before/after state hashes, timestamp, and correlation identifier. They exclude secrets and source/model plaintext.

Events form a per-project hash chain using the previous event hash, which makes accidental or unauthorized modification detectable. Corrections append superseding events rather than editing history. This is tamper-evident, not a claim of protection from a database administrator with full control.

### 9. Enforce retention and deletion with scheduled jobs

Encrypted raw source excerpts and provider request/response payloads receive a deletion timestamp 30 days after creation by default. A scheduled job removes expired payloads while retaining normalized findings, digests, versions, redaction indicators, and minimized audit events. Project deletion revokes credentials first, then removes project-owned records in a transaction or resumable deletion job.

Indefinite raw retention was rejected because source snippets can contain sensitive data. Immediate deletion was rejected because it would prevent short-term audit and evaluation of model behavior.

### 10. Separate prompt development from held-out evaluation

The `eval` area contains separately labeled development fixtures and 40 immutable held-out cases. Cases cover supported actions, critical security flaws, benign findings, missing context, stale revisions, malicious comments, and schema attacks. The harness records exact provider/model/prompt versions and reports macro action agreement, critical false accepts, missing-context recall, injection compliance failures, patch applicability, latency, token use, and reviewer disagreement.

The release profile enables suggested diffs only after all mandatory thresholds pass. If they fail, a feature flag suppresses diffs while preserving only explanation and verification behavior that passed its checks. Held-out cases are never used to tune the prompt.

### 11. Deploy on existing infrastructure with zero-cost checks

Docker Compose runs the web, API, and PostgreSQL services on an existing developer-controlled machine. A reverse proxy and purchased domain are not required for the baseline; environment-specific TLS or a pre-existing ingress may be added without changing the application design.

Startup validates that zero-cost mode is enabled, the provider/model is allowlisted, no fallback is configured, and required secrets are present. It cannot prove a provider's future commercial policy, so operations also require the account to remain on its free plan without a payment method. A provider-policy checklist is run before each release.

Structured JSON logs go to standard output. Spring health and metrics endpoints report component health, import failures, provider latency, quota headroom, abstention, disagreement, and expected cost while excluding payloads. Optional local Prometheus/Grafana containers may consume them without becoming runtime dependencies.

## Risks / Trade-offs

- **[Free provider limits or terms change]** -> Fail closed, display unavailability, retain a provider adapter, and require a deliberate review before allowlisting another free provider.
- **[A free provider processes proprietary source externally]** -> Send bounded redacted excerpts only, enable provider data controls, document the boundary, and let a project disable briefs entirely.
- **[Prompt injection changes model behavior]** -> Treat all repository text as data, disable tools, use strict output constraints, run semantic validation, and gate release on adversarial cases.
- **[Thirty lines are insufficient for a correct diagnosis]** -> Require `needs more context`, prohibit a diff, and record the missing evidence instead of silently expanding repository access.
- **[Suggested patch applies locally but is semantically wrong]** -> Keep patches advisory, revision-bound, minimal, and paired with verification; never apply or execute them.
- **[Two application stacks slow a small team]** -> Keep one Spring modular monolith, generate the TypeScript client from OpenAPI, and avoid microservices.
- **[GitHub identity is unavailable]** -> Existing sessions continue until expiry; new sign-in fails visibly without weakening authorization. A future identity adapter can replace OAuth without changing project roles.
- **[Hash-chain audit is mistaken for immutable external proof]** -> Describe it only as tamper-evident and include backup/restore checks in operational drills.
- **[Evaluation exhausts daily free tokens]** -> Run cases in controlled batches across reset windows and never upgrade automatically.

## Migration Plan

1. Establish the monorepo, local configuration, Docker Compose, database migrations, health checks, and zero-cost startup validation.
2. Add identity, projects, memberships, and ingestion credentials; verify cross-project isolation before accepting analysis data.
3. Add enriched-bundle tooling, transactional ingestion, normalized findings, run history, and deterministic gates using checked-in fixtures.
4. Add reviewer dispositions and append-only audit events; verify gate transitions without any AI integration.
5. Add the provider adapter behind a disabled feature flag, configure the verified free account/model, then add redaction, schema validation, quota enforcement, and lifecycle events.
6. Run development fixtures, freeze the prompt, execute the held-out evaluation, and enable only the advisory components that meet release thresholds.
7. Deploy the same containers to the selected existing machine, perform backup/restore and rollback drills, and record operational evidence.

Rollback restores the prior signed container images and the latest compatible PostgreSQL backup. Forward-only migrations require a tested compensating migration or restore procedure before release. Disabling Resolution Brief never disables ingestion, review, audit, or deterministic gating.
