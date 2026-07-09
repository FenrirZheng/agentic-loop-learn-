# Dry-run + pre-mortem（不可逆動作前，先讓模型當世界模型模擬一遍）

- **族別**：新族 D（world models）的**可 prompt 工程版**——如同 Constitutional AI（訓練期）之於 [34 guardrail loop](34-guardrail-feedback-loop.md)（工程即時版），這是把「不在本範例集內」的 world models 精神萃取成 orchestration 層可落地的形態
- **控制骨架**：`不可逆動作前 → 模擬執行後果（狀態變化＋副作用）→ pre-mortem「假設它已經失敗了，死因是什麼」→ 後果或死因不可接受 → 改計畫，否則 GO`
- **一句話**：用「執行前的便宜模擬」換「執行後的昂貴回滾」。

## Prompt 範例

```text
以下是即將執行的計畫（先別執行）：
{{PLAN}}

[1. Dry-run 模擬]
逐步模擬執行後的狀態變化，明確列出每一步的副作用：
- 哪些檔案 / 資料被改動或刪除？可逆嗎？
- 哪些外部系統被觸碰（API、email、部署目標）？誰會看到？
- 最壞情況下，哪個副作用最難收拾？

[2. Pre-mortem]
現在假設：三個月後回頭看，這個計畫「已經失敗了」。
寫出 3 個最可能的死因——要具體（名詞＋條件），
「考慮不周」「執行不力」這類形容詞不收。

[3. 裁決]
對每個死因回答：現有計畫有防護嗎？需要改計畫、還是加檢查點？
輸出：GO / NO-GO ＋ 修訂後的計畫。
```

## 為什麼這樣做可以

- **pre-mortem 的框架反轉繞過確認偏誤**：問「這計畫有什麼問題」，剛做完計畫的模型（和人）傾向辯護；改問「它**已經**失敗了，為什麼」，把想像力從辯護轉向搜索失敗路徑——決策科學驗證過這能挖出顯著更多具體風險（Klein）。
- **LLM 當自己的世界模型是可行的**：RAP 證明用 LLM 模擬「動作 → 狀態變化」再規劃，能顯著改善多步任務——本篇是它最簡的一步版：只在不可逆節點模擬一步。
- **模擬產物是可審查的中間態**：「將發生的副作用清單」比「已發生的事故報告」便宜太多。

## 什麼時候用 / 不要用

- ✅ 刪資料、部署、對外發送（email / PR / 公開發布）、大額 API 花費、改生產設定。
- ✅ 計畫將自動執行、人不在迴圈裡——模擬是最後一道人造檢查點。
- ❌ 完全可逆的動作——模擬是浪費，直接做＋事後驗證更快。
- ⚠️ **模擬是模型的想像，不是保證**——dry-run 通過仍要進 [51 sandbox-verify-commit](51-sandbox-verify-commit.md) 的隔離執行；兩招是連續的兩道閘。

## 陷阱

- **對一切動作都模擬**：閘門只設在不可逆節點，否則整個系統慢一倍。
- **pre-mortem 變 rubber stamp**：收了「可能考慮不周」這種答案等於沒做——強制要求死因是「具體名詞＋觸發條件」。
- **把模擬通過當安全證明**：模型想像不到的失效模式模擬也想像不到；這是降風險，不是零風險。

## 來源對應

- [RAP: Reasoning via Planning（arXiv:2305.14992）](https://arxiv.org/abs/2305.14992)（LLM as world model + 規劃搜尋）；pre-mortem：Gary Klein, [Performing a Project Premortem（HBR 2007）](https://hbr.org/2007/09/performing-a-project-premortem)。
- [agentic-loop.md 新族 D](../../agentic-loop.md)——world models 的訓練期原始形態仍在研究期；本篇是其工程即時版。
- 閘門鏈：[49 clarify](49-clarify-before-act.md)（題目對嗎）→ **50 模擬**（後果可接受嗎）→ [51 sandbox](51-sandbox-verify-commit.md)（真的執行也能回滾）。
