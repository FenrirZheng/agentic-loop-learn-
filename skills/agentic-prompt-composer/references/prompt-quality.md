# Prompt Quality — writing the emitted text itself

The catalog picks *which* primitives; this file governs *how the emitted prompt text is written*. A perfect composition rendered as vague prose still fails. Apply this to every artifact you emit — markdown specs AND every `agent()` prompt string inside an .mjs script.

## Anatomy of a strong agentic prompt

Emit prompts with these seven parts, in roughly this order. Skipping one should be a deliberate choice, not an accident.

1. **Role + stance** — not decoration; the stance sets the prior. A verifier gets "default: this finding is a FALSE ALARM"; a finder gets "assume bugs exist and you haven't found them all yet". Stance is the cheapest behavior knob you have.
2. **Inputs, explicit and delimited** — every external input appears as a named placeholder (`{{TARGET_DIR}}`, `{{DIFF}}`, `{{FINDING_JSON}}`) inside clear delimiters (fenced block / XML tag). **Fail closed**: state "if {{X}} is empty or missing, STOP and say so — do not guess a value."
3. **Procedure** — the selected primitives compiled into numbered steps. One step = one action + its observable result. Steps the model tends to skip get an explicit "do not skip" (plan-and-solve's whole reason to exist).
4. **Output contract** — the exact shape of the answer: field names, order, format, length caps. Prefer a literal skeleton the model fills in over a description of one. In .mjs this is the `schema` opt; in markdown, write the skeleton in a fenced block. A prompt without an output contract cannot be verified by the next stage — and everything you compose has a next stage.
5. **Stop condition, stated inside the prompt** — the loop guard also lives in the text the model sees ("repeat until pytest exits 0; paste the real output each round; max 5 rounds").
6. **Negative constraints** — the 2–4 failure modes this specific composition fears, phrased as prohibitions with the *reason*: "Never claim done without pasting real test output (models report imagined passes)". Reasoned prohibitions survive paraphrase; bare ones get rationalized away.
7. **Escape hatch** — the honest exit: "If you cannot verify a claim, mark it `UNVERIFIED` — an admitted gap is worth more than a confident guess." Without an escape hatch, the output contract *forces* hallucination when the model has nothing.

## Per-role prompt rules

### Subagent prompts (every `agent()` string, every Task/Agent dispatch)

- **Self-contained**: the subagent sees NONE of your conversation. Inline everything it needs — the finding JSON, the file list, the acceptance criteria. A prompt that says "verify the finding above" is broken by construction.
- **One job**: one agent = one responsibility. "Find bugs AND rank them AND write the report" belongs to three agents or a script.
- **Return contract**: tell it its final message IS the data ("return only the JSON, no preamble"). With `schema` this is enforced; without, say it explicitly.

### Verifier / skeptic prompts

- **Default-refute stance**: "Your job is to REFUTE this. Default verdict: false alarm. Only confirm if you can prove it." A neutral "please verify" collapses into agreement bias.
- **Demand evidence, not opinion**: a confirm requires `file:line` + the concrete failure scenario (inputs → wrong output); a refute requires the specific reason the claim doesn't hold. Forbid "seems plausible/unlikely" as verdict grounds.
- **Uncertainty maps to refute** (when false positives are the fear): "if you cannot decide, verdict = refuted." State the tie-break; don't leave it to mood.
- **Lens diversity is written, not implied**: each skeptic's prompt names its distinct lens (correctness / security / does-it-reproduce) and what that lens IGNORES. N copies of one skeptic prompt is redundancy, not diversity.

### Judge / rubric prompts

- **Anchor every criterion**: define what 0 and max look like per criterion, with a one-line example of each. "Rate clarity 1–10" unanchored produces noise centered on 7.
- **Generative verdict**: require the judge to write its analysis BEFORE the score ("for each criterion: cite the specific passage, explain, then score"). Reason-first judges are measurably more accurate than score-only ones.
- **Kill position/verbosity bias**: pairwise comparisons run both orders (A-B and B-A) and only count agreements; add "longer is not better — penalize padding" when judging prose.
- **Scores need consequences**: state what happens at each band ("< 6 → revise addressing the cited passages; ≥ 8 → accept"). A score nothing branches on is decoration.

### Finder / generator fan-out prompts

- **Name the angle, per prompt**: each fan-out member gets its own strategy sentence ("you approach this as a resource-lifecycle auditor: leaks, double-frees, unclosed handles — ignore style"). Cosmetic paraphrases of one prompt = scale pretending to be diversity.
- **SEEN dedup is the finder's job too**: pass the seen-list in and demand "report only findings NOT in this list" — don't rely solely on post-hoc dedup.
- **Force specificity fields**: every finding carries `file`, `line`, `summary`, and a `failure_scenario` (concrete inputs → concrete wrong behavior). Findings without a failure scenario are opinions.

## Few-shot: when one worked example earns its tokens

Add a single worked example when the output contract is unusual (nested JSON, a scoring format, a diff format) or when past runs show format drift. Skip it when the contract is a plain list/JSON — the skeleton suffices. Never let the example's *content* leak into the task (label it `EXAMPLE — different domain, format only`).

## Emitted-prompt smell test

Reject your own draft if any of these hold:

- A placeholder the runner must fill has no "abort if missing" rule. (fail-open)
- The output has no literal skeleton/schema. (unverifiable by the next stage)
- Any instruction says "be thorough / be careful / think hard" with no operational meaning. (vibes, not procedure)
- A verifier prompt doesn't state the default verdict and tie-break. (agreement bias in)
- Two fan-out prompts differ only in wording, not strategy. (fake diversity)
- The stop condition appears in your spec but not in the prompt text the model actually sees.
- No escape hatch, but the task can genuinely come up empty. (forced hallucination)
- A prohibition has no reason attached. (gets rationalized away on the third round)
