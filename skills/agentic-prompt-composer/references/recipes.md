# Composition Recipes

Ready-to-adapt stacks. Each says *when*, *the stack*, and *why the stack (not a single primitive)*. Pick the closest, then fill the placeholders.

---

## R1 · Exhaustive bug / risk hunt (high recall + high precision)

**When**: "find all the bugs / security issues / edge cases", and both missing-something and false-positives are costly.

**Stack**: fan-out finders (map) → **loop-until-dry** → **adversarial-verify** each finding → **judge-panel** synthesize.

```text
Round loop (stop when 2 consecutive rounds add nothing new):
  1. FAN-OUT: run several finders in parallel, each with a DIFFERENT lens
     (concurrency, boundaries, error-handling, resource leaks, auth).
     Maintain SEEN keyed by file:line:summary; report only NEW findings.
  2. VERIFY each NEW finding with 3 independent skeptics, each told:
     "default this is a false alarm unless you can prove it real." Drop if >=2 refute.
  3. Add survivors to SEEN.
Then AGGREGATE survivors: dedup, rank by severity, mark Must-fix / Nice-to-have.
```

**Why the stack**: loop-until-dry beats `find N` (the tail hides sparse bugs); diverse lenses beat re-running one finder (diversity > scale); adversarial-verify kills plausible-but-wrong findings; the aggregator merges better than a ranker picks. See [loop-until-dry](../../../examples/04-loop-until-dry.md), [adversarial-verify](../../../examples/12-adversarial-verify.md).

---

## R2 · Reliable code change (must actually work)

**When**: implement/fix code where tests or a compiler exist.

**Stack**: **plan-then-execute** → **until-oracle-passes** (stepwise) → **devil's-advocate** on the diff.

```text
1. PLAN: list steps + acceptance criteria; do not code yet.
2. For each step: implement, then RUN the oracle (pytest / build / typecheck);
   paste the REAL output; fix only the first real failure; repeat until green.
3. Before done: red-team the diff — "give 2 concrete scenarios where this breaks."
   Address or explicitly dismiss each.
Never claim done without real green output.
```

**Why the stack**: planning stops skip-steps; the oracle (not self-opinion) is the stop condition — the only safe refine loop; devil's-advocate catches the "passes tests but wrong design" gap. See [until-oracle-passes](../../../examples/05-until-oracle-passes.md), [plan-then-execute](../../../examples/15-plan-then-execute.md).

---

## R3 · Hard reasoning / math (unique answer, no external oracle)

**When**: a single correct answer, no test to run.

**Stack**: **plan-and-solve** (or **least-to-most** if there's a dependency chain) → **self-consistency** (N independent runs) → **confidence-weighted vote**.

```text
Run N independent solutions, each: plan the steps, then solve step-by-step
(never skip a step), end with `FINAL: <answer>` and `CONFIDENCE: <0-1>`.
Then tally: group identical FINALs, weight by summed confidence, output the top;
mark [LOW CONFIDENCE] if the lead is thin.
```

**Why the stack**: no oracle → generate-then-select instead of self-refine (which would degenerate); voting cancels scattered errors; plan-and-solve reduces per-run mistakes. Don't add debate — it usually loses to plain self-consistency here. See [self-consistency](../../../examples/06-self-consistency.md), [plan-and-solve](../../../examples/frontier/30-plan-and-solve.md).

---

## R4 · Open-ended generation (design / writing / naming)

**When**: wide solution space, quality judgeable but no unique answer.

**Stack**: **best-of-N** (diverse candidates) → **rubric-based judge** → optional **mixture-of-agents** synthesis.

```text
1. Generate N candidates that are DELIBERATELY different (vary angle/style/strategy).
2. Score each with an anchored rubric (define what 0 vs max looks like per criterion);
   require per-item reasons pointing to specifics.
3. Either take the winner, or (MoA) have an aggregator fuse the best parts of the top few.
```

**Why the stack**: diversity makes selection meaningful; anchored rubric turns fuzzy quality into a semi-objective checklist; MoA aggregation beats picking one when strengths are scattered. See [best-of-N](../../../examples/07-best-of-n.md), [rubric-judge](../../../examples/frontier/33-rubric-based-judge.md).

---

## R5 · Research / question needing external knowledge

**When**: needs fresh or external facts; hallucination risk.

**Stack**: **agentic-RAG** (retrieve-grade-correct loop) → **chain-of-verification** → **completeness-critic**.

```text
1. Decide if retrieval is needed; if so retrieve, GRADE the hits, rewrite query if weak (<=3x).
2. Draft the answer citing which claim came from which source; "insufficient data" where none.
3. CoVe: generate verification questions for each key claim, check them, fix the first failure.
4. Completeness-critic: "what angle/source/claim is missing?" -> next round of retrieval.
```

**Why the stack**: retrieval-in-the-loop grounds the refine step (the external signal self-refine needs); CoVe localizes errors cheaply; the critic drives coverage. See [agentic-RAG](../../../examples/frontier/27-agentic-rag.md), [chain-of-verification](../../../examples/frontier/24-chain-of-verification.md).

---

## R6 · Large corpus processing (too big for one context)

**When**: summarize/analyze/extract over a doc or file-set that won't fit.

**Stack**: **map-reduce** (or **divide-and-conquer** if even the merge is too big) + **subagent context isolation**.

```text
MAP: slice input; each slice processed independently (parallel), returns only a
     structured digest (not raw text). Run heavy per-slice search in isolated subagents.
REDUCE: merge digests — dedup, flag cross-slice conflicts, preserve all key facts/numbers.
If the merged digest is still too big, recurse (summary of summaries).
Log any slices dropped for cost — never silently truncate.
```

**Why the stack**: slicing keeps each step's attention full; isolation stops context rot; recursion handles arbitrarily large inputs. See [map-reduce](../../../examples/16-map-reduce.md), [divide-and-conquer](../../../examples/18-divide-and-conquer.md).

---

## R7 · Cost-bounded high-volume serving

**When**: many requests, most easy, cost matters.

**Stack**: **cascade-routing** (agreement/confidence gate) → escalate hard ones to a stronger tier, then apply the fitting recipe above.

```text
TIER 1 (cheap): answer; self-report confidence, or run 3x and check agreement.
GATE: confident/agreed -> accept. Divergent/low-confidence -> escalate.
TIER 2 (strong): full method (may itself be R2/R3/R5).
```

**Why the stack**: spends expensive compute only where cheap models disagree — the signal that the item is actually hard. See [cascade-routing](../../../examples/frontier/36-cascade-routing.md).

---

## Stacking pattern summary

Most stacks are: **decompose** (if big) → **generate** (diverse) → **verify/select** (oracle if possible, else adversarial/rubric) → **synthesize**, wrapped in a **loop with an explicit stop condition**. When in doubt, that skeleton is the default.
