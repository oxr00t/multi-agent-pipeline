# Advanced: Multi-Model (and Free) Routing with 9router

The pipeline works out of the box with plain Claude Code: the agents use the
standard model aliases (`inherit`, `sonnet`, `opus`). The real payoff of this
setup, though, is giving **each agent a different model** — a strong model
where judgment matters (planning, reviewing, escalation) and a cheap or free
model where throughput matters (implementation).

[9router](https://github.com/decolua/9router) is a free, open-source,
self-hosted AI router/proxy (dashboard + local API) that makes this easy. Its
**Combo** feature chains multiple models/providers into one virtual model
with automatic fallback or round-robin — and since a combo is just a model
name, it pairs directly with the `model:` field in this plugin's agent files.
Build your combos from free providers and the whole team runs for free.

## 1. Install 9router

```bash
npm install -g 9router
9router
```

The dashboard runs at `http://localhost:20128`; configuration lives in
`~/.9router`.

## 2. Let 9router configure Claude Code (CLI Tools)

In the dashboard, open **CLI Tools → Claude Code** (available when Claude
Code is installed). 9router generates the config for you — an `env` block
like this:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://127.0.0.1:20128/v1",
    "ANTHROPIC_AUTH_TOKEN": "<9router api key>",
    "ANTHROPIC_DEFAULT_FABLE_MODEL": "claude",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "opus",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "sonnet",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "claude-haiku-4-5-20251001"
  }
}
```

That is: Claude Code talks to 9router as if it were the Anthropic API, and
the standard aliases (`sonnet`, `opus`, `haiku`, ...) are remapped to the
model/combo names you choose. 9router handles the Anthropic ↔ OpenAI ↔ Gemini
format translation.

## 3. Create combos named exactly `sonnet` and `opus`

Dashboard → **Combos** → Create New. A combo is an ordered list of models
plus a strategy (**Fallback** = try in order, move to the next on error or
quota exhaustion; **Round Robin** = rotate load).

The trick: **name the combos exactly like the alias names** in the env remap
(`sonnet`, `opus`). Then the plugin's default agent files route to your
combos with zero edits:

| Combo name | Used by | Example contents |
|------------|---------|------------------|
| `opus`     | senior agent (and anything on `opus`) | strongest reasoning model first, cheaper fallback — e.g. `cx/gpt-5.6-terra[1m]` → `cx/gpt-5.6-luna` |
| `sonnet`   | coder agent (and anything on `sonnet`) | cheap/fast/free models — e.g. `cf/zai-org/glm-4.7-flash` → `groq/openai/gpt-oss-120b` → `ollama/glm-4.7-flash` |

Everything on `inherit` (architect, code-reviewer, and the main orchestrator
loop) keeps using whatever model your main Claude Code session runs on — set
that combo/model in the dashboard too if you want.

A fully free example: build `sonnet` from free tiers (GLM flash, `gpt-oss`
on Groq, a local Ollama model) and `opus` from your best free reasoning
model — the pipeline then costs nothing.

## 4. That's it

Send one test message per combo to confirm routing, then just run:

```
/pipeline:run Your project goal
```

## Alternatives (optional)

You don't have to shadow the alias names:

- **Direct:** fork the repo and set the agents' `model:` fields to any combo
  name you created (e.g. `model: builder`).
- **Custom remap:** point the env vars at different combo names, e.g.
  `ANTHROPIC_DEFAULT_SONNET_MODEL=builder`.

## Notes and troubleshooting

- Combos are dashboard-managed state under `~/.9router`; there is no
  documented combo config-file spec, so create and tune them in the UI.
- Community issues occasionally report combo-strategy quirks (for example
  fallback behaving like round-robin in some versions). If routing looks
  wrong, test the combo directly with a one-off prompt before debugging the
  pipeline.
- Keep the 9router process running whenever you use the pipeline.
