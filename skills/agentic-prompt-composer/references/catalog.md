# Technique Catalog — diagnosis → primitive

The full picker. Each row links to its deep-dive example (prompt + why-it-works + pitfalls). Paths are relative to this file.

## Decision tree (read top-down; first match wins, then stack)

1. **Objective oracle exists** (tests / compile / schema / linter / exact answer / retrieval truth)
   → [until-oracle-passes](../../../examples/05-until-oracle-passes.md). Stepwise verification → PRM-style (score each step, not just the end). *This is the most reliable loop and the only safe basis for self-refinement.*
2. **Want to self-refine but NO oracle**
   → First manufacture an external signal (tool / test / retrieval). If impossible, cap at fixed rounds and know it may not help — naive self-refine [degenerates](../../../examples/10-self-refine.md).
3. **Narrow solution space, unique answer** (math, extraction, classification)
   → [self-consistency](../../../examples/06-self-consistency.md) (+ [confidence-weighted voting](../../../examples/frontier/38-weighted-voting.md)). **Don't rush to [debate](../../../examples/11-debate.md).**
4. **Wide solution space** (design, writing, naming)
   → [best-of-N](../../../examples/07-best-of-n.md) / [mixture-of-agents](../../../examples/frontier/25-mixture-of-agents.md) (diversity > scale); [tournament](../../../examples/08-tournament.md) when relative comparison is easier than absolute scoring.
5. **Fear "plausible but wrong"**
   → [adversarial-verify](../../../examples/12-adversarial-verify.md) (N skeptics told to refute; majority-refute drops it).
6. **Fear "missed something"**
   → [loop-until-dry](../../../examples/04-loop-until-dry.md) + [completeness-critic](../../../examples/14-completeness-critic.md).
7. **Explore + must be able to backtrack** (planning, proof, hard synthesis)
   → [tree-search / LATS](../../../examples/21-tree-search.md).
8. **Cost-sensitive, high volume**
   → [cascade-routing](../../../examples/frontier/36-cascade-routing.md) (cheap first, escalate by confidence/agreement).
9. **Needs external / fresh knowledge**
   → [agentic-RAG](../../../examples/frontier/27-agentic-rag.md) (Self-RAG / CRAG — retrieval as an in-loop decision).
10. **Path can be predefined**
    → a **workflow** ([plan-then-execute](../../../examples/15-plan-then-execute.md) / [map-reduce](../../../examples/16-map-reduce.md) / [pipeline](../../../examples/17-pipeline.md)), NOT an agent.
11. **Plannable, no mid-task observation needed**
    → [ReWOO](../../../examples/frontier/28-rewoo.md) / [plan-and-solve](../../../examples/frontier/30-plan-and-solve.md) — avoid ReAct's token-heavy round-trips.
12. **Must observe environment mid-task** (debug, browse, tool loops)
    → [ReAct](../../../examples/19-react.md).
13. **Multi-step with a dependency chain**
    → [least-to-most](../../../examples/frontier/29-least-to-most.md).
14. **Unsure which reasoning structure**
    → [self-discover](../../../examples/frontier/31-self-discover.md) (model assembles its own).
15. **Cheap oracle + can mass-generate**
    → [AlphaCode-style](../../../examples/frontier/32-alphacode-style.md) (over-generate → oracle-filter → cluster-dedup).
16. **Open-ended, no hard oracle, need a score**
    → [rubric-based judge](../../../examples/frontier/33-rubric-based-judge.md) (anchored per-item scale).
17. **Too big for one context**
    → [divide-and-conquer](../../../examples/18-divide-and-conquer.md) (recursive) / [map-reduce](../../../examples/16-map-reduce.md) (one layer).
18. **Long-horizon agent**
    → context engineering: [subagent context isolation](../../../examples/frontier/35-subagent-context-isolation.md), pruning, compression.

## Full primitive table

