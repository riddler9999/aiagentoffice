# Security Model

## Classification

**CONFIRMED** rules are mandatory. **ASSUMED** defaults require validation.
**UNKNOWN** items are unresolved implementations or policies. **DEFERRED** items are
not present in MVP but must not be made impossible.

## CONFIRMED — Trust Boundaries

```text
Human client/browser/Game (untrusted client)
  → authenticated Command API
  → trusted orchestration (n8n)
  → scoped Worker job (potentially hostile repository/input)
  → external systems (GitHub, APIs)
  → authenticated ingress/reconciliation
  → Supabase authoritative state
```

Repository contents, uploaded files, web content, prompts, worker output, callbacks,
and external webhooks are untrusted until validated. Same-VPS deployment does not
make Worker code trusted or provide a security boundary.

## CONFIRMED — Human Authentication and Authorization

- Supabase Auth provides Owner identity using email/password.
- The Owner account is pre-provisioned/invite-only and public signup is disabled.
- UI/Game uses the same human identity model; neither contains privileged service
  credentials.
- RLS is enabled from MVP onward even with one Owner.
- Core mutations pass through the authenticated command boundary rather than
  arbitrary client table updates.
- High-risk approvals require recent re-authentication. MFA is not initially
  mandatory, but architecture must support step-up/MFA later.
- An approval authorizes only its exact unmodified action snapshot, scope, target,
  and validity window.

## CONFIRMED — Tenant and Context Isolation

- All applicable records are scoped to Workspace, optional Client, Project, Task,
  and Run.
- A run for Project A must not retrieve Project/Client B data without an explicit
  authorized reason.
- Internal Projects may omit Client but remain Workspace-scoped.
- RLS policies and trusted backend validation must enforce, not merely document,
  these boundaries.
- Full multi-agency SaaS is not implemented; the Workspace boundary is retained so
  future change does not require a flat data redesign.

## CONFIRMED — Service Identities and Credentials

- Every machine/service connection uses a separate scoped identity. No universal
  shared secret may authenticate all services.
- Credentials are independently rotatable/revocable and expire where practical.
- The Command API validates the Owner and sends n8n trusted scoped context; it does
  not unnecessarily forward raw Owner JWTs.
- Worker job/callback credentials bind job, run, project, allowed scope, and expiry.
  Knowing an identifier is not authentication.
- n8n may use protected privileged Supabase access, but important mutations remain
  auditable. Supabase `service_role` never reaches Web UI, Game, or Worker.
- Prefer scoped GitHub App installation tokens to broad long-lived PATs. Worker
  repository grants cover only the relevant repository and operations.
- Secret values never enter Git, memory, events, prompts, artifacts, or documentation.
  Project data stores only credential references and usage scope.

## CONFIRMED — Ingress and Protocol Security

- GitHub webhook signatures are cryptographically verified before processing.
- Commands, callbacks, and webhooks are schema-validated, authenticated,
  authorized, scoped, rate-limited where appropriate, deduplicated, correlated,
  and audited.
- Callback credentials/signatures include expiry and replay protection.
- External facts are reconciled against the owning system when ambiguity exists.
- Every retriable side effect uses an idempotency/Operation identity. A timeout does
  not prove failure.
- Breaking protocol changes require versioning and compatibility handling.

## CONFIRMED — Worker Isolation and Tool Security

- Jobs use controlled isolated filesystems/workspaces, explicit environment,
  resource limits, timeouts, termination, and cleanup.
- Workers never modify an Owner's dirty working tree or receive unrestricted host
  filesystem access.
- General agent containers must not mount `/var/run/docker.sock`.
- Tool grants combine Role baseline, Project policy, Task grant, and temporary
  Approval grant. A role name alone grants no unrestricted authority.
- Internet may be available for practicality, but policy must support later
  allow/deny restrictions; internal endpoints are not implicitly exposed.
- Developer may modify development/test schema and data when task-authorized. QA may
  create disposable test data. Neither receives production write access by default.
- Production workflow activation, deployment, database migration, destructive
  operations, security/infrastructure changes, paid material services, financial
  actions, and external client/Owner impersonation are approval-gated.

## CONFIRMED — Data, Memory, Logs, and Artifacts

- Raw production/client-sensitive data is not used in development/testing by
  default; use synthetic, fixture, anonymized, or sanitized data.
- Secret values are prohibited memory content. Agent-generated facts carry
  provenance/trust and cannot automatically become authoritative.
- Original files remain protected Artifacts; derived text and summaries are
  separate. Large files use object storage with database references.
- Canonical Events contain safe metadata, not raw prompt contexts, source trees,
  recordings, secrets, environments, or verbose stack traces.
- Persist safe structured results, model/template versions, context references,
  usage, latency, status, artifact references, error class, and redacted diagnostics.
- Raw prompts/responses are not retained indefinitely by default; temporary debug
  capture is policy-controlled and redacted.
- Automatic redaction should detect common API keys, tokens, passwords, cookies,
  Authorization headers, service keys, private keys, and secret environment names,
  but must never be represented as perfect. Minimize collection first.

## CONFIRMED — Audit and Retention

- Audit differentiates HUMAN, AGENT, SERVICE, and EXTERNAL_SYSTEM and records the
  requested-by, authorized-by, executed-by, correlation, and delegation chain.
- Canonical events are retained long-term by default unless explicit data policy
  governs archive/deletion.
- Deletion distinguishes content from minimum non-sensitive audit evidence.
  Required content is removed/anonymized rather than retained for convenience.
- Detailed logs default to finite retention; incident records/workspaces may be
  explicitly pinned.
- Hidden chain-of-thought is not collected as an audit requirement. Store useful
  rationale summaries, evidence, decisions, findings, and errors.

## CONFIRMED — Incident and Recovery Controls

- Owner is the initial human operator for deployment approval, restores, credential
  rotation, infrastructure decisions, incidents, and production access.
- Agents may detect, diagnose, recommend, and prepare changes but may not silently
  execute high-risk production recovery.
- Graceful cancellation is default; force termination is explicit, emergency-only,
  and audited.
- Durable state supports safe n8n/VPS restart and missed-callback recovery.
- Backup/recovery covers managed PostgreSQL, durable Artifacts, reproducible n8n
  workflows/configuration, secure credential recovery, and important metadata.

## ASSUMED

- Recent-authentication thresholds and Approval expiry vary by risk; exact durations
  are not yet selected.
- Current business-hours reliability and retention defaults are adequate initially,
  subject to operational and privacy validation.

## UNKNOWN

- Secret manager/broker and ephemeral token mechanism.
- Command/callback signature formats, network topology, rate limits, and trusted
  ingress implementation.
- Object storage access/bucket policy.
- Safe containerized-build mechanism without host Docker socket.
- Legal/regulatory retention and deletion requirements by Client.
- Exact incident severity levels, RTO/RPO, budget controls, and centralized security
  monitoring platform.

## DEFERRED

- Mandatory MFA, OAuth/passkeys, multiple human roles, client identities,
  multi-agency tenant enforcement, separate Worker hosts, and enterprise security/
  availability controls.

## Security Review Gate

Before any implementation phase crosses a boundary, confirm its threat model,
credential scopes, validation, RLS, idempotency, logs/redaction, failure recovery,
and approval behavior. Security controls must not be disabled merely to make a demo
pass.
