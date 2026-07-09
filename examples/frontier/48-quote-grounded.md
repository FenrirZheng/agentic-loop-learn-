# Quote-grounded generation（先逐字摘引、再作答：讓幻覺在結構上寫不出來）

- **族別**：三之補（[27 agentic RAG](27-agentic-rag.md) 的姊妹——27 管「怎麼查」，本篇管「查回來之後怎麼防摻私貨」）
- **控制骨架**：`第一步只准逐字抽引文＋出處 → 程式驗證引文真的在來源裡 → 第二步只准根據引文作答，每個 claim 指回引文編號`
- **核心轉變**：幻覺防治從**事後驗證**（[24 CoVe](24-chain-of-verification.md)：寫完再查）提前到**結構上不可能**——沒有引文的 claim 根本不准寫。

## Prompt 範例

第一步（只抽引文，不作答）：

```text
問題：{{QUESTION}}
文件：{{DOCUMENTS}}

從文件中「逐字」抽出與問題相關的引文，每條標 [1][2]… 與出處（文件名＋段落）。
規則：
- 一字不改的原文，不是改寫。
- 抽不到相關引文 → 輸出 NO_QUOTES，不要硬湊。
```

中間（程式，不是 LLM）：

```python
# 逐字引文是可程式驗證的 —— substring 檢查就是一個免費 oracle
for q in quotes:
    assert q.text in source_docs[q.doc], f"引文 {q.id} 不在來源裡（被幻覺）"
```

第二步（只根據引文作答）：

```text
只根據以下已驗證的引文回答問題，規則：
- 每個事實宣稱句尾標註支持它的引文編號 [n]。
- 引文不支持的內容一個字都不要寫——寧可回答「來源不足以回答這部分」。
引文：{{VERIFIED_QUOTES}}
問題：{{QUESTION}}
```

## 為什麼這樣做可以

- **約束生成，而非驗證生成**：模型在第二步的「原料」只有引文——摻私貨需要違反明示的生成規則，比「寫完再抓」少一個數量級的漏網。
- **逐字引文給了一個免費 oracle**：改寫無法程式驗證，逐字可以（substring check）。這把「引文本身被幻覺」這個次生風險也堵掉了。
- **claim→引文的對應讓人工抽查變 O(1)**：審查者不用讀全文，只需比對「這句 vs [3] 那條引文」。可稽核性本身就是價值。

## 什麼時候用 / 不要用

- ✅ RAG 問答、長文件 QA、法律 / 醫療 / 財務等需要 provenance 的輸出。
- ✅ 「答案將被引用或轉發」的場景——出處跟著答案走。
- ❌ 需要綜合推斷、結論本來就該超出來源字面的任務（會過度保守，把合理推論也悶死）。
- ⚠️ 引文可以真、解讀仍可以歪——第二步的「詮釋」仍要抽查 claim-引文對應是否真的成立。

## 陷阱

- **一步做完**：讓模型「邊答邊引」，引文會退化成裝飾——先抽後答的**順序**才是機制。
- **跳過 substring 驗證**：引文本身被幻覺的比率不低，這步是整個結構的地基。
- **來源本身是錯的**：本招保證「忠於來源」，不保證「來源為真」——garbage in, faithfully garbage out。來源品質另配 [27 CRAG](27-agentic-rag.md) 的檢索評估。

## 來源對應

- [Anthropic — Reduce hallucinations（官方 docs）](https://docs.anthropic.com/en/docs/test-and-evaluate/strengthen-guardrails/reduce-hallucinations)（"allow Claude to say I don't know" + "use direct quotes for factual grounding"）、[Anthropic — Citations API](https://docs.anthropic.com/en/docs/build-with-claude/citations)（此模式的產品化）。
- 對照：[24 CoVe](24-chain-of-verification.md)（事後驗證）vs 本篇（事前約束）——高風險場景兩層都上。
