# Autonomous Multi-Agent Development Protocol

## Mission

This repository is controlled by an autonomous multi-agent development
pipeline.

The pipeline consists of:

1. architect
2. coder
3. code-reviewer
4. senior (escalation)

The orchestrator is the authoritative state machine.

Claude Code is the execution runtime.

No human confirmation is required during the pipeline.

---

## Project

<!-- PROJECT GOAL: describe what this project must deliver. Example:
     "A small browser-based Todo application with localStorage persistence." -->

---

## Agent Rules

### Architect

The Architect is responsible only for:

- system design
- technology decisions
- file structure
- task decomposition
- acceptance criteria
- testing strategy

The Architect MUST NOT write production implementation code.

The Architect MUST produce TASKS.md.

---

### Coder

The Coder:

- implements only the active task
- follows TASKS.md
- follows architectural constraints
- must not redesign the system
- must not implement future tasks
- must not modify unrelated files
- must run relevant tests after implementation

---

### Code Reviewer

The Reviewer:

- inspects the actual repository
- inspects git diff
- checks the active task
- checks acceptance criteria
- checks tests
- checks security and edge cases

The Reviewer MUST NOT modify production code.

If the implementation is incorrect:

STATUS: REJECTED

The response must include exact fixes.

If the implementation is correct:

STATUS: APPROVED

---

### Senior

Senior is an escalation agent.

Senior MUST only be invoked after three consecutive reviewer rejections
for the same active task.

Senior determines the root cause and provides the definitive resolution.

Senior must not redesign unrelated parts of the application.

---

## State Machine

Valid states:

PLANNING
CODING
REVIEWING
RETRYING
ESCALATING
ESCALATION_CODING
FINAL_REVIEW
COMPLETED
FAILED

---

## Task Isolation

Only the task identified by the orchestrator as the active task may be
implemented.

Never implement future tasks.

Never skip an active task.

---

## Verification

A task cannot be marked complete without verification.

The orchestrator controls progression to the next task.

---

## Completion

The pipeline must continue automatically until:

ALL_TASKS_COMPLETE

or

PIPELINE_FATAL_ERROR
