# Emitting a Workflow .mjs script

How to render a composed orchestration as a **runnable Claude Code Workflow script** instead of a markdown spec. The payoff: the guardrails in step 4 of [SKILL.md](../SKILL.md) stop being "text the downstream LLM should obey" and become **executable, reviewable code** — a wrong stop condition is visible in a `while` guard, not buried in prose.

## When to choose this mode (and when not)

Emit `.mjs` when ALL of these hold:

- The target runtime is **Claude Code** (the user will run it there, via the `Workflow` tool or a saved workflow).
- The composition is **multi-agent orchestration** — fan-out, loops over agents, verification stages. Control flow lives *between* agents.
- The control path is **predefinable** (which is exactly when the catalog says "workflow, not agent").

Stay with the markdown prompt/spec when ANY of these hold:

- **Single-agent primitives dominate.** ReAct, plan-and-solve, CoVe-as-reasoning, self-refine-with-oracle happen *inside one agent's reasoning loop* — an mjs script can only put those instructions into an `agent()` prompt string; the interesting control flow isn't at script level. A markdown prompt is the honest artifact.
- **Target isn't Claude Code** — LangGraph, an API pipeline, a prompt for humans or another platform.
- **The ask is prompt strengthening**, not orchestration ("make this prompt stop hallucinating") — the product is text.

