# Architecture

## Classification and Current Phase

This is the Phase 0 architecture baseline. **CONFIRMED** rules are binding;
**ASSUMED** items are provisional defaults; **UNKNOWN** items require later
decisions; **DEFERRED** items are outside the current phase. No implementation is
described as existing.

## CONFIRMED — Architectural Invariants

```text
n8n       = Agent Runtime / Orchestrator
Supabase  = State / Memory / Source of Truth
Game/Web  = Visual / Interaction Layer
Worker    = Isolated execution environment for Codex and repository tools
```

- The Game/Web client is replaceable without redesigning n8n or Supabase.
- n8n workflows are replaceable without redesigning the clients.
- Stable Supabase-backed data and versioned domain protocols connect components.
- n8n execution history, Realtime, worker filesystem, and UI cache are not sources
  of truth.
- External systems are authoritative for their own facts; GitHub state is verified.

## CONFIRMED — Logical Topology

```text
Owner Web UI / Future Game
  │ Supabase Auth + versioned command
  ▼
Stable Command API (technology UNKNOWN)
  │ persist durable command; return command_id
  ▼
Supabase authoritative state/events ◄──────────────┐
  │ durable inbox/outbox                           │ validated transitions/results
  ▼                                                │
n8n orchestration ── async scoped job ──► Worker service
  │                                      │ isolated workspace
  │                                      ▼
  │                                 Codex + repository tools
  │                                      │ branch/commit/PR
  │                                      ▼
  └──────── callbacks/polling ──────── GitHub
                      ▲                  │ signed webhook
                      └── stable authenticated ingress

Supabase current state + events ── Realtime ──► Web UI / Future Game
```

The diagram is logical, not a commitment to HTTP paths, queue products, or host
layout.

## CONFIRMED — Domain Model

- **Workspace:** Agency boundary; one exists in the MVP.
- **Client:** optional customer boundary beneath Workspace.
- **Project:** long-lived work unit with one primary GitHub repository and optional
  monorepo working scope. A repository cannot be the primary repository of multiple
  MVP Projects.
- **Task:** Owner-requested meaningful outcome with goal, scope, acceptance criteria,
  priority, lifecycle, and approved plan.
- **Subtask:** executable unit created within an approved Task plan; may be omitted
  for a very small Task.
- **Logical Agent:** long-lived role identity/profile and future NPC.
- **Worker:** temporary isolated process/container that executes a job.
- **Agent Run:** one execution attempt by a logical Agent. Retry creates another Run.
- **Review:** independent QA assessment with its own history.
- **Approval:** human decision bound to an immutable action snapshot.
- **Delivery:** merge/release/deployment concern, separate from Task completion.
- **Handoff:** durable transfer of context/request; not assignment.
- **Artifact:** persistent output or reference; chat alone is not an Artifact.
- **Memory:** reusable scoped fact/decision with provenance, trust, and lifecycle.
- **Command:** request for an action.
- **Event:** append-only fact that an accepted state change/action occurred.
- **Operation:** idempotently tracked external or retriable side effect.

## CONFIRMED — State Separation

Provisional enum spellings are listed in `PROTOCOLS.md`; semantic separation is
confirmed:

- Task state represents business/work outcome.
- Run state represents one execution attempt.
- Run phase represents activity such as planning, coding, testing, or reviewing.
- Review state represents QA outcome.
- Approval state represents human decision.
- Delivery state represents merge/release/deployment.
- Agent availability represents ability to accept work, not current activity.

The Agent availability projection considers configured logical capacity and
physical worker capacity. One logical Agent may have several Runs. The Game derives
visual activity and attention indicators from Runs/events rather than treating one
Agent status as the complete truth.

## CONFIRMED — n8n Responsibilities

n8n owns coordination and permitted business-state transitions: command routing,
PM planning, task decomposition, assignment, handoffs, approval coordination,
Developer/QA dispatch, retries, cancellation, escalation, reconciliation, and
milestone/notification production.

