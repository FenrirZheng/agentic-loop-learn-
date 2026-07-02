# agentic-loop-learn

A knowledge base of LLM/agentic **control-structure** techniques — and a Claude Code **plugin** that turns that knowledge into an actionable skill for composing robust agentic prompts.

## Two layers

1. **Knowledge base** — 39 techniques, one file each (prompt example + why-it-works + pitfalls), aggregated in [index.md](index.md), grounded in the theory doc [agentic-loop.md](agentic-loop.md).
   - Core families: [examples/](examples/) · Frontier patterns: [examples/frontier/](examples/frontier/)
2. **Plugin** — [agentic-prompt-composer](skills/agentic-prompt-composer/SKILL.md): a skill + `/agentic-prompt` slash command that diagnoses a task, selects the right primitives, stacks them, and emits a ready-to-run agentic prompt with an explicit stop condition and the anti-degeneration guardrails baked in.

## Install the plugin

This repo is both a plugin and a local marketplace. From Claude Code:

```
/plugin marketplace add /home/fenrir/code/agentic-loop-learn
/plugin install agentic-prompt-composer@agentic-loop-learn
```

Then either let the skill trigger automatically ("help me design an agent loop for X"), or call it explicitly:

```
/agentic-prompt find every race condition in this service and only report the real ones
```

Validate the manifests any time:

```
claude plugin validate /home/fenrir/code/agentic-loop-learn --strict
```

## The one idea to remember

**Feedback objectivity is the main line.** Self-correction without an external signal degenerates. If an objective oracle exists (tests, compiler, schema, retrieval), build the loop around it; if not, prefer generate-then-select over naive self-refine. Everything else is a variation on how to spend inference-time compute — sequential (loops), parallel (generate-then-select), or decomposed.

## Layout

```
.claude-plugin/         plugin.json + marketplace.json
skills/agentic-prompt-composer/
  SKILL.md              the composition workflow (5 steps)
  references/
    catalog.md          diagnosis -> primitive decision tree + full table
    recipes.md          ready-to-adapt composition stacks (R1..R7)
    composition-guide.md stacking rules + anti-patterns
  evals/evals.json      eval cases for `claude plugin eval`
commands/agentic-prompt.md   the /agentic-prompt slash command
examples/               39 technique deep-dives
index.md                aggregator of the knowledge base
agentic-loop.md         underlying theory + sources
```
