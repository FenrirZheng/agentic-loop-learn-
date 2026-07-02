# Evaluator-Optimizer（一個 LLM 生、另一個 LLM 評+回饋，循環）

- **族別**：一之補（迴圈類的雙 agent 版）
- **控制骨架**：`repeat { optimizer 產出 → evaluator 評分+給具體回饋 → optimizer 依回饋改 } until 通過`
- **重點**：**評審與產出分離**成兩個角色/兩次呼叫。

## Prompt 範例

Optimizer：

```text
[OPTIMIZER] 任務：{{TASK}}。
（若有上一輪回饋：{{FEEDBACK}}，請只針對回饋的每一點定向修正。）
輸出你這一版的完整產出。
```

Evaluator（獨立呼叫，看不到 optimizer 的內心戲）：

```text
[EVALUATOR] 依這份 rubric 評下面的產出：{{RUBRIC}}
- 逐項打分 + 指出「具體哪裡不達標」（指到位置）。
- 給 optimizer「可執行的下一步修正」，不要只說『再好一點』。
- 若每項都達標，輸出 PASS。否則 REVISE + 回饋清單。
```

外層：`REVISE → 回 optimizer；PASS → 結束`。

## 為什麼這樣做可以

- **產出者當自己的裁判會自我強化偏誤**（見 [self-refine](../10-self-refine.md) 的退化警訊）。把 evaluator 拆成獨立角色，它沒有「護航剛才那版」的包袱，回饋更誠實。
- **回饋品質決定一切**：來源文件指出瓶頸是回饋品質不是迴圈次數。強制 evaluator 給「指到位置 + 可執行」的回饋，等於提高梯度品質。
- 這是 Anthropic 命名的標準工作流之一，本質是 [until threshold](../03-until-threshold.md) 的雙 agent 強化版。

## 什麼時候用 / 不要用

- ✅ 有清楚 rubric、值得多輪打磨、且希望評審中立的任務。
- ❌ 有客觀 oracle → 直接用 oracle 當 evaluator（[until oracle passes](../05-until-oracle-passes.md)）更可靠。

## 來源對應

- [agentic-loop.md 一之補](../../agentic-loop.md) — Evaluator-Optimizer。
