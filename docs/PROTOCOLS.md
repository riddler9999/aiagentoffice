# Domain Protocols

## Contract Status

This document is the hard semantic contract among Supabase, n8n, Workers, Web UI,
the future Game, and external ingress. **CONFIRMED** sections are normative.
**ASSUMED / PROVISIONAL** wire examples are not implemented API claims. **UNKNOWN**
details must be decided through change control. **DEFERRED** protocols are out of
scope.

Normative words `MUST`, `MUST NOT`, `SHOULD`, and `MAY` express requirement levels.

## CONFIRMED — Authority Contract

- UI/Game MUST submit intent as authenticated Commands and MUST NOT insert canonical
  Events or arbitrarily change operational state.
- n8n MUST coordinate authorized transitions and persist accepted results/state.
- Workers MUST execute only scoped Jobs and MUST NOT hold unrestricted business
  table or host access.
- Supabase MUST hold authoritative durable state and canonical history.
- Realtime MUST be treated as notification transport, not state or durable queue.
- GitHub MUST remain authoritative for GitHub-owned facts; ingress MUST verify and
  reconcile them.
- n8n internal history, UI cache, and worker storage MUST NOT override Supabase.

## CONFIRMED — Universal Envelope Semantics

Commands, Events, Handoffs, Jobs, Results, Callbacks, Approvals, and Operations MUST
support:

- stable unique identity;
- schema/protocol version;
- Workspace plus applicable Client/Project/Task/Subtask/Run scope;
- actor or service identity;
- creation/occurrence time;
- correlation identity for one business flow;
- causation identity where one record caused another;
- validated structured payload;
- idempotency identity where retriable processing or side effects exist.

Exact names, nullability, formats, and encodings remain provisional.

## CONFIRMED — Command Protocol

A Command requests an action; it does not assert success. The stable Command API
MUST authenticate the Owner, authorize the scope/action, validate the schema,
deduplicate by idempotency identity, persist the Command before dispatch, and return
an accepted `command_id` without requiring n8n to be online.

Conceptual lifecycle: `received → queued → processing → completed|failed|cancelled`.
These spellings are provisional. Retrying intake MUST return/reconcile the same
logical command rather than duplicate an action.

Provisional example:

```json
{
  "command_id": "CMD-...",
  "schema_version": "1",
  "command_type": "task.submit",
  "workspace_id": "...",
  "project_id": "...",
  "actor": { "type": "HUMAN", "id": "..." },
  "correlation_id": "...",
  "idempotency_key": "...",
  "payload": { "message": "Implement TASK-001" },
  "created_at": "..."
}
```

## CONFIRMED — Canonical Event Protocol

An Event states that a validated fact occurred. Canonical Events MUST be append-only,
schema-versioned, safely scoped, attributable, and deduplicable. Corrections MUST
append subsequent events. Events SHOULD contain safe metadata and protected
artifact/log references, not raw prompts, huge files, secrets, full environments,
or verbose stack traces.

Required semantic fields include event identity/type/version/time; Workspace and
applicable scope identifiers; actor identity; correlation/causation; and payload.
Entity-local sequence/version MUST be used when exact ordering matters. Global
ordering is not required. Delivery MAY be at-least-once; consumers MUST deduplicate.

Stable namespaced types include the families:

```text
project.created, project.updated
task.created, task.status_changed, task.completed, task.cancelled
subtask.created, subtask.assigned
run.created, run.started, run.completed, run.failed, run.cancelled
handoff.created
review.started, review.passed, review.failed
approval.requested, approval.approved, approval.rejected
delivery.requested, delivery.deployed, delivery.failed
agent.status_changed, agent.message_created
operation.started, operation.succeeded, operation.failed
github.pull_request_opened, github.pull_request_merged, github.check_updated
artifact.created
```

Names and payload fields are provisional until schema validation; namespace
semantics and deliberate versioning are confirmed.

## CONFIRMED — Handoff and Messaging Protocol

A Handoff transfers work/context; it does not assign authority. Assignment and any
state transition MUST be separate validated operations. Meaningful Handoffs MUST be
durable, scoped, attributable, recoverable, and machine-readable when action is
expected. Large context MUST be passed by references.

Provisional envelope fields:

```text
handoff_id, schema_version, workspace_id, client_id?, project_id, task_id,
subtask_id?, source_agent_id, source_run_id, target_role_or_agent_id,
handoff_type, objective, context_references, artifact_references, constraints,
expected_output, requested_action, created_at, correlation_id, causation_id?, status
```

Conceptual status: `pending|accepted|processed|rejected|cancelled`.

Developer-to-QA and QA-to-Developer correction Handoffs MAY route automatically
inside approved scope and limits. Developer business clarification normally routes
through PM. A pending Handoff MUST be recoverable without duplicate Runs.
Human-readable summaries MAY accompany structured fields. Hidden chain-of-thought
MUST NOT be required or persisted; plans, evidence, decisions, concise rationale,
errors, and recommendations SHOULD be persisted instead.

## CONFIRMED — Async Worker Job Protocol

n8n MUST submit a scoped asynchronous Job and receive acceptance plus a job
identity. An open HTTP request MUST NOT be required for job lifetime. A Job MUST bind
job/run/project identity, input/context references, allowed operations, credential
scope, timeout, expiry, version metadata, and callback/reconciliation identity.

Worker acceptance is not completion. Completion/failure normally arrives through
an authenticated callback; polling/reconciliation MUST recover loss. A callback
MUST authenticate independently of knowing `job_id`/`run_id`, validate payload and
scope, enforce expiry/replay protection, and be idempotent.

