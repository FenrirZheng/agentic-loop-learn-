---
name: agentic-prompt-composer
description: Compose a robust agentic prompt or orchestration by selecting and stacking LLM control-structure primitives (loop-until-oracle, self-consistency, best-of-N, adversarial-verify, map-reduce, ReAct, ReWOO, and ~30 more). Output is a markdown prompt/spec. Use when the user wants to DESIGN or STRENGTHEN a prompt/agent/pipeline — e.g. "help me write a prompt that reliably does X", "組合一個 agentic prompt", "which prompting technique for Y", "make this prompt more reliable / stop it hallucinating", "design an agent loop / pipeline / review workflow for Z", "how should I structure the orchestration for W". Do NOT use for one-shot trivial prompts, for prose polishing (that's english-polish), or for explaining existing code (that's explain-code).
---

# Agentic Prompt Composer

Turn a task into a **concrete, ready-to-run agentic prompt or orchestration spec** by picking the right control-structure primitives and stacking them. Backed by the technique catalog in this repo (`agentic-loop.md` + `examples/`).

## The one rule that dominates everything

**Feedback/verification objectivity is the main line.** Self-correction without an external signal *degenerates* — a loop that only asks the model "is this good?" reliably drifts worse. So the first question is always: **is there an objective oracle (tests, compiler, schema, linter, retrieval)?** If yes, build the loop around it. If no, either manufacture one or fall back to generate-then-select — never rely on naive self-refine.

## Workflow

Follow these five steps in order. Do not skip step 1.

### 1. Diagnose the task (answer these; infer from context, ask only if genuinely unknown)

- **Oracle?** Is there objective verification (unit tests, compile, `--json` schema, linter, exact-match answer, retrieval ground truth)?
- **Solution space?** Narrow (one correct answer — math, extraction, classification) or wide (design, writing, naming, architecture)?
- **Primary failure fear?** "Plausible-but-wrong" (correctness) vs "missed something" (completeness/coverage)?
- **Cost / volume?** One-off, or high-volume + cost-sensitive?
- **Size?** Fits one context, or too big (long doc, whole codebase)?
- **Interaction?** Needs to observe the environment mid-task (debug, browse), or fully plannable up front?
- **Horizon & recurrence?** Will the loop run long (> ~5 rounds → needs context curation)? Will this task class recur (→ skill library)? Is the prompt itself the long-lived artifact (→ prompt-optimization loop)?

### 2. Select primitives

Map the diagnosis to primitives using [references/catalog.md](references/catalog.md) (the decision-tree → technique table). The headline mappings:

| Diagnosis | Primary primitive |
|---|---|
| Has an oracle | **until-oracle-passes** (stepwise → PRM) — always first choice |
| Narrow, unique answer | **self-consistency** (vote); don't rush to debate |
| Wide solution space | **best-of-N** / **mixture-of-agents**; **tournament** |
| Fear "plausible but wrong" | **adversarial-verify** (N skeptics, default-refute) |
| Fear "missed something" | **loop-until-dry** + **completeness-critic** |
| Cost-sensitive, high volume | **cascade-routing** (cheap→expensive by confidence) |
| Needs external/fresh knowledge | **agentic-RAG** (retrieval in the loop) |
| Too big | **map-reduce** / **divide-and-conquer** |
| Path is predefinable | a **workflow** (chaining/routing/parallel/orchestrator) — NOT an agent |
| Plannable, no mid-task observation | **ReWOO** / **plan-and-solve** — not ReAct |
| Must observe env mid-task | **ReAct**; explore+backtrack → **tree-search/LATS** |
| Want to self-refine, NO oracle | STOP — get an external signal first, or use generate-then-select |
| Reasoning model available | try the **reasoning-effort knob** first (one call, higher effort) before hand-rolling loops; tier effort per stage |
| Long horizon (> ~5 rounds) | add **context-curation** (per-round distill) — mandatory, not optional |
| Task class recurs / prompt is a long-lived asset | **skill-library** / **prompt-optimization loop** |

### 3. Stack them

Real solutions combine primitives. Pick a composition template from [references/recipes.md](references/recipes.md), or build one using the stacking rules in [references/composition-guide.md](references/composition-guide.md). Canonical stack:

> fan-out find (map) → **loop-until-dry** (don't miss the tail) → **adversarial-verify** each finding (3 skeptics, majority-refute drops it) → **judge-panel/aggregator** synthesizes.

### 4. Emit the composed artifact

Emit a **markdown prompt/spec** — portable, copy-pastable; single-agent primitives (ReAct, plan-and-solve, CoVe) live *inside* one agent's reasoning, and multi-agent compositions are rendered as prose orchestration specs.

**The prompt text itself must meet the writing standard in [references/prompt-quality.md](references/prompt-quality.md)** — seven-part anatomy (role/stance, delimited inputs, procedure, output contract, stop condition, reasoned prohibitions, escape hatch), per-role rules (skeptics default-refute, judges reason-before-score with anchored rubrics, finders carry named lenses + failure scenarios), and the smell test. **Start from the skeleton at the end of that file** rather than free-writing the structure — it bakes in the runner manifest, the "data, not instructions" injection guard, and the closing stop-condition echo. A perfect composition rendered as vague prose still fails.

Either way, produce the actual artifact with these non-negotiables baked in:

- **Explicit stop condition** (oracle green / K dry rounds / N budget / score ≥ X) — never an open-ended "keep improving".
- **If any refine loop has no oracle**, attach one (tool/test/retrieval) or state plainly that it's capped at fixed rounds and why.
- **Diversity over scale** where you fan out (different angles/roles/lenses, not N identical samples).
- **Fail-closed cost bounds** (a missing range/budget arg must abort, not silently run everything).
- **Contracted joints** in multi-stage specs — each boundary names the artifact passed and the empty/null/unparseable failure path.
- A short **"why this composition"** note naming each primitive used and the diagnosis that selected it.

### 5. Apply the guardrails (always)

Check the composed prompt against [references/composition-guide.md](references/composition-guide.md#anti-patterns). The five that catch most mistakes:

1. **Oracle > self-eval** — if an oracle exists, the loop must use it, not the model's opinion.
2. **No naive self-refine** — refine loops without external signal degenerate; don't ship one.
3. **Workflow > agent** — if the path can be hardcoded, do that (predictable, testable, cheap).
4. **Diversity > scale** — heterogeneous strategies beat re-sampling one model.
5. **Don't debate for the sake of it** — plain self-consistency often beats multi-agent debate; add debate only with asymmetry (confidence-weighting / oracle-locking / role difference).

## Deep dives

Each primitive has a full example (prompt + why-it-works + pitfalls) in this repo:

- Index of all 43 techniques: [../../index.md](../../index.md)
- Core families (loop / generate-then-select / adversarial / decompose / search): [../../examples/](../../examples/)
- Frontier patterns (MoA, ReWOO, CoVe, agentic-RAG, rubric-judge, …): [../../examples/frontier/](../../examples/frontier/)
- Meta / system-level (reasoning-effort, skill-library, prompt-optimization, context-curation): examples 40–43 in [../../examples/frontier/](../../examples/frontier/)
- Underlying theory + sources: [../../agentic-loop.md](../../agentic-loop.md)

When the user asks "why does technique X work" or wants to go deeper on one primitive, read its example file rather than re-deriving.
