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
- edit files, except to create or update the user-facing Markdown plan file for the Plan Approval Gate
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
- create or update the final Markdown plan file using only the architect-provided plan
- open the final Markdown plan file in VS Code for user review
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

The main agent must also have access to a plan-file location inside the active workspace. If the user does not provide one, use a Markdown file under `docs/plans/` with a descriptive timestamped filename.

## Required Stage Order

Always run the stages in this order:

```text
requirements_gatherer -> requirements_reviewer -> architect -> user_plan_approval -> coder -> tester
```

Do not skip `requirements_reviewer`.
Do not spawn `architect` until `requirements_reviewer` returns `REQUIREMENTS_APPROVED_FOR_ARCHITECTURE`.
Do not spawn `coder` until the user explicitly approves the final implementation plan file.

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
  Owns: how reviewer-approved finalized requirements should be implemented in the existing codebase, including the final Markdown implementation plan.

user_plan_approval
  Owns: explicit user approval of the final implementation plan file before implementation begins.

coder
  Owns: implementing only the user-approved architect's plan with a focused code change.

tester
  Owns: verifying the implementation against reviewer-approved finalized requirements and the user-approved architecture plan.
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

## Plan Approval Gate

Implementation must not begin until the user has approved the final implementation plan.

After `architect` completes and before spawning `coder`:

1. Require the architect to provide a final implementation plan in Markdown.
2. Write that final plan to a Markdown plan file inside the active workspace.
3. Open the plan file in VS Code with `code <plan-file>`.
4. Ask the user to review the plan file and explicitly approve continuing.
5. Stop the workflow until the user approves the plan.

The plan file must include:
- reviewer-approved requirements summary
- relevant architecture decisions
- files or modules expected to change
- implementation steps
- validation and test plan
- known risks, exclusions, or assumptions

If the user requests changes to the plan:
- Send the requested changes back to `architect`.
- Update the plan file with the revised final plan.
- Reopen or refresh the plan file in VS Code.
- Ask the user for approval again.
- Do not spawn `coder` until the user approves the revised plan.

If the user approves the plan:
- Pass the approved plan file path, the final plan contents, the finalized requirements brief, and the reviewer report to `coder`.
- Continue to implementation and testing.

## Handoff Rules

- The raw user prompt is background context after requirements are finalized; it is not a substitute for approved requirements.
- The architect must design only from reviewer-approved finalized requirements.
- The coder must implement only from the user-approved architect's plan.
- The tester must verify only against reviewer-approved finalized requirements and the user-approved architecture plan.
- If any downstream agent finds a requirements gap, stop and route the issue back through `requirements_gatherer` and `requirements_reviewer`.
