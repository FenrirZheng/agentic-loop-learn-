# Emitting a Workflow .mjs script

How to render a composed orchestration as a **runnable Claude Code Workflow script** instead of a markdown spec. The payoff: the guardrails in step 4 of [SKILL.md](../SKILL.md) stop being "text the downstream LLM should obey" and become **executable, reviewable code** — a wrong stop condition is visible in a `while` guard, not buried in prose.

## When to choose this mode (and when not)

Emit `.mjs` when ALL of these hold:

- The target runtime is **Claude Code** (the user will run it there, via the `Workflow` tool or a saved workflow).
- The composition is **multi-agent orchestration** — fan-out, loops over agents, verification stages. Control flow lives *between* agents.
- The control path is **predefinable** (which is exactly when the catalog says "workflow, not agent").

Stay with the markdown prompt/spec when ANY of these hold:

- **Single-agent primitives dominate.** ReAct, plan-and-solve, CoVe, self-refine-with-oracle happen *inside one agent's reasoning loop* — an mjs script can only put those instructions into an `agent()` prompt string; the interesting control flow isn't at script level. A markdown prompt is the honest artifact.
- **Target isn't Claude Code** — LangGraph, an API pipeline, a prompt for humans or another platform.
- **The ask is prompt strengthening**, not orchestration ("make this prompt stop hallucinating") — the product is text.

Hybrid is normal: the mjs script is the skeleton, and each `agent()` prompt embeds the single-agent primitives (a verifier agent's prompt contains the adversarial default-refute framing; a solver agent's prompt contains plan-and-solve).

## Primitive → construct mapping

| Catalog primitive | Workflow construct |
|---|---|
| map-reduce / fan-out | `pipeline(items, ...)` (default) or `parallel(items.map(...))` (only when a stage needs ALL prior results — dedup, early-exit, cross-compare) |
| loop-until-dry | `while (dry < K)` + a `seen` Set keyed on stable identity; dedup against `seen`, not against survivors |
| until-oracle-passes | `while` loop: `agent()` produces → run the oracle → feed real failure output back; `schema` opt is itself a cheap structural oracle |
| adversarial-verify | N `agent()` skeptics per finding, prompt says "default to refuted unless proven", majority-refute drops it |
| best-of-N / mixture-of-agents | N generator `agent()` calls with **different** angle/strategy prompts → judge/aggregator agent(s) |
| self-consistency | N identical-prompt `agent()` calls with `schema` forcing `FINAL` + `CONFIDENCE` → vote in plain code, not in an agent |
| judge-panel / rubric-judge | parallel judge agents scoring against an anchored rubric → synthesis in code or one aggregator agent |
| cascade-routing | first `agent(..., {model: 'haiku', effort: 'low'})`, gate on confidence/agreement in code, escalate to a stronger call |
| completeness-critic | one final `agent()` whose findings become the next round's work-list |
| subagent context isolation | free — every `agent()` is an isolated context returning only its conclusion |
| plan-then-execute | phase 1 planner agent returns a structured plan (`schema`), script iterates its steps |

Aggregation, voting, dedup, and gating are **plain JavaScript**, not extra agents — code is the most objective judge available; don't pay an LLM to count votes.

## Script skeleton

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

// Fail-closed cost bound: a missing scope arg ABORTS, never defaults to "everything".
if (args?.target == null) throw new Error('args.target required (path or scope to analyze)')
log(`scope: ${args.target}`)   // verify the resolved bound before the long run

const FINDINGS = { type: 'object', properties: { findings: { type: 'array', items: {
  type: 'object', properties: { file: {type:'string'}, line: {type:'number'}, summary: {type:'string'} },
  required: ['file', 'summary'] } } }, required: ['findings'] }
const VERDICT = { type: 'object', properties: { refuted: {type:'boolean'}, reason: {type:'string'} },
  required: ['refuted'] }

const LENSES = ['concurrency', 'boundaries', 'error-handling', 'resource-leaks', 'auth']
const seen = new Set(), confirmed = []
const key = f => `${f.file}:${f.line}:${f.summary}`

