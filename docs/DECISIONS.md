# Architecture Decision Log

This file records decisions and open decision points. New technology or breaking
protocol decisions must be added here rather than retroactively presented as
requirements.

## Classification

- **CONFIRMED:** accepted decision.
- **ASSUMED:** reversible planning default awaiting evidence.
- **UNKNOWN:** open decision.
- **DEFERRED:** intentionally postponed.

## CONFIRMED Decisions

| ID | Decision | Rationale / consequence |
|---|---|---|
| ADR-001 | n8n is orchestration, Supabase is authoritative state, and Game/Web is interaction only. | Preserves loose coupling and replaceability. |
| ADR-002 | Coding occurs in external isolated Workers, not n8n. | Avoids mixing orchestration credentials/runtime with untrusted code execution. |
| ADR-003 | Codex is the primary MVP coding agent. | Product direction; invocation and model remain open. |
| ADR-004 | MVP supports GitHub only and one unique primary repository per Project; monorepo working scope is optional. | Narrows the initial integration and prevents ambiguous ownership. |
| ADR-005 | Owner workspaces are never modified; jobs use isolated clean workspaces/task branches. | Protects manual and uncommitted work. |
| ADR-006 | Workers return scoped structured results through trusted ingestion and do not directly write business tables. | Limits blast radius and centralizes transition validation. |
| ADR-007 | Long-running Developer/QA jobs use async submit, callback, and reconciliation. | Survives disconnects/restarts and avoids long-held requests. |
| ADR-008 | Commands are durable before n8n consumption; Realtime is not a queue. | Prevents lost Owner intent during n8n downtime. |
| ADR-009 | Current state, append-only events, and Realtime have separate responsibilities. | Supports correctness, audit, and responsive clients. |
| ADR-010 | Events are at-least-once with entity-local ordering and deduplication. | Avoids unjustified exactly-once/global ordering claims. |
| ADR-011 | Approvals bind immutable snapshots and expire; material change requires new approval. | Prevents authorization drift. |
| ADR-012 | Implementation, QA, approval, merge, and deployment are separate states. | Prevents false completion/deployment claims. |
| ADR-013 | Developer and QA are logically separate; QA independently verifies and does not edit production code in MVP. | Preserves review independence. |
| ADR-014 | Handoff and assignment are separate durable operations. | Makes authority explicit and routing recoverable. |
| ADR-015 | Agent, Run, Worker, availability, and Run phase are separate concepts. | Supports concurrency and accurate Game projection. |
| ADR-016 | One logical Agent/NPC may represent concurrent Runs with configurable capacity. | Avoids equating temporary containers with employees. |
| ADR-017 | Memory is typed, scoped, sourced, trusted, and lifecycle-managed; vector search is optional. | Prevents a generic vector table from becoming ungoverned truth. |
| ADR-018 | Originals are Artifacts; extracts/summaries are derived; large files use object storage. | Preserves evidence and keeps PostgreSQL appropriate. |
| ADR-019 | Supabase Cloud, existing-VPS n8n, same-VPS isolated Workers initially, and Vercel Web UI are the MVP deployment direction. | Balances cost and operation while retaining movable contracts. |
| ADR-020 | Supabase Auth email/password, invite-only, no public signup; high-risk actions require recent authentication. | Matches single-owner scope without eliminating future step-up/MFA. |
| ADR-021 | Separate scoped/rotatable service credentials; prefer GitHub App tokens; never expose service role to clients/Workers. | Enforces least privilege. |
| ADR-022 | Host Docker socket is not mounted into general Worker containers. | Socket access effectively undermines host isolation. |
| ADR-023 | The first MVP ends at a real QA-passed, Owner-reviewed, review-ready PR. | Proves delivery without granting merge/deploy authority. |
| ADR-024 | Protocols use confirmed semantic contracts plus explicitly provisional wire shapes. | Avoids inventing implementation while protecting boundaries. |
| ADR-025 | The Agency operates without the Game; Game real-work visuals are event-grounded and decorative behavior cannot fabricate work. | Keeps the Game replaceable and honest. |

## ASSUMED Decisions / Defaults

| ID | Default | Validation needed |
|---|---|---|
| A-001 | Initial single n8n instance. | Verify load/recovery; remain queue-mode-ready. |
| A-002 | Approximately 5–15 projects, 5–20 daily tasks, 1–3 concurrent jobs. | Measure real usage. |
| A-003 | 14-day n8n history, 30-day verbose logs, seven-day temporary workspaces. | Validate storage, privacy, and incident needs. |
| A-004 | Approximate retry backoff of 5s, 30s, and 2m. | Tune to APIs and failure modes. |

## UNKNOWN — Decisions Required Before Relevant Implementation

| ID | Decision | Evaluation criteria / timing |
|---|---|---|
| ODR-001 | Command API technology. | Security, durable intake, auth, transactions, cost; before Phase 1/3 boundary implementation. |
| ODR-002 | Worker language/framework and job transport. | Isolation, cancellation, recovery, operations; before Phase 2. |
| ODR-003 | Exact safe nested/container build mechanism. | Must not expose host socket; before executing untrusted builds. |
| ODR-004 | Object storage and bucket/access design. | File sizes, signed access, retention, isolation; before Artifact implementation. |
| ODR-005 | Secret manager/credential broker. | Scoped ephemeral grants, rotation, VPS constraints; before real Worker credentials. |
| ODR-006 | Codex invocation/API and exact model/version. | Current supported interface, cost, capability, telemetry; before Worker implementation. |
| ODR-007 | PM and QA model/provider. | quality, independence, cost, structured output; before agent runtime. |
| ODR-008 | Database transition/RPC/outbox design. | Atomic state/event persistence, RLS, concurrency; during Phase 1. |
| ODR-009 | Callback transport/signature and error envelope. | Replay protection, expiry, reconciliation; during Phase 2. |
| ODR-010 | Central logs/metrics platform. | current VPS, retention, alerts, cost; before operational hardening. |
| ODR-011 | Telegram integration. | secure identity, actionable alerts, rate limits; after core loop. |
| ODR-012 | Budget thresholds and legal/data retention obligations. | Owner/client policy; before enforcement commitments. |
| ODR-013 | Future Game technology. | Must follow stable contracts; not before Phase 4. |

## DEFERRED Decisions

- Multi-user roles and client access.
- Multi-agency SaaS architecture.
- Multi-repository Projects and additional Git providers.
- Mandatory MFA/passkeys/OAuth.
- Full Staging and enterprise HA.
- Broad vector search/RAG.
- Automatic merge/production deployment.
- Advanced Agent roles and Game implementation details.

## CONFIRMED Change Procedure

For a new or changed decision: describe evidence and alternatives, classify it,
record security/compatibility/migration consequences, update affected documents,
and version any breaking protocol. Never change a confirmed semantic contract only
because one implementation is more convenient.
