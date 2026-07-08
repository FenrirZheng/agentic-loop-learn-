---
description: Compose a robust agentic prompt/orchestration for a task by selecting and stacking control-structure primitives.
argument-hint: <describe the task you want an agentic prompt for>
---

You are composing an agentic prompt. Use the **agentic-prompt-composer** skill's method
(`skills/agentic-prompt-composer/SKILL.md` in this plugin) — invoke it if not already active.

Task from the user:

$ARGUMENTS

Do this:

1. **Diagnose** the task on the seven axes (oracle? solution-space width? failure fear? cost/volume? size? interaction? horizon & recurrence?). If a load-bearing axis is genuinely unknowable from the task text, ask at most 2 tight questions; otherwise infer and state your assumptions.
2. **Select** primitives via the catalog decision tree (`skills/agentic-prompt-composer/references/catalog.md`).
3. **Stack** them — pick or adapt a recipe (`skills/agentic-prompt-composer/references/recipes.md`).
4. **Emit** the finished artifact in a fenced block, with an explicit stop condition and (if any refine loop) a grounded oracle or a stated round cap. Emit a copy-pastable markdown prompt/spec; do not emit a runnable Workflow `.mjs` script unless the user explicitly asked for one (only then follow `skills/agentic-prompt-composer/references/workflow-mjs.md`). Write the prompt text to the standard in `skills/agentic-prompt-composer/references/prompt-quality.md` and run its smell test.
5. **Justify**: a short "why this composition" note naming each primitive and the diagnosis that chose it, then run the guardrail self-check (`skills/agentic-prompt-composer/references/composition-guide.md`).

If no task was provided in the arguments, ask the user for the task in one line, then proceed.
