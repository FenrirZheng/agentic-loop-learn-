# Self-consistency（獨立多跑取多數）

- **族別**：二、Generate-then-Select
- **控制骨架**：`同題獨立跑 N 次 → 對「最終答案」投票取眾數`
- **重點**：不迭代，改成並行發散 + 多數決收斂。

## Prompt 範例

三次獨立呼叫用同一個 prompt（關鍵是**獨立**、彼此看不到對方）：

```text
[呼叫 1..N，各自獨立、溫度略高]
題目：{{PROBLEM}}
請一步一步推理，最後一行只輸出：
FINAL: <你的最終答案>
```

彙整（第 N+1 次呼叫）：

```text
以下是同一題的 N 個獨立解答的 FINAL 值：
{{list_of_finals}}
規則：
- 只做計票，不要重新推理。
- 輸出出現次數最多的那個答案，以及票數分佈（例：42 → 3 票 / 40 → 1 票）。
- 若最高票只領先 1 票，標記 [LOW CONFIDENCE] 並列出前兩名。
```

## 為什麼這樣做可以

- **錯誤是分散的、正確是集中的**：對有唯一解的推理題，模型走錯路的方式五花八門（各種算錯），但走對路通常殊途同歸。獨立取樣後取眾數，等於讓「隨機錯誤」互相抵消、「系統性正確」浮出來。
- **並行比序列穩**：序列 self-refine 可能一路把自己帶偏；獨立取樣沒有這種相互汙染，也不會卡在單一推理鏈的局部最優。
- **多數決本身就吃掉大部分增益**——來源文件指出，self-consistency 的效果主要來自「多次取樣 + 投票」，這也是為什麼**先別急著上 debate**（辯論常贏不過單純的 self-consistency）。

## 什麼時候用 / 不要用

- ✅ **解空間窄、有唯一解**的推理題（數學、邏輯、抽取單一事實）。
- ❌ 開放式、沒有唯一解、答案無法精確比對（寫作、設計）——投票沒東西可投，改用 [universal self-consistency](frontier/37-universal-self-consistency.md)（讓一個 LLM 挑「最一致」的那個）或 [best-of-N](07-best-of-n.md)。

## 進階：信心加權

比純多數決更好一點的是**信心加權投票**——讓每個解自報信心，加權計票。但別過度期待增益：多數決已經拿走大部分好處。

## 來源對應

- [agentic-loop.md 二、Generate-then-Select](../agentic-loop.md) — Self-consistency。
- 開放式版本：[universal self-consistency](frontier/37-universal-self-consistency.md)；加權版：[weighted voting](frontier/38-weighted-voting.md)。
