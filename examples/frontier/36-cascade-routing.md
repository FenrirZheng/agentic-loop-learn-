# Model cascade / routing（成本量化版的升級階梯）

- **族別**：五之補（escalation ladder 的成本量化版）
- **控制骨架**：`便宜模型先答 → 依「信心 / 一致性 / 不確定性」門檻決定是否升級到貴模型`
- **與 escalation ladder 差別**：ladder 是概念；cascade 給出**量化的升級門檻**，甚至形式化成本保證。

## Prompt / 流程範例（agreement-based gating）

```text
[TIER 1] 用便宜模型「獨立跑 3 次」回答 {{QUESTION}}。

[GATE] 檢查這 3 次答案：
- 3 次一致 → 直接採用（高信心，不升級）。
- 有分歧 → 觸發升級。

[TIER 2] 用貴模型回答，允許多步推理/工具。以它的答案為準。
```

其他 gating 變體：
- **信心門檻**：tier 1 自報信心 < 0.7 才升級。
- **conformal prediction**：用校準過的統計門檻決定「答案是否可靠到不必升級」，可給出「錯誤率不超過 ε」的保證。

## 為什麼這樣做可以

- **升級決策需要可靠的訊號，而非模型隨口說『我確定』**。cascade 的核心是把 gate 建立在**可量化**的訊號上：多個便宜模型的**一致性**、校準過的**信心**、統計**門檻**——這些比自陳信心穩。
- **agreement-based 特別聰明**：便宜模型們都同意時，答案通常簡單且對；它們吵起來時，正是「這題難、值得付貴模型」的信號。用分歧當升級觸發，把貴算力精準花在難題上。
- 相比 [escalation ladder](../20-escalation-ladder.md) 的定性描述，cascade 讓「省多少、錯多少」可被量化甚至保證。

## 變體：speculative draft-verify（分工，不是路由）

cascade 是「整題路由」（簡單題便宜模型全包）；另一種拓撲是**每題都兩個模型分工**——便宜模型出完整草稿，貴模型只做審查與修訂：

```text
[DRAFT · 便宜模型] 完成 {{TASK}}，輸出完整草稿。
[VERIFY · 貴模型] 審查上述草稿：對的部分保留不重寫，只修有問題的段落，並列出改了什麼、為什麼。
```

適合「產出量大、但錯誤集中在少數難點」的任務（翻譯、樣板代碼、資料轉換）——貴模型的算力全花在修錯而非重複生成。這是 speculative decoding 的 orchestration 層類比。注意：若貴模型審查時幾乎整份重寫，說明任務不適合此拓撲，直接上貴模型。

## 什麼時候用 / 不要用

- ✅ 大流量、成本敏感、且能定義可靠 gating 訊號的服務。
- ✅ 需要對成本/錯誤率給出量化保證。
- ❌ 每題都難或錯誤代價極高——直接上最強檔，別冒便宜路徑的險。

## 來源對應

- [agentic-loop.md 五之補](../../agentic-loop.md) — Model cascade / routing（agreement-based、conformal-prediction 門檻）。
- 定性版：[escalation ladder](../20-escalation-ladder.md)。
