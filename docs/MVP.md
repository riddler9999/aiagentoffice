# Headless MVP Scope and Acceptance

## Classification

- **CONFIRMED:** required for the stated MVP unless explicitly labeled follow-up.
- **ASSUMED:** planning/capacity default.
- **UNKNOWN:** implementation choice still open.
- **DEFERRED:** outside the first MVP boundary.

## CONFIRMED — Objective

Prove a real end-to-end software-delivery loop, not merely AI conversation:

```text
Owner → command → PM context/clarification/plan → exact Owner approval
→ persisted Task/Subtasks → async Developer/Codex execution
→ isolated Git branch/commit/push/real PR → independent QA
→ actual QA failure → structured correction → new Developer run
→ QA pass → Owner final review → review-ready PR
```

The MVP stops before merge and production deployment.

## CONFIRMED — Included Owner Experience

- Supabase Auth email/password login with no public signup.
- Open/access a configured Project and submit a natural-language Task.
- Read and answer clarifications.
- Inspect and approve/reject an exact plan snapshot.
- View Task, Subtask, Run, Review, and relevant Delivery state.
- View implementation and QA reports plus branch, commit, and PR references.
- Give final approval/rejection without automatic merge.
- Pause, cancel, and retry where applicable.
- View Web notifications and a basic operational dashboard.

A polished Project-creation UI and Owner reassign control are not acceptance
blockers. Project configuration must nevertheless be persistent, controlled, and
not hard-coded.

## CONFIRMED — Included Runtime and Operations

- Durable idempotent command intake and clarification/approval flow.
- PM Task/Subtask planning and scoped context retrieval.
- Async Developer and QA dispatch with callbacks plus reconciliation.
- Codex in a scoped isolated clean Git workspace.
- Real code/checks/branch/commit/push/PR against GitHub.
- Independent QA PASS/FAIL/BLOCKED with evidence.
- Durable structured Handoffs and a maximum of three correction cycles.
- Immutable Run attempts and append-only canonical events.
- Scoped credentials, audit attribution, timeout, retry, graceful cancellation, and
  emergency force termination.
- Lost callback and external-side-effect reconciliation.
- Basic health/capacity, queue/failure/stale visibility, logs, usage/cost estimate,
  workspace cleanup, and Development/Production separation.

## CONFIRMED — Formal Acceptance Scenario

Use a controlled sample repository with deterministic acceptance criteria and a
deliberately imperfect first implementation. QA rejection must originate from a
real test/check/acceptance mismatch, not a fabricated status. Each correction is a
new Run and previous Reviews remain visible.

After formal acceptance, validate the system against one real Agency/client
repository under its policy. That second validation does not justify weakening
controlled acceptance.

## CONFIRMED — Exit Checklist

1. Owner authenticates; public signup is unavailable.
2. A persistent configured Project/repository/policy exists.
3. Owner submits a natural-language software Task durably.
4. PM retrieves correctly scoped context.
5. PM asks and persists required clarification.
6. PM creates a structured plan and acceptance criteria.
7. Owner approves the exact immutable plan snapshot.
8. Task/Subtasks and approval are persisted.
9. Developer Job dispatch is asynchronous and durable.
10. Codex runs in an isolated scoped workspace.
11. Developer makes real repository changes and runs relevant checks.
12. A task branch and attributable commit are created and pushed.
13. A real review-ready GitHub PR is opened idempotently.
14. QA runs independently from approved requirements and current code evidence.
15. QA rejects the intentionally imperfect implementation based on real evidence.
16. Structured correction feedback reaches Developer through a durable Handoff.
17. A distinct correction Run changes the implementation.
18. QA reruns independently and passes.
19. Implementation and QA reports/artifact references persist.
20. Owner sees PR, reports, evidence, lifecycle, and relevant notifications.
21. Owner can give final approval/rejection; no merge occurs automatically.
22. Agent Run attempts retain identity/input/history; Events remain append-only.
23. A deliberately lost completion callback is reconciled without duplicate GitHub
    or other side effects.
24. n8n restart during durable work does not lose authoritative state and recovery
    resumes/reconciles safely.
25. Pause/cancel/retry behave within policy and remain auditable.
26. Human, Agent, Service, and External System actors are distinguishable.
27. Worker, GitHub, and service credentials are scoped and not exposed to clients.
28. Capacity overflow and pending work remain durably queued.
29. Temporary workspace cleanup is demonstrated or verifiably scheduled.
30. Dashboard exposes required active/queued/blocked/failed/stale work, approvals,
    health, capacity, critical errors, and available cost/usage data.
31. The Agency remains correct when the UI is disconnected.
32. Relevant checks pass and known limitations/failures are reported rather than
    hidden.

All checklist items must have executable or inspectable evidence before the
Headless MVP is declared operationally proven.

## CONFIRMED — Required Recovery Tests

### Lost callback

Complete a Worker job while suppressing normal callback handling. Reconciliation
must discover actual job/result/GitHub state, update durable state, and reuse the
existing branch/commit/PR without duplicating effects.

### n8n restart

Restart orchestration while a durable Task/Run exists. On recovery, n8n must use
Supabase state/operations—not memory or an old open request—to resume or reconcile.

### Worker crash (SHOULD for first demo)

Architecture support is required. If time constrains the first demonstration, test
immediately afterward that stale/crashed work becomes failed/timed-out and a retry
creates a new linked Run.

## CONFIRMED — Not Acceptance Blockers

- Polished Project creation/configuration UI.
- Telegram notifications (immediate follow-up).
- Worker-crash demonstration only if it would delay the first demo; support remains
  mandatory.
- Reassign UI control if PM can perform policy-bound reassignment.
- Exact billing reconciliation and enterprise monitoring.

## ASSUMED

- Initial capacity and retention defaults are those in `PRODUCT.md` and
  `REQUIREMENTS.md`; tune them using measured data.
- A purpose-built sample repository will be created/selected in a later approved
  implementation task; none is selected by this document.

## UNKNOWN — Must Be Decided During Phase Planning

- Command API and durable inbox implementation.
- Database schema/transition transaction mechanism.
- Worker runtime, queue, isolation/build, callback, and Codex invocation.
- Sample repository and exact intentional failure/acceptance fixture.
- Artifact storage, credential broker, model/provider configuration, health/logging
  tooling, and cost-pricing source.

## DEFERRED Beyond First Headless MVP

- Automatic merge and production deployment.
- Polished Project management, Telegram polish, email, full Staging, advanced
  analytics, exact invoice reconciliation, multi-user/client access, additional
  Git providers/repositories, advanced Agents, and Game implementation.

## Phase Order

1. **Phase 0:** Discovery and architecture documents (this baseline).
2. **Phase 1:** Supabase foundation and stable persistence contracts.
3. **Phase 2:** n8n orchestration and Worker integration foundation.
4. **Phase 3:** End-to-end headless Agency MVP and acceptance evidence.
5. **Phase 4:** Game integration against stable domain contracts.
6. **Phase 5:** Advanced multi-agent capabilities.

Owner review/approval is required before Phase 1 implementation starts.
