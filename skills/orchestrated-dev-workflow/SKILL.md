---
name: orchestrated-dev-workflow
description: Run a staged Codex subagent workflow for software changes. The main agent acts only as an orchestrator and message router. It must not perform requirements analysis, architecture, coding, testing, or requirements review itself.
---

# Orchestrated Dev Workflow

Use this skill when the user wants to implement a software change using a staged subagent workflow.

The main agent is **only an orchestrator**.

The main agent must not:
- inspect the codebase
- analyze requirements
- brainstorm solutions
- design architecture
- write code
- edit files
- run tests
- review the implementation
- review or approve requirements itself
- reinterpret subagent outputs
- fill in missing requirements by itself
- make technical decisions on behalf of specialist agents
- proceed past requirements with assumptions, gaps, contradictions, or unanswered questions

The main agent may only:
- start the correct subagent at the correct stage
- pass inputs from one stage to the next
- forward subagent questions to the user
- pass user answers back to the relevant subagent
- stop the workflow when a stage requires user input
- summarize the workflow status using only subagent-provided conclusions
- report the final subagent verdict

The specialist agents own the actual work.

## Required Agents

This workflow expects the following Codex subagents to exist:

1. `requirements_gatherer`
2. `requirements_reviewer`
3. `architect`
4. `coder`
5. `tester`

## Required Stage Order

Always run the stages in this order:

```text
requirements_gatherer -> requirements_reviewer -> architect -> coder -> tester
```

Do not skip `requirements_reviewer`.
Do not spawn `architect` until `requirements_reviewer` returns `REQUIREMENTS_APPROVED_FOR_ARCHITECTURE`.

## Core Principle

Do not let every agent reinterpret the whole problem independently.

Each stage must consume the previous stage's written handoff as its source of truth.

Ownership boundaries:

```text
requirements_gatherer
  Owns: what should be built, what is unclear, finalized requirements, acceptance criteria.
  Must stop with user questions whenever anything is unclear or any assumption is detected.

requirements_reviewer
  Owns: auditing requirements against the original user request and gathered user answers.
  Must confirm there are no assumptions, gaps, contradictions, or request mismatches before architecture.

architect
  Owns: how reviewer-approved finalized requirements should be implemented in the existing codebase.

coder
  Owns: implementing the architect's plan with a focused code change.

tester
  Owns: verifying the implementation against reviewer-approved finalized requirements and the architecture plan.
```

## Requirements Gate

The workflow must not proceed to architecture while requirements contain assumptions.

Allowed requirements_gatherer readiness values:
- `NOT_READY_FOR_REQUIREMENTS_REVIEW`
- `READY_FOR_REQUIREMENTS_REVIEW`

Allowed requirements_reviewer verdicts:
- `REQUIREMENTS_APPROVED_FOR_ARCHITECTURE`
- `REQUIREMENTS_NEED_USER_INPUT`
- `REQUIREMENTS_NEED_GATHERER_REVISION`

If `requirements_gatherer` returns `NOT_READY_FOR_REQUIREMENTS_REVIEW`:
- Stop the workflow.
- Forward the gatherer's questions to the user.
- Pass the user's answer back to `requirements_gatherer`.
- Resume only from `requirements_gatherer`.

If `requirements_gatherer` returns `READY_FOR_REQUIREMENTS_REVIEW`:
- Spawn `requirements_reviewer`.
- Pass it the original user request, all gathered user answers, and the finalized requirements brief.

If `requirements_reviewer` returns `REQUIREMENTS_NEED_USER_INPUT`:
- Stop the workflow.
- Forward the reviewer's questions to the user.
- Pass the user's answer back to `requirements_gatherer` and then rerun `requirements_reviewer`.
- Do not route around the gatherer; the gatherer owns the finalized requirements brief.

If `requirements_reviewer` returns `REQUIREMENTS_NEED_GATHERER_REVISION`:
- Stop the workflow.
- Pass the reviewer feedback back to `requirements_gatherer`.
- Rerun `requirements_reviewer` after the gatherer revises the brief.

If `requirements_reviewer` returns `REQUIREMENTS_APPROVED_FOR_ARCHITECTURE`:
- Pass the finalized requirements brief and reviewer report to `architect`.
- Continue the staged workflow.

## Handoff Rules

- The raw user prompt is background context after requirements are finalized; it is not a substitute for approved requirements.
- The architect must design only from reviewer-approved finalized requirements.
- The coder must implement only from the architect's plan.
- The tester must verify only against reviewer-approved finalized requirements and the architecture plan.
- If any downstream agent finds a requirements gap, stop and route the issue back through `requirements_gatherer` and `requirements_reviewer`.
