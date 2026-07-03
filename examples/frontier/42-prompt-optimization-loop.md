# Prompt optimization loop（OPRO-lite：迭代的對象是 prompt，不是答案）

- **族別**：新族 B（自我改進 / 自動優化 pipeline）的**可操作版**
- **控制骨架**：`固定一份 eval set（oracle）→ 對候選 prompt 打分 → 讓 LLM 讀「歷史候選+分數」提出新候選 → 迴圈到分數收斂或預算用完`
- **核心轉變**：前面所有招數迭代的是「這一題的答案」；這裡迭代的是「產生答案的 prompt 本身」。DSPy/OPRO 的精神，用一個 eval 迴圈就能落地簡化版。

## Prompt / 流程範例

```text
[SETUP — 先做,做不出來就停]
建一份 eval set：10–30 個「輸入 → 期望輸出/可判定準則」的案例，
必須含 3–5 個已知會讓現行 prompt 失敗的 hard case。
評分方式（擇一，客觀優先）：exact match > schema/規則檢查 > anchored rubric 評分。
沒有 eval set 就沒有這個迴圈——不要用「我看新版比較順眼」代替。

[LOOP — 最多 {{N}} 輪,或連續 2 輪最高分無進步即停]
1. SCORE：用目前候選 prompt 跑完整份 eval set，記录每案例過/不過與總分。
2. PROPOSE：把「歷屆候選 prompt + 各自總分 + 這一輪失敗案例的實際輸出」全部給
   optimizer，要求：
   「分析失敗案例的共同模式 → 提出 2 個新候選 prompt，一個保守修補、一個結構重寫。
    禁止只做同義改寫；每個候選附一句『針對哪個失敗模式』。」
3. 新候選各自跑 eval set，留下總分最高者進下一輪。

[EXIT]
輸出：最佳 prompt、最終分數、仍失敗的案例清單（誠實列出，這是下一輪 eval set 的種子）。
```

## 為什麼這樣做可以

- **eval set 就是 oracle**——它把「prompt 好不好」從品味問題變成可測量問題，於是這個 refine 迴圈不會退化（對比 naive self-refine）。OPRO 的關鍵發現：LLM 看得懂「歷史嘗試+分數」的軌跡，能從中歸納出往高分走的方向。
- **hard case 是進步的來源**：全是簡單案例的 eval set 一輪就滿分，迴圈失去梯度；已知失敗案例給了 optimizer 具體可攻的目標。
- **每輪兩個候選（保守+重寫）是廉價的多樣性**：只出一個候選容易困在局部最優；保守修補防止重寫丟掉已有的分數。

## 什麼時候用 / 不要用

- ✅ 同一 prompt 會被大量重複使用（生產 pipeline、skill、system prompt）——優化成本攤得開。
- ✅ 輸出可判定（有結構、有規則、或 rubric 穩定）。
- ❌ 一次性 prompt——直接用 [prompt-quality 的原則](../../skills/agentic-prompt-composer/references/prompt-quality.md)手寫就好。
- ❌ eval set 建不出來（輸出品質純主觀且無 rubric）——迴圈沒有訊號,分數是噪音。

## 陷阱

- **Overfit eval set**：分數上去了、真實輸入變差。留 20% 案例當 held-out,只在最後驗收用,不參與迴圈。
- **eval set 太小**：5 個案例的滿分毫無意義,雜訊蓋過訊號。
- **optimizer 只做同義改寫**：分數平了就停,別讓它空轉;明示「禁止同義改寫」+ 要求註明針對的失敗模式。
- **忘了記錄歷史**：OPRO 的力量來自「軌跡」;只給上一版就退化成 self-refine。

## 來源對應

- [agentic-loop.md 新族 B](../../agentic-loop.md) — DSPy(宣告式模組+optimizer)、OPRO(LLM 讀歷史+分數當優化器)、TextGrad(自然語言梯度)。
- 互補:[until oracle passes](../05-until-oracle-passes.md)(eval set 即 oracle)、[rubric-based judge](33-rubric-based-judge.md)(無硬 oracle 時的評分)、[hill-climbing](../22-hill-climbing.md)(只留更高分)。
