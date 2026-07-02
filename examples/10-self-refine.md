# Self-refine / Reflexion（產出 → 自我批判 → 依批判修正）

- **族別**：三、Adversarial / Critique
- **控制骨架**：`固定幾輪 { 產出 → 針對自己的產出批判 → 依批判改 }`
- **與 convergence 差別**：強制「每一輪都先批判再改」，而不是無腦再改一版。

## Prompt 範例

```text
任務：{{TASK}}。做兩輪 self-refine。

第 1 輪：
[DRAFT 1] 先給一版完整答案。
[CRITIQUE 1] 現在切換成嚴格審稿人，專門挑 DRAFT 1 的毛病：
  列出「具體、可修正」的缺陷（每條指到位置），至少 3 條；
  若真的挑不出，說明你「用什麼外部依據」確認它沒問題（不准只說『看起來很好』）。
[REVISE 1] 只針對 CRITIQUE 1 的每一條做定向修正。

第 2 輪：對 REVISE 1 重複 CRITIQUE → REVISE。
最後輸出 [FINAL]。
```

## 為什麼這樣做可以

- **把「批判」和「產出」在時間上分開**，避免模型一路自我肯定。要求批判「指到具體位置、至少 N 條」逼它真的找碴，而不是敷衍地說「已經很好了」。
- **定向修正**（只改被批判到的點）比整篇重寫收斂更穩、不會引入新問題。

## ⚠️ 最重要的警訊：沒有外部訊號，self-refine 會越改越差

這是來源文件反覆強調、也最反直覺的一條：

> **沒有外部訊號時，LLM 的 intrinsic self-correction 通常會越改越差**（degeneration-of-thought）。瓶頸是「回饋品質」，不是迴圈次數。

換句話說，**純靠模型自己批判自己**很危險——它可能把對的改成錯的、越反省越鑽牛角尖。所以：

1. **能接 oracle 就一定要接**：讓批判基於「真的跑了測試/工具/檢索的結果」，而不是模型的感覺。這就是 CRITIC 精神（見 [until oracle passes](05-until-oracle-passes.md)）。
2. 若真的沒 oracle，**固定輪數**（如 2 輪）比開放迴圈安全——別讓它無限反省。
3. 更進一步，把「批判者」換成獨立角色/模型（見 [evaluator-optimizer](frontier/23-evaluator-optimizer.md)），比自己批判自己更不會自我強化偏誤。

## 什麼時候用 / 不要用

- ✅ 有外部訊號可接、或至少能請外部工具驗證的任務。
- ✅ 固定少數幾輪的品質提升。
- ❌ 純內省、無任何外部依據、又想開放迴圈跑——這正是會退化的場景，不如不改。

## 來源對應

- [agentic-loop.md 三、Adversarial / Critique](../agentic-loop.md) — Self-refine / Reflexion；退化警訊見〈一之補〉的重大警訊。
- 精準定位錯誤節點只重修一步的版本：[Chain-of-Verification](frontier/24-chain-of-verification.md)。
