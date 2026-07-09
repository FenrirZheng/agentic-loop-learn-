---
description: Compose a robust agentic prompt/orchestration for a task by selecting and stacking control-structure primitives; saves the prompt and its rationale as two markdown files under ./prompts/.
argument-hint: <describe the task you want an agentic prompt for>
---

You are composing an agentic prompt. Use the **agentic-prompt-composer** skill's method
(`${CLAUDE_PLUGIN_ROOT}/skills/agentic-prompt-composer/SKILL.md`) — invoke it if not already active.

Task from the user:

$ARGUMENTS

Do this:

1. **Diagnose** the task on the eight axes (oracle? solution-space width? failure fear? cost/volume? size? interaction? horizon & recurrence? edges — ambiguity / irreversible actions / untrusted content + tools / abstain-vs-wrong?). If a load-bearing axis is genuinely unknowable from the task text, ask at most 2 tight questions; otherwise infer and state your assumptions.
2. **Select** primitives via the catalog decision tree (`${CLAUDE_PLUGIN_ROOT}/skills/agentic-prompt-composer/references/catalog.md`).
3. **Stack** them — pick or adapt a recipe (`${CLAUDE_PLUGIN_ROOT}/skills/agentic-prompt-composer/references/recipes.md`).
4. **Emit** the finished artifact — a copy-pastable markdown prompt/spec — in a fenced block, with an explicit stop condition and (if any refine loop) a grounded oracle or a stated round cap. Write the prompt text to the standard in `${CLAUDE_PLUGIN_ROOT}/skills/agentic-prompt-composer/references/prompt-quality.md` and run its smell test.
5. **Justify**: a short "why this composition" note naming each primitive and the diagnosis that chose it, then run the guardrail self-check (`${CLAUDE_PLUGIN_ROOT}/skills/agentic-prompt-composer/references/composition-guide.md`).
6. **Save** both artifacts as two markdown files under `./prompts/` in the current working directory (create the directory if it doesn't exist):
   - `./prompts/<slug>.prompt.md` — the **pure prompt**: exactly the copy-pastable artifact from step 4 and nothing else — no frontmatter, no commentary, no fence markers. Purity is the point: this file is what gets pasted into an agent verbatim.
   - `./prompts/<slug>.why.md` — the **why**: the step-1 diagnosis (the eight axes and what you inferred), each primitive chosen and the diagnosis line that selected it, the recipe/stacking decision, and the step-5 guardrail self-check result. Open with a link to the sibling: `Prompt file: [<slug>.prompt.md](<slug>.prompt.md)`. (The prompt file carries no backlink — that would break its copy-paste purity.)

   `<slug>` is a short kebab-case name derived from the task (e.g. `flaky-test-triage`). If either file already exists, suffix both with `-2`, `-3`, … — never overwrite an existing pair. After saving, report both paths to the user.

If no task was provided in the arguments, ask the user for the task in one line, then proceed.
