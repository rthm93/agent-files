---
name: orchestrated-dev-workflow
description: Explicit invocation only. Use this staged Codex subagent workflow only when the user names `orchestrated-dev-workflow` or directly asks to use this exact skill.
---

# Orchestrated Dev Workflow

## Activation Policy

Use this skill only when the user explicitly invokes it by name, for example
`orchestrated-dev-workflow`, or directly asks Codex to use this exact skill.

Do not activate this skill just because a prompt asks for software changes,
subagents, staged planning, requirements gathering, architecture, TDD, testing,
or an approval-gated workflow.

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
- read bundled agent TOML profile files only to copy their contents into subagent spawn prompts
- pass inputs from one stage to the next
- forward subagent questions to the user
- pass user answers back to the relevant subagent
- stop the workflow when a stage requires user input
- open the planner-written Markdown architecture plan file and TDD test plan file in VS Code for user review
- summarize the workflow status using only subagent-provided conclusions
- report the final subagent verdict

The specialist profile prompts own the actual work.

## Required Profile Files And Built-In Roles

This workflow uses Codex built-in subagent roles and the bundled TOML files as
profile prompts. The TOML files are not automatically registered as custom
subagent types by the plugin manifest.

Required bundled profile files, resolved relative to the plugin root:

1. `agents/requirements_gatherer.toml`
2. `agents/requirements_reviewer.toml`
3. `agents/architect.toml`
4. `agents/tdd_test_planner.toml`
5. `agents/coder.toml`
6. `agents/tester.toml`

When this skill is loaded from `skills/orchestrated-dev-workflow/SKILL.md`,
the bundled profile files are at `../../agents/*.toml` from this file's
directory. Do not resolve `agents/*.toml` relative to the user's active
workspace unless the user explicitly points this workflow at a different
profile bundle.

When spawning a stage with Codex's built-in subagent facility such as
`multi_agent_v1`, use this mapping:

| Workflow stage | Built-in subagent role | Profile file to inject |
| --- | --- | --- |
| `requirements_gatherer` | `default` | `agents/requirements_gatherer.toml` |
| `requirements_reviewer` | `default` | `agents/requirements_reviewer.toml` |
| `architect` | `default` | `agents/architect.toml` |
| `tdd_test_planner` | `default` | `agents/tdd_test_planner.toml` |
| `coder` | `worker` | `agents/coder.toml` |
| `tester` | `worker` | `agents/tester.toml` |

Prefer `default` for requirements gathering, requirements review, architecture,
and test planning. If `default` is unavailable, use `worker` for those stages.

Use `explorer` only for a bounded codebase question requested by the owning
stage. Its answer must be returned to the owning stage and must not bypass any
gate or become a substitute for the required stage handoff.

The main agent must also have access to plan-file locations inside the active
workspace. If the user does not provide them, use Markdown files under
`docs/plans/` with descriptive timestamped filenames. The architecture plan and
TDD test plan are separate artifacts.

## Required Stage Order

Always run the stages in this order:

```text
requirements_gatherer -> requirements_reviewer -> parallel(architect, tdd_test_planner) -> plan_consistency_check -> user_plan_approval -> coder -> tester
```

Do not skip `requirements_reviewer`.
Do not spawn `architect` or `tdd_test_planner` until `requirements_reviewer`
returns `REQUIREMENTS_APPROVED_FOR_ARCHITECTURE`.
Do not spawn `coder` until the user explicitly approves both final plan files.

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
  Must confirm there are no assumptions, gaps, contradictions, or request mismatches before architecture and TDD test planning.

architect
  Owns: how reviewer-approved finalized requirements should be implemented in the existing codebase, including the final Markdown architecture plan.

tdd_test_planner
  Owns: how reviewer-approved finalized requirements should be proven through unit, integration, e2e, or manual verification, including the final Markdown TDD test plan.

plan_consistency_check
  Owns: confirming the architecture plan and TDD test plan do not conflict before user approval.

user_plan_approval
  Owns: explicit user approval of both plan files before implementation begins.

coder
  Owns: implementing only the user-approved architecture plan and TDD test plan with a focused code change.

tester
  Owns: verifying the implementation against reviewer-approved finalized requirements, the user-approved architecture plan, and the user-approved TDD test plan.
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
- Spawn a built-in subagent with the `requirements_reviewer` profile.
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
- Spawn a built-in subagent with the `architect` profile and pass it the finalized requirements brief and reviewer report.
- In parallel, spawn a built-in subagent with the `tdd_test_planner` profile and pass it the finalized requirements brief and reviewer report.
- Continue the staged workflow.