Provisional result example:

```json
{
  "schema_version": "1",
  "job_id": "JOB-...",
  "run_id": "RUN-...",
  "project_id": "...",
  "task_id": "...",
  "status": "implementation_complete",
  "repository": { "branch": "agent/project/TASK-001-change", "commits": ["..."] },
  "changed_files": ["..."],
  "checks": [{ "command": "...", "status": "passed" }],
  "artifacts": ["ART-..."],
  "known_issues": [],
  "next_action": "qa_review"
}
```

The minimum semantics—identity, outcome, repository context where applicable,
changes/artifacts, verification, issues/errors, and next lifecycle action—are
normative. Exact JSON is provisional.

## CONFIRMED — Run and Review Protocol

Every execution/retry MUST create a distinct Agent Run linked to the preceding
attempt. Run identity, input snapshot, assignment, attempt lineage, and terminal
evidence MUST NOT be silently rewritten. Lifecycle updates are allowed while the
Run is active and MUST produce events. Administrative correction uses superseding
records/events.

QA MUST receive independently assembled approved requirements, acceptance criteria,
diff/repository references, and constraints. A Developer's success claim alone is
insufficient. QA result MUST be PASS, FAIL, or BLOCKED with evidence. FAIL MUST
create structured correction feedback. A maximum of three correction cycles is
allowed before PM/Owner escalation.

## CONFIRMED — Approval Protocol

An Approval MUST bind exact action type, scope, target environment, risk, immutable
artifact/version/hash, requesting actor/time, expiry, approving actor/time, and
status. A broad mutable `approved=true` is invalid. Material changes invalidate the
Approval for the changed object. High-risk Approval MUST require recent
authentication. Approval to implement MUST NOT imply merge or deployment authority.

## CONFIRMED — Idempotency and Reconciliation

Each retriable external side effect MUST have a stable Operation identity derived
from the scoped logical intent (for example project/task/run/type/target). This
includes PRs, GitHub comments, branches, deployment requests, messages, database
mutations, and paid external operations.

“No response” MUST NOT mean “did not happen.” Recovery MUST inspect the Operation
record and external source, reuse/reconcile an existing result, and execute only
when absence is established. After a lost worker response, recovery checks the
remote branch, commits, PR, job state, and recorded Operations before continuing.

## CONFIRMED — Retry, Timeout, and Cancellation

- Transient infrastructure failures: maximum three attempts.
- Tool/API failures: maximum three attempts only when safe/idempotent.
- Agent crash/timeout: initial attempt plus at most two additional attempts.
- Backoff is configurable and exponential; approximately 5s, 30s, and 2m is an
  initial example, not a fixed wire contract.
- Timeout is configurable by run type and may receive a permitted override.
- Graceful cancellation stops new operations, completes/aborts the current safe
  atomic operation, persists evidence, stops the worker, and marks the Run.
- Force termination exists for security/runaway emergencies and is auditable.

## CONFIRMED — State Families (Provisional Enum Spellings)

Semantic separation is normative; exact strings may be refined before schema lock.

```text
Task:     draft, planning, awaiting_approval, ready, in_progress, blocked,
          completed, cancelled
Run:      queued, starting, running, waiting_for_input, waiting_for_approval,
          cancelling, succeeded, failed, blocked, cancelled, timed_out
Run phase: planning, thinking, coding, testing, reviewing, waiting_on_tool,
           preparing_handoff
Review:   pending, in_review, passed, failed, blocked
Approval: pending, approved, rejected, expired, cancelled
Delivery: not_requested, pending_approval, queued, deploying, deployed, failed,
          rolled_back
Agent availability: offline, available, at_capacity, paused, disabled, degraded
```

Task completion MUST NOT imply merge/deployment. Agent availability MUST NOT encode
Run activity. Capacity overflow MUST remain durably queued. Disabling blocks new
assignments but, by default, allows active Runs to finish; graceful cancellation or
force termination requires an explicit action.

## CONFIRMED — Client Realtime and Game Protocol

Clients MUST fetch current state and missed events on start/reconnect, deduplicate,
then resume Realtime. Stale cached state MUST be visibly marked. The Game SHOULD
snap to current projected state and surface missed important milestones rather than
replay every animation.

The Game MAY derive visual state from availability, active/queued counts, a selected
primary Run/activity, attention count, latest important event, and synchronization
time. A real-work animation MUST be grounded in canonical state/events. Decorative
behavior MAY exist but MUST NOT imply work that did not occur. Direct NPC
conversation MUST still pass through command, task, approval, and permission rules.

## ASSUMED / PROVISIONAL

- All JSON, field names, enum spellings, identifier formats, and endpoint concepts
  in this document are initial shapes pending Phase 1/2 validation.
- A future initial protocol version may be called `1`; no deployed version exists.

## UNKNOWN

- URLs, HTTP topology, required/optional field matrix, token/signature formats,
  queue and callback transports, transaction/RPC mechanism, artifact upload, secret
  brokerage, and precise error envelope.

## DEFERRED

- Multi-user/client command authority, non-GitHub provider events, production
  deployment automation, and Game-technology-specific transport bindings.

## CONFIRMED — Change Control

Confirmed semantics MUST NOT change silently. A necessary change requires rationale
and decision record in `DECISIONS.md`, synchronized documentation, a version bump
for breaking changes, migration/compatibility handling, and deliberate consumer
updates. Provisional shapes MUST be validated during Phase 1/2 before promotion to
CONFIRMED.
