# Composition Guide — stacking rules & anti-patterns

How to combine primitives well, and the mistakes to refuse to ship.

## Stacking rules

1. **Decompose outermost, verify innermost.** If the task is big, split first (map-reduce / divide-and-conquer); apply the quality machinery (verify/select) to each piece, not the raw whole.
2. **Generate diverse, then select objective.** The generation axis wants variety (different angles/strategies/roles). The selection axis wants objectivity (oracle > rubric > relative comparison > raw self-opinion). Keep the two axes distinct — don't let one prompt both brainstorm and grade.
3. **Every loop needs an explicit, externally-checkable stop condition.** Rank of preference: oracle green > K dry rounds > score ≥ threshold (anchored rubric) > N budget. Open-ended "keep improving" is never acceptable.
4. **Attach an oracle to any refine loop, or cap it.** A critique→revise loop with no external signal degenerates; either ground the critique in a tool/test/retrieval result, or fix the round count and say why.
5. **Prefer a workflow to an agent.** If you can write the control path as fixed code (chaining, routing, parallel, orchestrator-workers), do that — predictable, testable, cheap. Reserve agents for when the next step genuinely depends on runtime observation.
6. **Isolate expensive subtasks.** Big searches/verifications go in a subagent that returns only its conclusion, keeping the main context clean (no context rot on long runs).
7. **Match verifier diversity to failure modes.** If a claim can fail in several ways, give each verifier a distinct lens (correctness / security / does-it-reproduce) instead of N identical skeptics.

## Anti-patterns

Refuse to emit a composition that commits any of these:

- **Self-eval where an oracle exists.** If tests/compile/schema can decide it, the loop must use them — not "does this look right?".
- **Naive self-refine (no external signal).** The single most common and most damaging mistake. Ground it or don't loop it.
- **Agent where a workflow suffices.** Adds unpredictability and cost for no benefit when the path is knowable.
- **N identical samples called "diversity".** Re-sampling one model at one setting is not diversity; vary the strategy/angle/role.
- **Debate for its own sake.** Two same-model agents arguing usually loses to plain self-consistency; only add debate with asymmetry (confidence-weighting, oracle-locking, real role/knowledge difference).
- **`find N` for open-ended discovery.** Fixed counts miss the long tail; use loop-until-dry.
- **Convergence treated as correctness.** Stable ≠ right; a convergence loop can lock onto a confidently wrong answer.
- **Cost/scope bound that fails open.** A missing range/budget arg must ABORT, not default to "run everything". Log the resolved scope before a long run.
- **Silent truncation.** If you cap coverage (top-N, no-retry, sampling), say what was dropped — silence reads as "covered everything".
- **Score/flag with no base-rate check.** A `needs_review`/`is_anomaly` flag that fires on nearly everything carries no signal; verify its trigger rate on real data.

## The three "prefer the plainer side" defaults

When unsure, bias toward the simpler option:

1. **Diversity > scale** — heterogeneous strategies beat more samples of one.
2. **Workflow > agent** — hardcode the path when you can.
3. **Process reward (PRM) > outcome reward (ORM)** — score each step when possible, not just the final result.

## Sanity self-check before emitting

- Does every loop have an explicit stop condition, and is it the most objective one available?
- If there's a refine loop, is it grounded in an external signal (or knowingly capped)?
- Where I fan out, is it genuinely diverse?
- Could this be a workflow instead of an agent?
- Could a single higher-effort call replace this hand-rolled loop (reasoning-effort knob), and is effort tiered per stage rather than raised globally?
- If the loop will run long (> ~5 rounds), is context curation (per-round distill / subagent isolation) built in?
- Are cost bounds fail-closed and is dropped scope logged?
- Does the emitted prompt text pass the [prompt-quality smell test](prompt-quality.md#emitted-prompt-smell-test) (output contract, stance, escape hatch, reasoned prohibitions)?
- Did I name each primitive used and the diagnosis that selected it?
