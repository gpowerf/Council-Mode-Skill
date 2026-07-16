# Council Mode — opencode Skill

> A heterogeneous multi-agent consensus framework that reduces LLM hallucination and bias by dispatching a query to a diverse panel of models, then synthesizing their outputs into a single, balanced answer.

Based on the paper *"Council Mode: A Heterogeneous Multi-Agent Consensus Framework for Reducing LLM Hallucination and Bias"* (arXiv:2604.02923).

This is an [opencode](https://opencode.ai) **skill**. When triggered, the running agent switches into an **Orchestrator** role that runs a strict three-phase consensus process and uses [OpenRouter](https://openrouter.ai) to access a diverse panel of models.

---

## How it works

For every query the skill runs a three-phase process:

### Phase 1 — Intelligent Triage
The Orchestrator analyzes the query and decides whether Council Mode is warranted. Simple, factual questions are answered directly. Complex, multi-step, or high-stakes questions (architectural decisions, code reviews, strategic planning, research synthesis) proceed to Phase 2.

### Phase 2 — Parallel Generation
The query is formulated into a standalone prompt and dispatched **in parallel** to a panel of heterogeneous agents. Each agent answers independently, without seeing the others' responses.

| Role | Persona & Focus | OpenRouter Model ID |
| :--- | :--- | :--- |
| **Analytical Thinker** | Logic, reasoning, breaking down complex problems. Strong at knowledge, math, and software engineering. | `deepseek/deepseek-v4-pro` |
| **Creative Strategist** | Novel solutions, out-of-the-box thinking, efficiency. | `google/gemini-2.5-flash` or `meta-llama/llama-4-maverick` |
| **Critical Reviewer** | Identifying flaws, edge cases, and potential biases with a critical eye. | `anthropic/claude-opus-4.8` |
| **Long-Horizon Reasoner** | Maintaining context, following standards, and handling long-running tasks. | `z-ai/glm-5.2` |

### Phase 3 — Structured Synthesis
The Orchestrator acts as the Consensus Model and produces a structured synthesis:

- **Areas of Agreement** — the foundation of the final answer.
- **Areas of Disagreement / Divergence** — presented as alternative perspectives.
- **Unique Findings** — insights raised by only one agent.

The final output is rendered as:

```
## Final Council Synthesis

### Consensus View
### Divergent Perspectives & Unique Insights
### Final Recommendation & Conclusion
```

---

## Installation

This skill follows the standard opencode skill layout:

```
.opencode/skills/council-mode/SKILL.md
```

### Option A — Copy into your project (project-scoped)

Drop the `.opencode/skills/council-mode/` folder into your repository. opencode auto-discovers any `**/SKILL.md` under `.opencode/skills/`.

### Option B — Register via `skills.paths`

If you keep skills outside your project, point opencode at them in `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "skills": {
    "paths": ["/abs/path/to/Council-Mode-Skill/.opencode/skills"]
  }
}
```

### Option C — Global install

Copy the `council-mode/` folder to `~/.config/opencode/skills/council-mode/` so every project can use it.

> After installing, **quit and restart opencode** so the new skill is loaded.

---

## Usage

Once installed, simply ask a high-stakes or complex question and mention you want it handled with rigor — e.g. *"review this architecture using a council"* or *"I need a highly reliable, unbiased answer on …"*. The skill's description is written so the model triggers it on complex, multi-step, or high-stakes queries.

The model will then run the three-phase process and return a synthesized council output.

---

## Optional: OpenRouter routing config

A ready-to-use `opencode.json` snippet lives in [`examples/opencode.openrouter.json`](./examples/opencode.openrouter.json). It registers the four council model IDs with OpenRouter and sets sensible per-model routing. Copy it into your project (or merge into an existing `opencode.json`):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "openrouter": {
      "models": {
        "anthropic/claude-opus-4.8": {
          "options": {
            "provider": { "order": ["anthropic"], "allow_fallbacks": false }
          }
        }
      }
    }
  }
}
```

What this does (and does **not** do):

- `options.provider` is passed through to OpenRouter's request body, so `order` and `allow_fallbacks` control **which inference provider serves a given model ID** — e.g. pin Claude Opus to Anthropic's own endpoint, or prefer Llama 4 Maverick from Together/Fireworks.
- It is **not** a safety net for non-existent model IDs. `allow_fallbacks: true` only falls back across *providers of the same model*, never to a different model. That's why every ID in the table above is a verified live OpenRouter slug.
- The `~` prefix (e.g. `~anthropic/claude-opus-latest`) is an OpenRouter-side convention for hidden/virtual aliases, not an opencode config key. Don't use it as a key in `provider.*.models`; pass the concrete slug instead.

Run `/connect` once to add your OpenRouter API key, then `/models` to confirm the council models are reachable.

---

## Requirements

- An [opencode](https://opencode.ai) installation.
- An [OpenRouter](https://openrouter.ai) account/API key configured as a provider so the four council models are reachable. See opencode's provider configuration in the [config schema](https://opencode.ai/config.json).

> **Note on model IDs:** All four model IDs are verified **live** on OpenRouter (checked against `GET https://openrouter.ai/api/v1/models`). Swap any of them in `SKILL.md` to suit your account — keep the panel heterogeneous. Context windows and pricing as of this writing: `deepseek/deepseek-v4-pro` (1M ctx, ~$0.44/$0.88 per Mtok), `google/gemini-2.5-flash` (1M ctx, ~$0.30/$2.50), `meta-llama/llama-4-maverick` (1M ctx, ~$0.20/$0.80), `anthropic/claude-opus-4.8` (1M ctx, ~$5/$25), `z-ai/glm-5.2` (1M ctx, ~$0.95/$2.99).

---

## Guidelines enforced by the skill

- **Heterogeneity** — council members represent diverse reasoning styles to minimize bias.
- **Cost awareness** — Council Mode incurs higher token costs; it is only invoked when the accuracy benefits justify the expense.
- **Objectivity** — the Orchestrator remains neutral and synthesizes rather than favoring one agent.
- **Efficiency** — individual agent responses are kept concise to manage context window and cost.
- **Tool use** — agents may use tools (file reading, command execution) per opencode's agent capabilities when the task requires it.

---

## Repository contents

```
.opencode/skills/council-mode/SKILL.md   # the skill itself
examples/opencode.openrouter.json        # OpenRouter routing config (drop-in)
README.md
LICENSE
```

---

## License

MIT — see [LICENSE](./LICENSE).
