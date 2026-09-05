# AI Agent Office — Instructions for Coding Agents

## Scope

These instructions apply to this repository. This project is an AI automation and
software-development agency with three deliberately separated layers:

- **n8n:** agent runtime, orchestration, handoffs, tools, approvals, and automation.
- **Supabase/PostgreSQL:** authoritative state, memory, contracts, and audit history.
- **Game/Web clients:** replaceable visual and interaction layers; never the home of
  core AI or business logic.

An isolated **Coding Worker** runs Codex and repository tools. A logical Agent, an
Agent Run, and a worker process are different concepts.

## Before Making Changes

1. Read the documentation relevant to the change. At minimum, read
   `docs/ARCHITECTURE.md`, `docs/PROTOCOLS.md`, and `docs/SECURITY.md` for changes
   that cross a component boundary.
2. Inspect the current branch, working tree, recent history, and existing changes.
3. Determine requirements and technology from repository evidence. Never invent a
   requirement or silently convert an `UNKNOWN` into a decision.
4. Preserve user-authored and unrelated work. Do not reset, clean, or overwrite a
   dirty workspace.
5. Ask for clarification when a material requirement, authority boundary, or
   security decision is missing.

## Architecture Boundaries

- Keep n8n, Supabase, Coding Workers, Web UI, and the future Game loosely coupled.
- Supabase is the durable source of truth. n8n execution history and Realtime
  delivery are not authoritative business state.
- UI/Game submits versioned commands through an authenticated stable boundary. It
  must not write canonical events or arbitrary lifecycle state.
- n8n coordinates business-state transitions. It must not become the coding
  workspace or the only holder of durable state.
- Workers execute scoped jobs and return structured results. Never give a worker
  unrestricted Supabase business-table or host access.
- External systems remain authoritative for facts they own; verify and reconcile
  GitHub facts rather than fabricating them.
- The Game must consume domain contracts, not n8n workflow internals. It must not
  contain core AI, orchestration, approval, or business logic.

## Protocol and Documentation Change Control

- Treat the `CONFIRMED` semantic rules in `docs/PROTOCOLS.md` as hard contracts.
- Never silently change a command, event, handoff, approval, job, result, callback,
  state, or authentication protocol.
- A breaking contract change requires an explained decision in
  `docs/DECISIONS.md`, a protocol/schema version change, compatibility handling,
  and synchronized updates to all affected documentation and components.
- Keep documentation synchronized with architectural and behavioral changes.
- Clearly label facts and proposals as `CONFIRMED`, `ASSUMED`, `UNKNOWN`, or
  `DEFERRED`. Do not present provisional examples as implemented APIs.

## Implementation and Verification

- Make the smallest complete, maintainable change within the approved scope.
- Avoid unrelated refactors, formatting, dependency upgrades, and cleanup.
- Add or update tests where appropriate. Validate untrusted input at boundaries.
- Preserve idempotency, tenant/project scoping, audit attribution, error handling,
  cancellation, retry limits, and reconciliation behavior.
- Run the relevant formatting, lint, type, unit, integration, build, database,
  browser, workflow, and representative execution checks supported by the change.
- Verify work before declaring completion. Report exact commands and failures.
- Distinguish **implementation complete** from **verified**, **QA passed**,
  **owner approved**, **merged**, and **deployed**.

## Security and Production Safety

- Never expose, print, persist in memory, or commit secrets, credentials, tokens,
  cookies, passwords, private keys, or production connection strings.
- Store only approved credential references in project data. Use separate,
  scoped, rotatable service identities and least privilege.
- Never weaken authentication, authorization, RLS, isolation, validation, tests,
  logging safeguards, or security policy merely to make something work.
- Do not use raw production/client-sensitive data in development or tests by
  default; use synthetic, fixture, anonymized, or sanitized data.
- Treat production deployment, production database mutation, merge, destructive
  operations, security/credential changes, financial actions, and external client
  communication as approval-gated.
- Do not mount the host Docker socket into general coding-agent containers.
- Do not activate, publish, overwrite, or delete live n8n workflows without
  explicit approval, and never embed real credentials in exported workflow JSON.

## Git and Collaboration

- Do not commit directly to `main` or `master`.
- Use an isolated clean clone/worktree and a task branch for meaningful work.
- Do not force-push or rewrite protected history. Keep commits attributable and
  exclude chain-of-thought and sensitive runtime data.
- Record durable handoff context in tracked documentation, commits, PRs, or system
  records rather than relying only on chat history.

## Current Phase

Phase 0 documentation is the current baseline. Do not implement Supabase schemas,
n8n workflows, Workers, Web UI, or Game code until the Owner approves the next
phase.