Hybrid is normal and usually optimal: the mjs script is the skeleton; each `agent()` prompt embeds the single-agent primitives, written to the standard in [prompt-quality.md](prompt-quality.md) (a verifier agent's prompt carries the default-refute stance; a solver agent's prompt carries plan-and-solve; a judge agent's prompt carries the anchored rubric).

## The one structural fact that shapes everything

**The script sandbox has no filesystem, no shell, no network.** Only agents touch the world. Consequences:

- **Oracle-in-agent**: the script cannot run pytest. An agent runs it and returns structured results — `agent('Run `pytest -x`. Return {green, first_failure} — first_failure is the REAL pasted output, not a summary.', {schema: ORACLE})` — and the *script* branches on `.green`. The oracle's objectivity survives because the branch is code; the agent is only the transport.
- **Judge-in-code**: counting votes, deduping, gating on confidence, picking a tournament winner from scores — plain JavaScript, never an extra agent. Code is the most objective judge available; don't pay an LLM to count.
- **State lives in script variables** (`seen` Sets, `confirmed` arrays, round counters) — not in any agent's memory. Agents are stateless one-shot calls; anything the next stage needs must be passed in its prompt.

## Primitive → construct mapping

| Catalog primitive | Workflow construct |
|---|---|
| map-reduce / fan-out | `pipeline(items, ...)` (default) or `parallel(items.map(...))` (only when a stage needs ALL prior results — dedup, early-exit, cross-compare) |
| pipeline | `pipeline(items, stage1, stage2, ...)` — no barrier; item A can be in stage 3 while B is in stage 1 |
| loop-until-dry | `while (dry < K)` + a `seen` Set keyed on stable identity; dedup against `seen`, not against survivors |
| until-oracle-passes | `while` loop: producer `agent()` → **oracle-in-agent** returns structured pass/fail → feed the REAL failure output into the next producer prompt; `schema` opt is itself a free structural oracle |
| until-budget | `while (budget.total && budget.remaining() > floor)` — the budget object is the fuse |
| until-convergence | loop comparing this round's output to last round's in code (string/field equality), hard round cap alongside — convergence ≠ correctness |
| adversarial-verify | N `agent()` skeptics per finding, each prompt naming a distinct lens + default-refute stance; majority-refute drops it (count in code) |
| best-of-N / mixture-of-agents | N generator `agent()` calls with **different** strategy prompts → judge/aggregator agent; MoA = second layer of agents each reading ALL first-layer outputs (a genuine `parallel` barrier) before one aggregator fuses |
| self-consistency | N identical-prompt `agent()` calls with `schema` forcing `{final, confidence}` → tally in plain code (weighted vote = sum confidence per distinct `final`) |
| universal self-consistency | N generators → ONE picker agent that reads all candidates and picks the most mutually-consistent (open-ended answers where code can't compare) |
| tournament | pairwise judge agents; bracket bookkeeping in code; run each pair BOTH orders (A-B, B-A) and advance only consistent wins — position bias is real |
| judge-panel / rubric-judge | parallel judge agents, each scoring one dimension against an anchored rubric (`schema` with per-criterion scores + cited evidence) → weighted merge in code |
| evaluator-optimizer | two-agent loop: generator `agent()` → evaluator `agent()` returns `{score, concrete_fixes[]}` → feed fixes verbatim into next generator prompt; stop on score ≥ X or round cap; keep generator and evaluator prompts separate (never one agent doing both) |
| chain-of-verification | after a draft: one agent generates verification questions (`schema: {questions[]}`) → fan out one checker agent per question → patch agent fixes ONLY the first failed claim |
| escalation-ladder / cascade-routing | first `agent(..., {model: 'haiku', effort: 'low'})` ×3 → gate on agreement in code → escalate disagreements to `{model: 'opus', effort: 'high'}`. The `model`/`effort` opts ARE the ladder |
| reasoning-effort knob | `opts.effort: 'low'…'max'` per call — spend thinking where the composition needs it (hardest verify/judge stages), not everywhere; often replaces a hand-rolled refine loop entirely |
| hill-climbing | keep `best` + `bestScore` in code; each round's candidate is scored (oracle or rubric agent); replace only on strictly better; stop after K non-improving rounds |
| tree-search (bounded) | beam in code: expand M candidate next-steps per node (M agents), score each, keep top-B, iterate to depth D — caps written as constants; full LATS/MCTS usually belongs inside one ReAct-ish agent instead |
| agentic-RAG | loop: retriever agent → grader agent scores hits (`schema: {relevant: bool, why}`) → if weak, query-rewrite agent (≤3×) → answer agent with graded evidence inline |
| skeleton-of-thought | outline agent returns `{sections[]}` → `pipeline(sections, expandAgent)` → merge agent (or merge in code if mechanical) |
| completeness-critic | one final `agent()` whose findings become the next round's work-list (feeds the `while` loop) |
| guardrail feedback loop | pre/post checks as plain code (regex, schema validation, length caps) around each agent call; on failure, re-call the agent with the specific violation quoted |
| subagent context isolation | free — every `agent()` is an isolated context returning only its conclusion |
| plan-then-execute | phase 1 planner agent returns a structured plan (`schema`), script iterates its steps |
| prompt-optimization loop (OPRO-lite) | candidates = prompt strings; oracle = a frozen eval set run by grader agents; loop proposes new candidates from the scored history; see [example 42](../../../examples/frontier/42-prompt-optimization-loop.md) |
| skill-library accumulation | at run end, one agent distills oracle-passed artifacts into a reusable recipe file the NEXT run receives via `args`; see [example 41](../../../examples/frontier/41-skill-library.md) |

Aggregation, voting, dedup, gating, bracket bookkeeping are **plain JavaScript**, not extra agents.

## Cost & quality tiering idioms

- **`model` / `effort` per call**: mechanical stages (formatting, extraction, dedup-assist) → `{model: 'haiku', effort: 'low'}`; generation → session default (omit); the hardest verify/judge stages → `{effort: 'high'}` or `'xhigh'`. Omit `model` when unsure — inheriting the session model is almost always right.
- **`agentType`**: reuse a purpose-built subagent (`{agentType: 'code-reviewer'}`) instead of re-explaining its whole role in the prompt; composes with `schema`.
- **`isolation: 'worktree'`**: ONLY when agents mutate files in parallel and would conflict — it's expensive (~200–500ms + disk each). Read-only fan-outs never need it.
- **`workflow(nameOrRef, args)`**: compose a saved/previous workflow as one stage of a bigger one (single nesting level). Prefer this over inlining a second copy of a stack you already ship.
- **Every `agent()` can return `null`** (user skip / terminal error) — `.filter(Boolean)` before use; `parallel()` never rejects (failed thunks → `null`); a throwing `pipeline` stage drops that item to `null`.

## Schema design (the free structural oracle)

- **Small and flat.** A verdict is `{refuted: boolean, reason: string}` — not a nested report. Big schemas invite filler; small ones are checkable.
- **Enums for verdicts**: `{"enum": ["confirmed", "refuted", "unverifiable"]}` beats free text — and `"unverifiable"` is the escape hatch that prevents forced hallucination (see [prompt-quality.md](prompt-quality.md)).
- **Force the evidence fields**: findings carry `file`, `line`, `summary`, `failure_scenario` as `required` — a finding schema without required evidence fields collects opinions.
- **Confidence as number 0–1** so code can gate/weight it.
- Schema validates **structure, not truth** — a well-formed wrong answer passes. Pair it with adversarial-verify when plausible-but-wrong is the fear.

## Skeleton A — loop-until-dry + adversarial-verify (discovery-shaped)

```js
export const meta = {
  name: 'kebab-case-name',
  description: 'One line: what it does',
  phases: [
    { title: 'Find', detail: 'fan-out finders, loop until dry' },
    { title: 'Verify', detail: '3 skeptics per finding' },
    { title: 'Synthesize' },
  ],
}

// WHY THIS COMPOSITION: no free oracle for "is this finding real";
// fear = both missing the tail (loop-until-dry) and false positives (adversarial-verify).
// Fail-closed cost bound: a missing scope arg ABORTS, never defaults to "everything".
if (args?.target == null) throw new Error('args.target required (path or scope to analyze)')
log(`scope: ${args.target}`)   // verify the resolved bound before the long run

const FINDINGS = { type: 'object', properties: { findings: { type: 'array', items: {
  type: 'object', properties: { file: {type:'string'}, line: {type:'number'},
    summary: {type:'string'}, failure_scenario: {type:'string'} },
  required: ['file', 'summary', 'failure_scenario'] } } }, required: ['findings'] }
const VERDICT = { type: 'object', properties: {
  verdict: { enum: ['confirmed', 'refuted', 'unverifiable'] }, reason: {type:'string'} },
  required: ['verdict', 'reason'] }

// Diversity is written per-prompt, not implied: each lens says what it IGNORES.
const LENSES = [
  ['concurrency',   'races, deadlocks, unsynchronized shared state — ignore style and naming'],
  ['boundaries',    'off-by-one, empty/max inputs, overflow — ignore architecture'],
  ['error-handling','swallowed errors, wrong fallbacks, partial failure — ignore performance'],
  ['resources',     'leaks, double-close, unbounded growth — ignore readability'],
]
const seen = new Set(), confirmed = []
const key = f => `${f.file}:${f.line}:${f.summary}`

let dry = 0, round = 0
while (dry < 2 && round < 6) {                       // explicit stop: K dry rounds + hard cap
  round++
  if (budget.total && budget.remaining() < 50_000) { log('budget floor hit, stopping'); break }
  const found = (await parallel(LENSES.map(([lens, brief]) => () =>
    agent(`You are a ${lens} auditor of ${args.target}: ${brief}.
Assume bugs exist that you have not found yet.
Report ONLY findings not in this seen-list: ${[...seen].join('; ') || '(none)'}
Every finding needs a concrete failure_scenario (inputs -> wrong behavior); no scenario, no finding.`,
      { label: `find:${lens}`, phase: 'Find', schema: FINDINGS })
  ))).filter(Boolean).flatMap(r => r.findings)
  const fresh = found.filter(f => !seen.has(key(f)))
  if (!fresh.length) { dry++; continue }
  dry = 0; fresh.forEach(f => seen.add(key(f)))      // dedup vs seen, NOT vs confirmed

  const judged = await parallel(fresh.map(f => () =>
    parallel([
      ['reproduce', 'can this failure_scenario actually occur with real inputs?'],
      ['code-path', 'does the code actually reach this state? read it.'],
      ['spec',      'is this actually wrong per the intended behavior, or by design?'],
    ].map(([lens, q]) => () =>
      agent(`You are a skeptic (lens: ${lens}). DEFAULT VERDICT: refuted.
Confirm ONLY with file:line evidence. Question: ${q}
If you cannot decide, verdict = unverifiable (counts as refuted).
Finding: ${JSON.stringify(f)}`,
        { label: `verify:${f.file}`, phase: 'Verify', schema: VERDICT })
    )).then(vs => ({ f, real: vs.filter(Boolean).filter(v => v.verdict === 'confirmed').length >= 2 }))
  ))
  confirmed.push(...judged.filter(j => j.real).map(j => j.f))
  log(`round ${round}: ${fresh.length} fresh, ${confirmed.length} confirmed total`)
}

phase('Synthesize')
return { confirmed, rounds: round, scanned_scope: args.target }
```

## Skeleton B — until-oracle-passes via oracle-in-agent (build-shaped)

```js
export const meta = {
  name: 'implement-until-green',
  description: 'Plan, implement stepwise, loop each step until the real test suite passes',
  phases: [{ title: 'Plan' }, { title: 'Build', detail: 'implement -> oracle -> fix, per step' }, { title: 'Red-team' }],
}

// WHY THIS COMPOSITION: pytest exists => until-oracle-passes is the only loop worth having;
// plan-then-execute stops skip-steps; devil's-advocate catches "green but wrong design".
if (args?.task == null) throw new Error('args.task required (what to implement)')
if (args?.testCmd == null) throw new Error('args.testCmd required (fail closed: no default oracle)')
log(`task: ${args.task} | oracle: ${args.testCmd}`)

const PLAN = { type: 'object', properties: { steps: { type: 'array', items: {
  type: 'object', properties: { what: {type:'string'}, acceptance: {type:'string'} },
  required: ['what', 'acceptance'] } } }, required: ['steps'] }
const ORACLE = { type: 'object', properties: { green: {type:'boolean'},
  first_failure: {type:'string'} }, required: ['green'] }

phase('Plan')
const plan = await agent(
  `Plan the implementation of: ${args.task}. Steps small enough that each is testable.
Do NOT write code yet.`, { schema: PLAN })
if (!plan?.steps?.length) throw new Error('planner returned no steps')
log(`plan: ${plan.steps.length} steps`)

phase('Build')
for (const [i, step] of plan.steps.entries()) {
  let failure = null
  let green = false
  for (let attempt = 1; attempt <= 5; attempt++) {   // hard cap per step, stated
    await agent(`Implement step ${i + 1}/${plan.steps.length}: ${step.what}
Acceptance: ${step.acceptance}
${failure ? `Previous attempt failed. REAL oracle output (fix the FIRST failure only):\n${failure}` : ''}
Never claim done without running ${args.testCmd} yourself first.`,
      { label: `build:${i + 1}.${attempt}`, phase: 'Build' })
    const check = await agent(
      `Run \`${args.testCmd}\`. Return green + first_failure = the REAL pasted output of the first
failing test (verbatim, not summarized). If everything passes, green = true.`,
      { label: `oracle:${i + 1}.${attempt}`, phase: 'Build', schema: ORACLE })
    if (check?.green) { green = true; break }
    failure = check?.first_failure ?? '(oracle agent failed — treat as red)'
  }
  if (!green) return { done: false, stuck_at: step.what, last_failure: failure } // honest exit, not silent continue
}

