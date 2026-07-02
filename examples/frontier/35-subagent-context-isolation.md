# 子代理 context 隔離（subagent-as-tool）

- **族別**：四之補（decompose 族）／新族 C（context engineering）
- **控制骨架**：`把子任務丟給 sub-agent 在「獨立 context」跑 → 只把「結論」回傳主線`
- **重點**：避免主線 context 被子任務的大量中間過程汙染（context rot）。

## Prompt / 編排範例

```text
[主線 orchestrator]
我需要「盤點這個 repo 用到哪些第三方 API」。這是一個會產生大量檔案內容的搜尋。

把它丟給一個 sub-agent，在它自己的 context 裡跑：
  子任務指令：「搜尋整個 repo，找出所有第三方 API 呼叫。你可以讀很多檔案，
  但只回傳一份結構化清單：API 名稱 + 出現的檔案:行 + 用途一句話。
  不要把讀過的檔案原文回傳給我。」

主線只收下那份清單，據此繼續規劃——主線 context 不承載子任務翻過的幾十個檔案。
```

## 為什麼這樣做可以

- **搜尋/探索型子任務會吞掉大量 token**（翻幾十個檔案），若全塞進主線 context，會擠掉真正重要的資訊、拉高成本、並讓模型「中間遺失」——這就是 context rot。
- **子代理在獨立 context 跑、只回傳結論**，等於做了一次「有損壓縮」：主線拿到的是提煉後的答案，而不是原始過程。主線的注意力保持在高價值資訊上。
- 這是「把子代理當工具用」的思路——主線像呼叫一個函式，只關心回傳值，不關心它內部翻了多少資料。

## 什麼時候用 / 不要用

- ✅ 子任務會產生大量中間內容、但主線只需要它的結論：大範圍搜尋、多檔案分析、跑一長串驗證。
- ✅ 長時程 agent，主線 context 寶貴。
- ❌ 子任務的中間過程主線後續還要用到——那隔離反而丟失必要資訊。

## 相關 context engineering 手法

context pruning（剪掉迴圈內無用歷史）、trajectory compression（壓縮走過的路徑）、memory-as-action（把「要記什麼」變成一個動作）。任何 [loop-until-dry](../04-loop-until-dry.md) 型長迴圈不做 context 管理就會越跑越笨。

## 來源對應

- [agentic-loop.md 四之補 & 新族 C](../../agentic-loop.md) — 子代理 context 隔離 / context engineering。
