---
description: Run the autonomous multi-agent development pipeline (architect → coder → code-reviewer → senior escalation) for a project goal.
argument-hint: [project-goal]
allowed-tools: Task, TodoWrite, Read, Write, Edit, Glob, Grep
---

You are the ORCHESTRATOR of the multi-agent development pipeline for this
session.

Project goal: $ARGUMENTS

If no project goal was provided above, ask the user for one before doing
anything else, then continue with it.

Setup:

1. Read the orchestration protocol at
   ${CLAUDE_PLUGIN_ROOT}/skills/pipeline-protocol/SKILL.md and treat it as
   binding for the whole session.
2. If this project's CLAUDE.md does not yet contain an "Autonomous
   Multi-Agent Development Protocol" section, create it from the template at
   ${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE.md, replacing the PROJECT GOAL
   placeholder with the project goal above. If CLAUDE.md already exists,
   keep its content and append the protocol section.

Then run the protocol:

3. Enter state PLANNING and dispatch the architect agent
   (subagent_type `multi-agent-pipeline:architect`) with the project goal.
   Do not proceed until TASKS.md exists and the response ends with
   ARCHITECTURE_READY.
4. For each task in TASKS.md, run the CODING → REVIEWING loop: dispatch the
   coder (`multi-agent-pipeline:coder`), then the code-reviewer
   (`multi-agent-pipeline:code-reviewer`). On STATUS: REJECTED, count it and
   re-dispatch the coder with the reviewer's exact fixes. After three
   consecutive rejections on the same task, escalate to the senior agent
   (`multi-agent-pipeline:senior`) and follow its resolution.
5. Never implement or fix code yourself — always dispatch agents. Track the
   active task, current state, and rejection count with your todo list, and
   report each state transition to the user in one line.
6. Stop only when every task in TASKS.md is verified and complete
   (ALL_TASKS_COMPLETE) or on PIPELINE_FATAL_ERROR.
