# Product Definition

## Status Vocabulary

- **CONFIRMED:** explicitly agreed product or architecture requirement.
- **ASSUMED:** planning estimate/default, not a hard requirement.
- **UNKNOWN:** unresolved decision that must not be silently invented.
- **DEFERRED:** intentionally outside the current scope.

## CONFIRMED — Product Purpose

AI Agent Office is an internal AI Automation & Software Development Agency. It
will eventually support requirements analysis, automation and agent design, n8n
workflows, Supabase/database and API integration, full-stack applications,
debugging, testing, QA, and technical documentation.

The first product goal is narrower: accept an Owner task for an existing GitHub
repository and deliver a planned, implemented, independently verified,
review-ready pull request with persistent state and evidence.

The product is not complete merely because it can chat or create a plan. “Done”
requires agreed requirements, verified acceptance criteria, relevant passing
checks, QA pass, required documentation, disclosed limitations, and all required
human approvals. Implementation completion, QA pass, approval, merge, and
deployment are distinct outcomes.

## CONFIRMED — Users and Inputs

- The MVP has one authenticated Agency Owner.
- External clients and internal collaborators do not log in.
- The Owner communicates with clients and supplies requirements, change requests,
  bug reports, repositories, documents, screenshots, recordings, architecture,
  API/database information, feedback, constraints, acceptance criteria, and
  credential references.
- Missing material information triggers clarification; agents must not invent it.
- Projects can belong to a Client or be internal (`client_id` may be absent).

## CONFIRMED — Owner Experience

The headless MVP uses a minimal Vercel-hosted Web UI. The Owner can authenticate,
open a configured Project, submit natural-language commands, answer
clarifications, inspect and approve an exact plan snapshot, monitor milestones,
inspect implementation and QA evidence, and pause, cancel, retry, approve, or
reject as permitted.

Live token streams are not required. The interface emphasizes task/run/review
state, pending approvals, failures, health, cost estimates, and detailed logs on
demand. Web notifications are required; Telegram critical alerts follow shortly
after the core loop.

## CONFIRMED — Initial Agency Roles

- **PM Agent:** retrieves scoped context, analyzes requirements, asks
  clarifications, creates plans/subtasks, assigns and schedules work, coordinates
  approvals, retries/reassignments, dependencies, and escalation. It cannot change
  project scope autonomously.
- **Developer Agent:** operates within an approved task in an isolated workspace;
  inspects, edits, tests, documents, commits, pushes a task branch, and opens a PR.
  It cannot merge protected branches or deploy production.
- **QA Agent:** independently verifies requirements, diffs, tests, builds, lint,
  types, security basics, and regressions. It reports PASS, FAIL, or BLOCKED and
  does not edit production code in the MVP.

Logical Agents are long-lived identities/NPCs. Runs are individual immutable
attempts. Workers are temporary execution processes. These are not interchangeable.

## CONFIRMED — Product Principles

1. Autonomy is bounded by approved scope and least privilege.
2. Every important action is attributable to project, task, agent, run, and any
   required approval.
3. The current repository is more authoritative than stale repository memory.
4. Important outputs are persistent Artifacts; chat alone is not an artifact.
5. The whole Agency operates without the Game.
6. The future Game and current Web UI are replaceable clients of stable contracts.
7. The Game may visualize real events and allow interaction, but does not own AI or
   business logic.

## CONFIRMED — Success Outcome

The controlled acceptance repository must demonstrate:

```text
Owner command → PM clarification/plan → plan approval → persisted work
→ asynchronous Codex Developer run → real branch/commit/push/PR
→ independent QA failure → structured correction handoff
→ new Developer run → QA pass → Owner final review → review-ready PR
```

It must also demonstrate lost-callback reconciliation without duplicate side
effects and n8n restart recovery from durable state. Merge and deployment remain
separate approval-gated actions.

## ASSUMED — Initial Capacity

- 5–15 active projects.
- 5–20 tasks per day.
- 1–3 simultaneous coding/QA jobs.
- Typical jobs last 5–30 minutes; some exceed 60 minutes.
- Most artifacts are small; recordings may be hundreds of megabytes.

These are capacity-planning assumptions, not product limits, and must be measured.

## UNKNOWN

- Monthly infrastructure/LLM budget and alert thresholds.
- Whether the product will ever become multi-agency SaaS.
- Exact UI design beyond required capabilities.
- Exact model/provider choices outside Codex being the primary coding agent.

## DEFERRED

- Client login and multiple internal users.
- Multi-agency SaaS.
- Multi-repository Projects and non-GitHub providers.
- Solution Architect, Researcher, UI/UX, DevOps, Security, and Business Analyst
  agents.
- Automatic merge, production deployment, OAuth, passkeys, and enforced MFA.
- Full staging, advanced analytics, email alerts, exact invoice reconciliation,
  broad vector/RAG retrieval, and the Game implementation.
