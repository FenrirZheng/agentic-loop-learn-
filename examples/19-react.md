# ReAct（reason → act → observe 迴圈）

- **族別**：五、Search / 動態策略
- **控制骨架**：`迴圈 { Thought：想下一步 → Action：呼叫工具 → Observation：讀結果 } 直到能回答`
- **重點**：每一步都依「觀察到的真實結果」決定下一步——agentic 的基礎骨架。

## Prompt 範例

```text
你可以使用工具：search(query)、read(url)、calc(expr)。
用嚴格的 ReAct 格式回答問題，每輪輸出：

Thought: <根據目前已知，下一步該做什麼、為什麼>
Action: <工具名>(<參數>)
（我會回填）Observation: <工具的真實回傳>

重複上述，直到你有足夠證據。硬規則：
- 絕不憑空捏造 Observation——只能用我回填的真實結果推進。
- 每個 Thought 必須引用「上一個 Observation 的具體內容」，不能無視觀察硬推。
- 準備好時輸出：
  Final Answer: <答案> + <你依據的關鍵 Observation>
```

## 為什麼這樣做可以

- **控制流由「結果」決定，而非事先寫死**：很多任務下一步該做什麼，取決於上一步查到什麼。ReAct 讓模型把「推理」和「行動」交錯，用真實觀察修正方向，而不是一開始就腦補整條路徑然後照著錯下去。
- **Thought 外顯**讓推理可檢查、可干預；**強制引用上一個 Observation** 防止模型無視工具結果、自說自話（這是 ReAct 最常見的失敗——查了卻不看）。
- **禁止捏造 Observation** 是防幻覺護欄：模型很愛自己編一個「工具應該會回這個」然後往下跑。

## 什麼時候用 / 不要用

- ✅ **互動型、需要邊做邊看環境**的任務：查資料回答、操作 API、debug（跑一下看報錯再決定）。
- ❌ 任務**可事先完整規劃**、不需要中途觀察——那用 [ReWOO](frontier/28-rewoo.md) 或 [Plan-and-Solve](frontier/30-plan-and-solve.md)，省掉逐步往返的大量 token。
- ❌ 路徑能寫死 → 用 workflow，別上 agent。

## 一句話對照

- **ReAct = 邊看邊想邊做**（互動任務強，但 think-act-observe 來回很耗 token）。
- **ReWOO / Plan-and-Solve = 先想好再做**（可規劃任務省又穩）。

## 進階

ReAct 是很多進階 agent 的底座：加上樹搜尋 + value function + 反思回溯 = [LATS](21-tree-search.md)；把檢索決策拉進迴圈 = [agentic RAG](frontier/27-agentic-rag.md)。

## 來源對應

- [agentic-loop.md 五、Search / 動態策略](../agentic-loop.md) — ReAct；省 token 的對照替代見〈四之補(2)〉。