## Plan Consistency And Approval Gate

Implementation must not begin until the user has approved both the final
architecture plan and final TDD test plan.

After both `architect` and `tdd_test_planner` complete and before spawning
`coder`:

1. Require the architect to create or update a final architecture plan in a
   Markdown plan file inside the active workspace.
2. Require the architect to return the exact repo-relative architecture plan
   file path.
3. Require the TDD test planner to create or update a final TDD test plan in a
   Markdown plan file inside the active workspace.
4. Require the TDD test planner to return the exact repo-relative TDD test plan
   file path.
5. Confirm both `architect` and `tdd_test_planner` returned readiness
   `READY_FOR_PLAN_REVIEW`. If either returns `BLOCKED`, stop and report the
   blocker using only that agent's stated reason.
6. Perform a plan consistency check before user approval. The main agent may
   read both plan files and compare them only for direct contradictions between
   stated implementation boundaries, public contracts, validation constraints,
   planned verification scenarios, readiness values, and handoff statements. It
   must not redesign either plan.
7. If the architecture plan and TDD test plan conflict, stop and route the
   conflict back to `architect`, `tdd_test_planner`, or both as appropriate.
8. Open both plan files in VS Code with `code <architecture-plan-file>` and
   `code <tdd-test-plan-file>`.
9. Ask the user to review both plan files and explicitly approve continuing.
10. Stop the workflow until the user approves both plans.

The architecture plan file must include:
- reviewer-approved requirements summary
- relevant architecture decisions
- files or modules expected to change
- implementation steps
- validation coordination and testability considerations that do not duplicate
  the detailed TDD test plan
- known risks, exclusions, or assumptions

The TDD test plan file must include:
- reviewer-approved requirements summary
- requirement-to-test mapping
- planned unit, integration, e2e, or manual verification
- test-first guidance for coder
- validation commands and expected evidence
- coverage gaps, risks, and tester handoff

If the user requests changes to either plan:
- Send architecture-plan changes back to `architect`.
- Send TDD-test-plan changes back to `tdd_test_planner`.
- Require the relevant planning agent to update the same plan file or return the new plan file path.
- Re-run the plan consistency check.
- Reopen or refresh both plan files in VS Code.
- Ask the user for approval again.
- Do not spawn `coder` until the user approves both revised plans.

If the user approves both plans:
- Read the approved Markdown architecture plan file.
- Read the approved Markdown TDD test plan file.
- Spawn a built-in subagent with the `coder` profile and pass the approved
  architecture plan file path, architecture plan contents, TDD test plan file
  path, TDD test plan contents, finalized requirements brief, and reviewer
  report to it.
- Continue to implementation and testing.

After `coder` completes:
- Spawn a built-in subagent with the `tester` profile.
- Pass it the coder's implementation report, the changed-file summary, the
  approved architecture plan file path, architecture plan contents, TDD test
  plan file path, TDD test plan contents, finalized requirements brief, and
  reviewer report.
- Report the tester's final verdict to the user using only the tester-provided conclusions.

## Handoff Rules

- Every stage spawn prompt must include the full contents of that stage's TOML
  profile file. Do not rely on the profile name alone.
- Every stage spawn prompt must clearly state the built-in role being used, the
  workflow stage being performed, and that the injected TOML profile contents are
  authoritative instructions for the stage.
- Every stage spawn prompt must include the stage input: the original user
  request when relevant, all accumulated user answers, the previous stage
  handoff, and the exact gate verdicts or output fields the stage must return.
- The orchestrator must pass the stage output forward verbatim except for
  user-facing summaries and reading the approved Markdown plan files for
  approval and coder/tester handoff.
- The raw user prompt is background context after requirements are finalized; it is not a substitute for approved requirements.
- The architect must design only from reviewer-approved finalized requirements.
- The TDD test planner must plan tests only from reviewer-approved finalized requirements.
- The coder must implement only from the user-approved architecture plan and user-approved TDD test plan.
- The tester must verify only against reviewer-approved finalized requirements, the user-approved architecture plan, and the user-approved TDD test plan.
- If the architecture plan and TDD test plan conflict, stop and route the issue
  back to the relevant planning agent before implementation begins.
- If any downstream agent finds a requirements gap, stop and route the issue back through `requirements_gatherer` and `requirements_reviewer`.