let dry = 0
while (dry < 2) {                                    // explicit stop: K dry rounds
  if (budget.total && budget.remaining() < 50_000) { log('budget floor hit, stopping'); break }
  phase('Find')
  const found = (await parallel(LENSES.map(lens => () =>
    agent(`Find ${lens} issues in ${args.target}. Report only NEW issues not in: ${[...seen].join('; ') || '(none)'}`,
      { label: `find:${lens}`, phase: 'Find', schema: FINDINGS })
  ))).filter(Boolean).flatMap(r => r.findings)
  const fresh = found.filter(f => !seen.has(key(f)))
  if (!fresh.length) { dry++; continue }
  dry = 0; fresh.forEach(f => seen.add(key(f)))      // dedup vs seen, NOT vs confirmed

  const judged = await parallel(fresh.map(f => () =>
    parallel([0,1,2].map(i =>
      () => agent(`You are skeptic #${i+1}. Default: this is a FALSE ALARM unless you can prove it real by reading the code: ${JSON.stringify(f)}`,
        { label: `verify:${f.file}`, phase: 'Verify', schema: VERDICT })
    )).then(vs => ({ f, real: vs.filter(Boolean).filter(v => !v.refuted).length >= 2 }))
  ))
  confirmed.push(...judged.filter(j => j.real).map(j => j.f))
  log(`round done: ${fresh.length} fresh, ${confirmed.length} confirmed total`)
}

phase('Synthesize')
return { confirmed, rounds_dry: dry, scanned_scope: args.target }
```

## Hard runtime rules (violating these breaks the script)

- **`meta` must be a pure literal** — no variables, spreads, calls, or template strings inside it. `name` + `description` required.
- **Plain JavaScript, not TypeScript** — type annotations fail to parse.
- **No `Date.now()`, `Math.random()`, argless `new Date()`** — they throw (they'd break resume). Pass timestamps in via `args`; get diversity by varying prompts per index, not by RNG.
- **`agent()` can return `null`** (user skip / terminal error) — `.filter(Boolean)` before use; `parallel()` never rejects, failed thunks become `null`.
- **`pipeline()` is the default** for multi-stage work; a `parallel()` barrier between stages is justified only when stage N genuinely needs ALL of stage N−1 (dedup across the full set, early-exit on zero, cross-item comparison).
- **Concurrency is capped** (~10 live agents; queue handles the rest) and total agents capped at 1000 — design rounds, don't design for unbounded fan-out.
- **`budget`**: guard dynamic loops with `budget.total && budget.remaining() > floor`; without the `budget.total` check, `remaining()` is `Infinity` and the loop runs to the agent cap.

## Guardrails → code (the step-4 non-negotiables, compiled)

| SKILL.md non-negotiable | mjs form |
|---|---|
| Explicit stop condition | the literal `while` guard: oracle-green flag, `dry < K`, round/budget cap — never `while (true)` with an agent deciding "good enough" |
| Refine loop needs an oracle | the loop body *runs* the oracle (test command, `schema` validation, retrieval check) and branches on its real output; if genuinely none, a hard round cap with a comment saying why |
| Diversity over scale | visibly different prompt strings/lenses/models across the fan-out — N copies of one prompt is the anti-pattern made greppable |
| Fail-closed cost bounds | `if (args?.x == null) throw` — never `args.x \|\| EVERYTHING`; `log()` the resolved scope first so the launch can be verified before walking away |
| "Why this composition" note | still prose — put it in a comment block at the top of the script and repeat it when handing the script to the user |
| No silent truncation | every cap (top-N, sampling, dropped slices) gets a `log()` naming what was dropped |

## Emission checklist

Before handing over the script, verify:

1. `meta` is a pure literal and phase titles match the `phase()` calls.
2. Every loop's stop condition is code, and it's the most objective one available.
3. Scope/cost args fail closed and the resolved scope is `log()`ged on line one of the body.
4. Fan-outs are genuinely diverse (different prompts, not clones) — unless self-consistency, where identical prompts are the point and the vote happens in code.
5. All `agent()` results are `.filter(Boolean)`ed before use.
6. No `Date.now()` / `Math.random()` / TS syntax snuck in.
7. Single-agent primitives the composition needs (ReAct, CoVe, plan-and-solve…) are embedded in the relevant `agent()` prompt strings — the script alone isn't the whole composition.
8. Tell the user how to run it: paste as `Workflow({script})`, or save and use `Workflow({scriptPath, args})`.