phase('Red-team')
const critique = await agent(
  `Red-team the diff for "${args.task}": give 2 concrete scenarios where it breaks
despite green tests (wrong design, missing case, API misuse). If none, say NONE with reasons.`,
  { effort: 'high' })
return { done: true, steps: plan.steps.length, redteam: critique }
```

## Hard runtime rules (violating these breaks the script)

- **`meta` must be a pure literal** — no variables, spreads, calls, or template strings inside it. `name` + `description` required; phase titles must match `phase()` calls exactly.
- **Plain JavaScript, not TypeScript** — type annotations fail to parse.
- **No `Date.now()`, `Math.random()`, argless `new Date()`** — they throw (they'd break resume). Pass timestamps in via `args`; get diversity by varying prompts per index, not by RNG.
- **`agent()` can return `null`** — `.filter(Boolean)` before use; `parallel()` never rejects; a throwing pipeline stage nulls that item.
- **`pipeline()` is the default** for multi-stage work; a `parallel()` barrier between stages is justified only when stage N genuinely needs ALL of stage N−1 (dedup across the full set, early-exit on zero, cross-item comparison, MoA layers).
- **Inside `pipeline()`/`parallel()` stages, set `opts.phase` explicitly** — the global `phase()` state races across concurrent items.
- **Pipeline stage callbacks receive `(prevResult, originalItem, index)`** — use `originalItem`/`index` to label later stages instead of threading context through every return value.
- **Concurrency is capped** (~10–16 live agents; the queue handles the rest); lifetime cap 1000 agents; one `pipeline`/`parallel` call takes ≤4096 items. Design rounds, not unbounded fan-out.
- **`budget`**: guard dynamic loops with `budget.total && budget.remaining() > floor`; without the `budget.total` check, `remaining()` is `Infinity` and the loop runs to the agent cap.
- **`args` arrive as real JSON values** — but treat them as hostile anyway: `args?.x == null → throw`. Never `args.x || EVERYTHING`.
- **Resume exists**: same script + args replays cached `agent()` results. Debug an odd result by reading the run's `journal.jsonl` before assuming agents returned nothing.

## Guardrails → code (the step-4 non-negotiables, compiled)

| SKILL.md non-negotiable | mjs form |
|---|---|
| Explicit stop condition | the literal `while` guard: oracle-green flag, `dry < K`, round/budget cap — never `while (true)` with an agent deciding "good enough" |
| Refine loop needs an oracle | the loop body *runs* the oracle via oracle-in-agent and branches on its structured output; if genuinely none, a hard round cap with a comment saying why |
| Diversity over scale | visibly different prompt strings/lenses/models across the fan-out — N copies of one prompt is the anti-pattern made greppable |
| Fail-closed cost bounds | `if (args?.x == null) throw` — never `args.x \|\| EVERYTHING`; `log()` the resolved scope first so the launch can be verified before walking away |
| "Why this composition" note | a comment block at the top of the script naming each primitive + the diagnosis that chose it; repeat it when handing the script over |
| No silent truncation | every cap (top-N, sampling, dropped slices, stuck steps) gets a `log()` or an honest field in the return value naming what was dropped |

## Emission checklist

Before handing over the script, verify:

1. `meta` is a pure literal and phase titles match the `phase()` calls.
2. Every loop's stop condition is code, and it's the most objective one available — plus a hard cap even on oracle loops (oracles can stay red forever).
3. Scope/cost args fail closed and the resolved scope is `log()`ged on line one of the body.
4. Fan-outs are genuinely diverse (each prompt names its lens and what it ignores) — unless self-consistency, where identical prompts are the point and the vote happens in code.
5. Every `agent()` prompt passes the [prompt-quality.md](prompt-quality.md) smell test: self-contained, output contract, stance, escape hatch (`unverifiable` in verdict enums).
6. All `agent()` results are `.filter(Boolean)`ed / null-checked before use.
7. Votes, gates, dedup, brackets are plain code — no agent is counting.
8. `model`/`effort` tiering: cheap where mechanical, high effort only on the hardest verify/judge calls, omitted elsewhere.
9. No `Date.now()` / `Math.random()` / TS syntax snuck in.
10. Single-agent primitives the composition needs (ReAct, CoVe, plan-and-solve…) are embedded in the relevant `agent()` prompt strings — the script alone isn't the whole composition.
11. The return value reports honest failure states (`stuck_at`, dropped scope) — never a success-shaped object on a failed run.
12. Tell the user how to run it: paste as `Workflow({script})`, or save and use `Workflow({scriptPath, args})`.
