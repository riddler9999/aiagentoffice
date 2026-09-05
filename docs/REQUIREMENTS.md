# System Requirements

## Classification

Every requirement is classified as **CONFIRMED**, **ASSUMED**, **UNKNOWN**, or
**DEFERRED**. Identifiers are stable references for discussion, not implemented
test IDs.

## CONFIRMED — Functional Requirements

### Identity and project context

- **FR-001:** The MVP shall support one pre-provisioned/invite-only Owner using
  Supabase Auth email/password; public signup shall be disabled.
- **FR-002:** A Workspace shall contain Clients and Projects; Projects may omit a
  Client for internal work.
- **FR-003:** A Project shall have one unique primary GitHub repository, goals,
  context, policies, credential references, and an optional monorepo working scope.
- **FR-004:** First acceptance may bootstrap Project configuration through a
  controlled admin process; configuration must remain persistent and not hard-coded.

### Commands, planning, and approvals

- **FR-010:** The authenticated command boundary shall durably accept commands and
  return a command identity even while n8n is unavailable.
- **FR-011:** Commands shall be schema-versioned, scoped, authenticated, authorized,
  idempotent, and traceable.
- **FR-012:** The PM shall retrieve only relevant Workspace/Client/Project context,
  ask for missing material information, and avoid silently inventing requirements.
- **FR-013:** Meaningful implementation shall not start before the Owner approves an
  exact immutable plan snapshot.
- **FR-014:** Material scope, architecture, security, cost, timing, or production
  side-effect changes shall enter `SCOPE_CHANGE_REQUIRED` behavior and require a
  new approval.
- **FR-015:** Approvals shall bind action, scope, target, risk, artifact/version/hash,
  actor, time, and expiry. Material mutation invalidates prior approval.
- **FR-016:** High-risk approvals shall require recent re-authentication.

### Work execution and GitHub delivery

- **FR-020:** n8n shall coordinate asynchronous Developer and QA jobs; it shall not
  execute/edit project code or hold an HTTP request open for a long job.
- **FR-021:** Each coding job shall use a controlled isolated clean workspace and
  task branch; it shall not modify an Owner dirty working directory.
- **FR-022:** Codex shall be the primary MVP coding agent, without prescribing its
  invocation mechanism or model/version.
- **FR-023:** A Developer run shall return a structured result including identity,
  outcome, repository context, changes/artifacts, checks, issues, and next action.
- **FR-024:** Meaningful tasks shall use attributable branches and commits and open
  a real review-ready GitHub PR. Direct protected-branch push, force push, merge,
  and production deployment are prohibited without separate authority.
- **FR-025:** GitHub shall remain authoritative for branch, commit, PR, merge, and CI
  facts; authenticated ingestion/reconciliation shall verify them.
- **FR-026:** QA shall independently inspect evidence and rerun checks where
  technically possible rather than trust Developer claims.
- **FR-027:** QA failure shall create structured correction feedback and a new
  Developer run; at most three QA correction cycles are allowed before escalation.

### State, recovery, and controls

- **FR-030:** Supabase shall persist current state, commands, tasks, subtasks, runs,
  reviews, approvals, deliveries, handoffs, operations, memories, artifacts,
  messages, notifications, usage, and canonical events as needed by the domain.
- **FR-031:** Agent Run attempts shall retain immutable identity/input lineage and
  terminal evidence; retry creates a new run linked to its predecessor.
- **FR-032:** Canonical events shall be append-only. Corrections shall append new
  evidence rather than silently rewriting history.
- **FR-033:** Owner controls shall include pause, resume, cancel, retry, approve, and
  reject. Reassign is optional for the first acceptance UI.
- **FR-034:** Cancellation shall default to graceful stop and persisted state;
  explicit force termination shall exist for emergencies.
- **FR-035:** Lost callbacks shall be reconciled by actual job/external state and
  shall not duplicate side effects.
- **FR-036:** Durable state shall survive UI disconnect, worker failure, n8n restart,
  and VPS restart; retry/recovery must not rely on an open process connection.