Initial logical workflow boundaries are Command Intake, PM Orchestration, Developer
Dispatch, QA Dispatch, Approval, GitHub Reconciliation, Recovery/Scheduler, and
Notification. They are responsibilities, not fixed implementation files.

n8n shall not edit source code, be a persistent development workspace, fabricate
GitHub facts, keep the sole durable state, or hold synchronous connections for
long-running Codex jobs. Initial single-instance n8n is acceptable, but contracts
must be queue-mode-ready and recovery cannot depend on process memory.

## CONFIRMED — Worker and Git Architecture

- Each job receives run-scoped context, permissions, credentials, expiry, timeout,
  and resource limits.
- Each development run uses a clean isolated checkout/worktree and task branch.
- Task branches follow the suggested pattern
  `agent/<project-code>/<task-id>-<short-name>`; exact validation is provisional.
- A dirty Owner workspace is never reset, cleaned, or overwritten.
- The Worker returns structured results to trusted ingestion/n8n and receives no
  unrestricted Supabase service role or host access.
- Callbacks are authenticated and replay-resistant. Polling/reconciliation handles
  lost callbacks.
- GitHub Actions may perform CI but are not the primary implementation worker.
- Parallel repositories are allowed. Same-repository tasks require dependency and
  conflict analysis; file paths alone are not proof of independence. Uncertainty
  favors serialization.

## CONFIRMED — Persistence and Consistency

Supabase stores durable command, domain, operation, memory, artifact, usage, and
event records. Business transitions should atomically validate the expected entity
version, update current state, append an event, and create any durable outbox work.
The exact database function/API mechanism is UNKNOWN.

Event delivery is at-least-once. Entity-local sequence/version supports ordering;
global total ordering is unnecessary. Consumers reload authoritative state and
missed events after disconnect.

## CONFIRMED — Memory Architecture

Workspace, Client, Project, Task, Decision, Episodic, and repository-derived memory
are separate typed/scoped records. Provenance and scope are always available;
trust classes include Owner-confirmed, System-confirmed, Source-derived,
Agent-inferred, and Unverified. Mandatory procedures live in version-controlled
documentation or explicit procedures, not only semantic retrieval.

Current execution context is temporary. Promotion to durable memory is explicit.
Repository knowledge is tied to a revision and is potentially stale when HEAD
changes. Originals remain Artifacts; extractions/summaries are derived artifacts;
only selected validated facts become Memory. Secrets never become memory content.

## CONFIRMED — Deployment

- Supabase Cloud provides the MVP database, Supabase Auth, and Supabase Realtime
  platform capabilities.
- n8n is self-hosted on the existing VPS.
- Coding Workers may share that VPS initially but remain runtime-isolated and
  independently terminable/movable.
- The minimal Web UI deploys to Vercel.
- Development and Production are distinct; permanent Staging is not required.
- Durable object storage holds large files; temporary worker storage is not backup.
- Important n8n definitions/configuration are reproducible/version-controlled.

Same-host containers reduce cost but do not create a strong security boundary.
Resource exhaustion and host compromise remain risks requiring constrained workers,
monitoring, and a later option to move workers to separate hosts.

## ASSUMED

- Initial n8n runs as one instance because expected load is low.
- Capacity figures in `PRODUCT.md` guide initial sizing only.
- Configurable retention defaults are 14 days for n8n history, 30 days for verbose
  logs, and seven days for temporary failed/cancelled workspaces.

## UNKNOWN

- Command API, Worker language/framework, job transport/queue, callback transport,
  exact isolation/build mechanism, object-storage design, secret broker, Codex
  invocation/model, PM/QA providers, Telegram integration, and centralized
  logs/metrics.
- Exact transaction/RPC design, schema/table layout, endpoint paths, and signature
  formats.
- How same-host untrusted builds will receive safe nested container capability.

## DEFERRED

- n8n queue mode from day one, separate worker hosts, full Staging, self-hosted
  Supabase, multi-repository Projects, other Git providers, broad vector retrieval,
  Game technology/implementation, and advanced agent roles.
