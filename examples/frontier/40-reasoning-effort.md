# Reasoning-effort 旋鈕（讓模型內建的迴圈先跑，別急著手疊）

- **族別**：新族 A（reasoning models — 迴圈內化進權重）的**可操作版**
- **控制骨架**：`調 effort/思考預算 → 內建長 CoT 自我檢查 → 外面仍包一層客觀 oracle`
- **核心轉變**：以前 self-refine / self-consistency 要你手動 orchestrate；reasoning model 已把「多想、自查、換策略」訓進權重。你要調的第一個旋鈕是 **reasoning effort**，不是先疊迴圈。

## Prompt / 流程範例

單一呼叫（把 effort 當第一個變因）：

```text
[設定] reasoning effort = high（或 API 的 thinking budget 調大）

解 {{HARD_PROBLEM}}。
要求：
1. 得出答案前，先驗算一次關鍵步驟（自查是內建行為，但明示可以強化）。
2. 最後輸出 `FINAL: <答案>`，以及你最不確定的一步 `WEAKEST: <描述>`。
```

Claude Code Workflow 中，effort 是每個 `agent()` 呼叫的參數 — 分層花：

```js
// 機械性階段:便宜、低 effort
const digest = await agent(extractPrompt, { model: 'haiku', effort: 'low' })
// 最難的判決階段:高 effort,取代手疊的 refine 迴圈
const verdict = await agent(hardJudgePrompt, { effort: 'xhigh' })
```

外層仍要 oracle：

```text
effort 調高後若仍答錯,先跑一次「effort=high 的單發」vs「effort=medium 的 self-consistency x5」
比較正確率,再決定要哪種花法 — 別兩種都上。
```

## 為什麼這樣做可以

- **同樣的算力，「花在哪」比「花多少」重要**。effort 旋鈕把算力花在模型內部的序列思考（會自查、會回溯），比 prompt 層再包一圈 self-refine 便宜且不易退化——內建迴圈是用 RLVR 對「可驗證的對錯」訓出來的，比你 prompt 出來的自我批判有更好的回饋品質。
- **手疊迴圈的最大風險（無外部訊號的自我修正會退化）在內建迴圈上部分緩解**，但**沒有消除**——模型內部自評一樣會錯，所以外層的客觀 oracle（測試、schema、檢索）不可省。
- **effort 分層 = cascade routing 的縱向版**：橫向是換模型（haiku→opus），縱向是同模型調思考深度。兩者都是「難的才多花」。

## 什麼時候用 / 不要用

- ✅ 目標 runtime 有 effort/thinking-budget 旋鈕（Claude / o 系列 / R1 類）。
- ✅ 你正想手疊 self-refine 或 self-consistency——先試「單發調高 effort」這個更便宜的基線。
- ✅ Workflow 腳本裡對 verify / judge 階段做算力分層。
- ❌ 任務本身簡單，高 effort 只是變慢變貴。
- ❌ 以為 effort 調高就不需要外部驗證——內部自評仍會自信地錯。

## 第二顆旋鈕：Chain-of-Draft（effort 調「想多久」，CoD 調「寫多短」）

沒有 effort 旋鈕、或想再往下壓成本時，還有一顆正交的旋鈕——限制**每步推理的長度**：

```text
逐步思考，但每一步的草稿不超過 5 個詞（只留關鍵運算/關鍵詞，不寫完整句子）。
最後一行輸出 FINAL: <答案>。
```

實測 token 降到 CoT 的 **7.6%** 而準度相當（數學/常識推理）——大部分 CoT token 是給人看的敘述，不是推理本身。兩顆旋鈕組合：effort 決定內部思考預算，CoD 決定外顯草稿密度；先單獨量測，別直接疊滿。

## 陷阱

- **全域調高**：每個呼叫都 xhigh 等於沒有分層；預設繼承 session，只在最難的節點升。
- **effort 與手疊迴圈疊加**：兩層都在做「多想幾遍」，token 翻倍、增益重疊。先量測單層。
- **拿 effort 當 oracle 用**：「想得久」不是「驗證過」；stop condition 仍必須是外部訊號。

## 來源對應

- [agentic-loop.md 新族 A](../../agentic-loop.md) — reasoning models、RLVR、「調 effort 而非自己疊迴圈,但仍需外部 oracle」。
- [Chain of Draft: Thinking Faster by Writing Less（arXiv:2502.18600）](https://arxiv.org/abs/2502.18600) — 每步 ≤5 詞的簡潔推理。
- 互補:[cascade routing](36-cascade-routing.md)(橫向換模型)、[until oracle passes](../05-until-oracle-passes.md)(外層必包)。
