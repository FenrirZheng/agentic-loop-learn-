# Plan-and-Solve（先訂計畫再照計畫逐步解，治 zero-shot CoT 跳步）

- **族別**：四之補(2)（prompt 層規劃法）
- **控制骨架**：`先叫模型「訂一個計畫」→ 再「照計畫逐步解」`
- **重點**：專治 zero-shot CoT 會**跳步**、漏中間環節的毛病。

## Prompt 範例

```text
題目：{{PROBLEM}}

請先「理解題目並訂一個計畫」，再照計畫執行：
1. [PLAN] 先復述題目已知與要求，然後列出「解這題要依序做哪幾步」（先不算）。
2. [SOLVE] 嚴格照 PLAN 的每一步逐步計算，每一步都寫出中間結果，不要跳過任何一步。
3. [ANSWER] 給最終答案，並回頭檢查它是否用到了 PLAN 裡每一步的結果。
```

## 為什麼這樣做可以

- **zero-shot CoT（「讓我們一步步想」）常常跳步**：模型會略過它自認為顯然的中間環節，而錯誤往往就藏在被跳過的那一步。先強制「訂計畫」讓它把所有必要步驟外顯，再「照計畫算」時就難以偷偷跳過。
- **「復述已知與要求」**這個開頭動作能減少誤解題意——很多錯是從一開始讀錯題目就注定的。
- 相比 [plan-then-execute](../15-plan-then-execute.md)（偏工程任務、計畫可被人核可），Plan-and-Solve 更輕、專用於**單題推理**的 prompt 內建兩段式。

## 什麼時候用 / 不要用

- ✅ zero-shot 推理題、應用題，且你觀察到模型會跳步/算漏。
- ✅ 不想給 few-shot 範例、只靠 prompt 結構就想穩住推理。
- ❌ 需要工具/檢索的互動任務——用 [ReAct](../19-react.md) 或 [agentic RAG](27-agentic-rag.md)。

## 來源對應

- [agentic-loop.md 四之補(2)](../../agentic-loop.md) — Plan-and-Solve。
- 依賴鏈版：[Least-to-Most](29-least-to-most.md)；工程任務版：[plan-then-execute](../15-plan-then-execute.md)。