- **FR-037:** Capacity overflow and pending handoffs shall remain durably queued.
- **FR-038:** Temporary failed/cancelled workspaces shall default to seven-day
  retention and allow incident pinning; cleanup shall be observable.

### Memory, artifacts, and realtime

- **FR-040:** Memory shall distinguish Workspace, Client, Project, Task, Decision,
  Episodic, and repository-derived types with scope, provenance, trust, and lifecycle.
- **FR-041:** Agent run context shall not automatically become durable memory;
  critical Agent-generated claims require authoritative evidence or validation.
- **FR-042:** Repository-derived knowledge shall reference branch/commit and be
  considered potentially stale after repository changes.
- **FR-043:** Original uploaded material shall remain an Artifact; extraction,
  transcription, summary, and promoted memory shall be separate derived records.
- **FR-044:** Large artifacts shall live in durable object storage, not PostgreSQL;
  PostgreSQL stores metadata and references.
- **FR-045:** Realtime shall notify clients but shall not be the durable queue or
  source of truth. Reconnecting clients shall fetch current state and missed events.

### Owner interface and operations

- **FR-050:** The Web UI shall show task, subtask, run, review, approval, and relevant
  delivery status plus implementation/QA reports and GitHub references.
- **FR-051:** The dashboard shall show active/queued/blocked/failed/stale work,
  retries, durations, capacity, pending approvals, health, critical errors, and
  available usage/cost estimates.
- **FR-052:** Web notifications are required. Telegram critical notifications are
  an immediate follow-up and cannot be a core-system dependency.
- **FR-053:** The system shall record model/provider, run/project, available token or
  usage data, and estimated cost where technically available.

## CONFIRMED — Non-Functional Requirements

- **NFR-001 Security:** Least privilege, RLS, scoped service identities, credential
  rotation, input validation, secret minimization, and auditability are mandatory.
- **NFR-002 Isolation:** Worker jobs shall have controlled filesystems, credentials,
  timeout, resources, and cleanup and shall not receive the host Docker socket.
- **NFR-003 Reliability:** Business-hours reliable internal operation shall recover
  safely from expected restarts, temporary network failure, and missed callbacks.
- **NFR-004 Replaceability:** Game/Web, n8n workflows, and Workers shall depend on
  stable domain protocols rather than each other's implementation details.
- **NFR-005 Consistency:** State transitions and their canonical event/outbox effects
  shall avoid partial, contradictory persistence; exact transaction mechanism is
  unresolved.
- **NFR-006 Delivery:** Events may be at-least-once; consumers shall deduplicate by
  stable identity. Global total ordering is not required; entity-local ordering is.
- **NFR-007 Privacy:** Raw production/client-sensitive data shall not be used for
  tests by default. Logging shall minimize data and redact common secrets where
  practical without claiming perfect detection.
- **NFR-008 Maintainability:** Protocols and workflows shall be versioned; important
  workflow definitions/configuration shall be reproducible and version-controlled.
- **NFR-009 Observability:** Human, Agent, Service, and External System actors and
  requested/authorized/executed chains shall remain distinguishable.
- **NFR-010 Accessibility/UX:** Offline or stale client state shall be explicit; a
  visual animation shall never be the only representation of real work status.

## ASSUMED

- Initial load and job-duration assumptions are listed in `PRODUCT.md` and must not
  be implemented as hard limits.
- n8n execution history defaults to 14 days, verbose worker/n8n logs to 30 days,
  and temporary workspaces to seven days; all are configurable operational defaults.

## UNKNOWN

- Concrete API, queue, callback, database RPC/transaction, object-storage, secret
  broker, metrics/logging, and worker technologies.
- Exact budget thresholds, service-level objectives, and legal retention duties.
- Exact field nullability and enum encodings pending schema validation.

## DEFERRED

- Multi-user/client login, multi-agency SaaS, multiple repositories per Project,
  non-GitHub Git, automatic merge/deploy, mandatory MFA, full staging, semantic
  retrieval at scale, advanced analytics, email notifications, and Game code.
