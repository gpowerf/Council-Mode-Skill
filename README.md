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
| **Creative Strategist** | Novel solutions, out-of-the-box thinking, efficiency. | `google/gemini-2.5-flash-preview-05-20` or `meta-llama/llama-4-*` |
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

## Requirements

- An [opencode](https://opencode.ai) installation.
- An [OpenRouter](https://openrouter.ai) account/API key configured as a provider so the four council models are reachable. See opencode's provider configuration in the [config schema](https://opencode.ai/config.json).

> **Note on model IDs:** The model IDs in the table are the ones referenced by the skill. A few are forward-looking / aspirational identifiers from the paper. Swap them for live OpenRouter model IDs in `SKILL.md` if any are unavailable on your account.

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
README.md
LICENSE
```

---

## License

MIT — see [LICENSE](./LICENSE).
