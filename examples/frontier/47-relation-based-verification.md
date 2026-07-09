# 關係式驗證（round-trip + metamorphic：沒有 ground truth 也有客觀訊號）

- **族別**：三之補（驗證家族）
- **控制骨架**：`正向產出 → 從任務結構導出「必須成立的關係」→ 檢查關係 → 不成立 = fail`
- **定位**：現有驗證招（[12](../12-adversarial-verify.md)/[24](24-chain-of-verification.md)/[33](33-rubric-based-judge.md)）的訊號都是「另一個 LLM 的判斷」；關係式驗證的訊號來自**任務結構本身**——客觀性介於 LLM-judge 與硬 oracle 之間，正好填「沒測試可跑、又不想全信 judge」的空隙。

## Prompt 範例

**Round-trip（正向做完，反向重建，比對等價）**：

```text
[STEP 1] 為以下函式寫一段自然語言規格（涵蓋所有行為，包括邊界）：
{{CODE}}

[STEP 2，fresh context——不可看到原 code]
根據以下規格實作函式：
{{STEP1_SPEC}}

[STEP 3，程式執行，非 LLM]
對原 code 與重建 code 跑同一組輸入，行為不一致的輸入點列出來。
不一致 ⇒ 規格漏了東西，或 code 有規格外的隱藏行為——都值得人看。
```

同型：翻譯→回譯→語意比對；結構化抽取→用抽取結果重寫原文→事實比對；摘要→擴寫→關鍵 claim 比對。

**Metamorphic（擾動輸入，檢查輸出按預期關係變化）**：

```text
原題：{{QUESTION}} → 原答案：{{ANSWER}}

生成 3 個擾動版本並重答（fresh context）：
1. 同義改寫（答案應不變）
2. 插入一段無關資訊（答案應不變）
3. {{任務特定的等變關係，例：輸入數字全部 ×2 → 輸出應 ×2}}

任何「該不變的變了 / 該等變的沒等變」都是 fail。
```

## 為什麼這樣做可以

- **關係是客觀性質，不是意見**。「回譯後意思應該一樣」「加無關句不該改變答案」不依賴任何 judge 的品味——檢查結果可重現、可程式化。
- **fresh context 是機制核心**：STEP 2 看得到原 code 就會抄，round-trip 假通過。隔離讓「規格是否完整」真正被測到。
- **實證**：Round-Trip Correctness 與人工標註的 benchmark 高度相關（ICML 2024），metamorphic testing 是軟體測試界對付「oracle 不存在」的標準解法，搬到 LLM 輸出上同樣成立。

## 什麼時候用 / 不要用

- ✅ 沒有現成測試的 code、翻譯、結構化抽取、摘要——任務有「可逆」或「等變」結構的。
- ✅ 大量輸出要自動篩掉可疑件、人工只看 fail 的（配 [16 map-reduce](../16-map-reduce.md) 規模化）。
- ❌ 開放式創作——「關係」定義不出來就沒有著力點。
- ⚠️ **關係成立 ≠ 正確**（必要非充分）：round-trip 一致可能是「一致地錯」。它是廉價的第一層濾網，不是終審。

## 陷阱

- **round-trip 共用 context**：反向步驟看得到正向的輸入，自己抄自己，永遠通過。
- **金絲雀關係（永遠成立的 MR）**：擾動太弱（改個標點）測不出東西——好的 MR 要真的施壓。
- **把通過當充分條件**：它只證明「沒有測到矛盾」；高風險輸出仍要疊 [12 adversarial verify](../12-adversarial-verify.md) 或人工抽查。

## 來源對應

- [Unsupervised Evaluation of Code LLMs with Round-Trip Correctness（ICML 2024，arXiv:2402.08699）](https://arxiv.org/abs/2402.08699)、[Metamorphic Testing of LLMs for NLP（arXiv:2511.02108）](https://arxiv.org/abs/2511.02108)、[Metamorphic Prompt Testing（arXiv:2406.06864）](https://arxiv.org/html/2406.06864v1)。
- 驗證訊號的客觀性光譜：LLM-judge（[33](33-rubric-based-judge.md)）＜ **關係式（本篇）** ＜ 硬 oracle（[05](../05-until-oracle-passes.md)）——能上就往右上。
