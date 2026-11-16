# Advanced: Multi-Model Routing with 9router

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

## 1. Install 9router

```bash
npm install -g 9router
9router
```

The dashboard runs at `http://localhost:20128`; configuration lives in
`~/.9router`.

## 2. Add providers and models

In the dashboard, connect the providers you actually have (Anthropic
subscription, GLM, MiniMax, Kilocode, free tiers, ...). Send one test message
per model to confirm it works before wiring it into a combo.

## 3. Create combos

Dashboard → **Combos** → Create New. A combo is an ordered list of models
plus a strategy:

| Combo name | Ordered models | Strategy |
|------------|----------------|----------|
| `planner`  | strongest reasoning model → cheaper fallback | Fallback |
| `builder`  | cheap, fast coding model → free fallback     | Fallback |
| `judge`    | strongest reasoning model                    | Fallback |

- **Fallback**: try models in order; move to the next on error or quota
  exhaustion.
- **Round Robin / sticky**: rotate load across models or accounts.

## 4. Connect Claude Code to 9router

Either environment variables:

```bash
export ANTHROPIC_BASE_URL=http://127.0.0.1:20128
export ANTHROPIC_API_KEY=9r_yourkey   # generate in the dashboard
```

or `~/.claude/config.json`:

```json
{
  "anthropic_api_base": "http://localhost:20128/v1",
  "anthropic_api_key": "your-9router-api-key"
}
```

9router handles the Anthropic ↔ OpenAI ↔ Gemini format translation, so Claude
Code talks to it as if it were the Anthropic API.

## 5. Point the pipeline agents at your combos

Pick **one** of the following:

**Option A — direct (recommended):** fork this repo and set the agents'
`model:` fields to your combo names:

```yaml
# agents/coder.md
model: builder
```

**Option B — alias shadowing:** name your combos `sonnet` and `opus`, and the
plugin's default agent files route to them with zero edits.

**Option C — environment remap:**

```bash
export ANTHROPIC_DEFAULT_SONNET_MODEL=builder
export ANTHROPIC_DEFAULT_OPUS_MODEL=planner
```

## Example budget setup

| Pipeline role | Model policy |
|---------------|--------------|
| main loop (orchestrator) | mid-tier — it only dispatches and tracks state |
| architect | strongest reasoning you can afford |
| coder | cheap + fast, with a free-tier fallback |
| code-reviewer | strong — a weak reviewer ruins the pipeline |
| senior | strongest — it runs rarely, so cost is negligible |

## Notes and troubleshooting

- Combos are dashboard-managed state under `~/.9router`; there is no
  documented combo config-file spec, so create and tune them in the UI.
- Community issues occasionally report combo-strategy quirks (for example
  fallback behaving like round-robin in some versions). If routing looks
  wrong, test the combo directly with a one-off prompt before debugging the
  pipeline.
- Keep the 9router process running whenever you use the pipeline.