### Loop family (differ only by stop condition)
| Primitive | Stop condition | Example |
|---|---|---|
| until-convergence | output ≈ previous | [01](../../../examples/01-until-convergence.md) |
| until-budget | N rounds / tokens spent | [02](../../../examples/02-until-budget.md) |
| until-threshold | self-score ≥ X | [03](../../../examples/03-until-threshold.md) |
| loop-until-dry | K rounds with no new finding | [04](../../../examples/04-loop-until-dry.md) |
| until-oracle-passes ★ | external verifier green | [05](../../../examples/05-until-oracle-passes.md) |

### Generate-then-select
| Primitive | Selection | Example |
|---|---|---|
| self-consistency | vote majority | [06](../../../examples/06-self-consistency.md) |
| best-of-N | judge picks | [07](../../../examples/07-best-of-n.md) |
| tournament | pairwise PK | [08](../../../examples/08-tournament.md) |
| judge-panel | multi-angle judges + aggregate | [09](../../../examples/09-judge-panel.md) |
| universal self-consistency | LLM picks most-consistent (open-ended) | [37](../../../examples/frontier/37-universal-self-consistency.md) |
| weighted voting | confidence-weighted majority | [38](../../../examples/frontier/38-weighted-voting.md) |
| AlphaCode-style | mass-gen + oracle filter + cluster | [32](../../../examples/frontier/32-alphacode-style.md) |

### Adversarial / critique
| Primitive | Mechanism | Example |
|---|---|---|
| self-refine ⚠️ | produce → self-critique → fix | [10](../../../examples/10-self-refine.md) |
| debate ⚠️ | two sides + judge | [11](../../../examples/11-debate.md) |
| adversarial-verify ★ | N skeptics default-refute | [12](../../../examples/12-adversarial-verify.md) |
| devil's-advocate | red-team own recommendation | [13](../../../examples/13-devils-advocate.md) |
| completeness-critic | "what's missing?" | [14](../../../examples/14-completeness-critic.md) |
| chain-of-verification | locate first error, fix only that | [24](../../../examples/frontier/24-chain-of-verification.md) |
| guardrail feedback loop | pre/post rule checks as feedback | [34](../../../examples/frontier/34-guardrail-feedback-loop.md) |
| evaluator-optimizer | separate generator vs evaluator | [23](../../../examples/frontier/23-evaluator-optimizer.md) |
| rubric-based judge | anchored per-item scale | [33](../../../examples/frontier/33-rubric-based-judge.md) |

### Decomposition
| Primitive | Shape | Example |
|---|---|---|
| plan-then-execute | plan first, then do | [15](../../../examples/15-plan-then-execute.md) |
| map-reduce | slice → parallel → merge | [16](../../../examples/16-map-reduce.md) |
| pipeline | items flow through stages, no barrier | [17](../../../examples/17-pipeline.md) |
| divide-and-conquer | recursive split/merge | [18](../../../examples/18-divide-and-conquer.md) |
| skeleton-of-thought | outline → parallel expand | [26](../../../examples/frontier/26-skeleton-of-thought.md) |
| graph-of-thoughts | thoughts as a graph, merge nodes | [39](../../../examples/frontier/39-graph-of-thoughts.md) |
| subagent context isolation | subtask in its own context | [35](../../../examples/frontier/35-subagent-context-isolation.md) |

### Search / dynamic
| Primitive | Strategy | Example |
|---|---|---|
| ReAct | reason→act→observe | [19](../../../examples/19-react.md) |
| escalation ladder | cheap fails → escalate | [20](../../../examples/20-escalation-ladder.md) |
| tree-search / LATS | expand→score→backtrack | [21](../../../examples/21-tree-search.md) |
| hill-climbing | accept only improvements | [22](../../../examples/22-hill-climbing.md) |
| cascade routing | confidence-gated model tiers | [36](../../../examples/frontier/36-cascade-routing.md) |

### Planning-style (prompt-layer)
| Primitive | Idea | Example |
|---|---|---|
| ReWOO | plan whole blueprint, execute once | [28](../../../examples/frontier/28-rewoo.md) |
| least-to-most | easy→hard dependency chain | [29](../../../examples/frontier/29-least-to-most.md) |
| plan-and-solve | plan then solve (anti skip-step) | [30](../../../examples/frontier/30-plan-and-solve.md) |
| self-discover | assemble own reasoning structure | [31](../../../examples/frontier/31-self-discover.md) |
