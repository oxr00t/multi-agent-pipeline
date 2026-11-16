# Multi-Agent Pipeline for Claude Code

English | [مستندات فارسی](README.fa.md)

Turn Claude Code into an autonomous multi-agent dev team. One command runs a
full build pipeline: the **architect** plans, the **coder** implements, the
**code-reviewer** verifies, and the **senior** agent unblocks deadlocks —
no babysitting required.

![Parallel subagents running in Claude Code](docs/parallel-agents.png)

## Why

In typical single-agent coding loops, the model that plans is the same model
that writes and the same model that reviews — so drift goes unnoticed. This
plugin separates the roles into four specialized subagents bound by a strict
state machine. And with the **Combo** feature of
[9router](https://github.com/decolua/9router), you can give each role its own
model (strong models for judgment, cheap/free models for throughput).

## Install

```
/plugin marketplace add YOUR_GITHUB_USERNAME/multi-agent-pipeline
/plugin install multi-agent-pipeline@multi-agent-pipeline
```

## Quick start

In any project:

```
/multi-agent-pipeline:pipeline Build a snake game with HTML canvas and localStorage high scores
```

What happens:

1. On first run, the protocol is scaffolded into your project's `CLAUDE.md`
2. architect → produces `TASKS.md` (architecture, tasks, acceptance criteria)
3. For each task: coder → code-reviewer; `STATUS: REJECTED` means a retry
   with the reviewer's exact fixes
4. After 3 consecutive rejections on one task, the senior agent escalates
5. Stops only at `ALL_TASKS_COMPLETE` or `PIPELINE_FATAL_ERROR`

## The team

| Agent | Role | Default model | Output contract |
|---|---|---|---|
| architect | system design, task decomposition, `TASKS.md` | inherit (your main model) | `ARCHITECTURE_READY` |
| coder | implements only the active task | sonnet | `IMPLEMENTATION_COMPLETE` |
| code-reviewer | verifies against acceptance criteria | inherit | `STATUS: APPROVED` / `STATUS: REJECTED` |
| senior | root-cause escalation after 3 rejections | opus | `ROOT_CAUSE` / `DECISION` / `REQUIRED_CHANGES` |

## How it works (state machine)

```
PLANNING → CODING → REVIEWING ─┬─ APPROVED → next task → ... → ALL_TASKS_COMPLETE
                               └─ REJECTED → RETRYING ─┬─ (3rd rejection) → ESCALATING → ...
```

The protocol ships as a skill (`pipeline-protocol`) plus a slash command
(`pipeline`), and is mirrored into your project's `CLAUDE.md` so every agent
reads the same rules.

## Give each agent a different model (advanced)

By default everything works with standard Claude Code model aliases. To route
each agent to different, cheaper, or free models through 9router's Combo
feature, see [docs/9router-combo.md](docs/9router-combo.md).

## Permissions & safety

- Plugin agents' `permissionMode` is ignored by Claude Code by design. The
  architect is read-only in practice via its tool allowlist
  (Read/Glob/Grep/Web); the coder and senior can write files and run
  commands.
- Run it in a project you trust, start from a clean git state so diffs stay
  reviewable, and keep your main-loop permission prompts on if you want a
  kill switch.
- Autonomous does not mean unattended: it never asks questions, so review
  the result like a pull request.

## Customization

Fork the repo, edit
`plugins/multi-agent-pipeline/agents/*.md` (models, tools, prompts), and
install from your fork. Everything is plain Markdown — no build step.

## FAQ

**Does it work without 9router?**
Yes — with standard Claude Code models. 9router is optional, for multi-model
routing.

**Where does `TASKS.md` come from?**
The architect creates it in your project root on first run.

**Can I use only part of the team?**
Yes — the agents work individually too (see `/agents`), but the pipeline
protocol is what ties them together.

## License

MIT
