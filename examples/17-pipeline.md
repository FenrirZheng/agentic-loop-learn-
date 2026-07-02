# Pipeline（多階段流水線，每個 item 獨立流過所有 stage）

- **族別**：四、Decomposition
- **控制骨架**：`每個 item 依序流過 stage1 → stage2 → …，item 之間不設 barrier`
- **重點**：階段之間**不等齊**——item A 可以在 stage 3，item B 還在 stage 1。

## Prompt 範例

以「審查一批檔案」為例，每個檔案獨立流過三個 stage：

```text
Stage 1 [REVIEW]  ── 針對這個檔案列出候選問題（附行號）。
Stage 2 [VERIFY]  ── 對上一步每個候選問題，判定是真的還是假警報（附理由）。
Stage 3 [WRITEUP] ── 把「確認為真」的問題寫成一句話修復建議。

規則：每個檔案自己走完 1→2→3，不必等其他檔案。
一個檔案在某 stage 失敗（例如 stage 2 全部否決），就中止它、跳過它剩下的 stage，
不影響其他檔案。
```

編排上：檔案 A 一產出 review 就立刻進 verify，不必等檔案 B 的 review 做完。

## 為什麼這樣做可以

- **消除 barrier 的等待浪費**。若用「先把所有檔案的 review 都做完（barrier），再統一做 verify」，整體 wall-clock 會被最慢的那個 review 拖住——快的檔案在乾等。Pipeline 讓每個 item 一到位就往下走，整體時間 ≈「最慢的單一 item 鏈」而非「每 stage 最慢者之和」。
- **階段職責單一**：每個 stage 的 prompt 只聚焦一件事（找 / 驗 / 寫），比讓一個巨型 prompt 一次做完更穩、更好除錯。
- **item 隔離**：一個 item 中途掛掉不牽連其他 item，故障被侷限。

## 什麼時候用 / 不要用

- ✅ 一批同構 item、各自要走同一串多階段處理、且**階段間不需要跨 item 資訊**。
- ❌ 某個 stage 需要「所有 item 的前一階段結果一起看」（例如全局去重、跨 item 排序、算總數才能早退）——那個點需要 **barrier**（見下方）。

## 什麼時候才真的需要 barrier

只有當 stage N 需要「上一階段全部 item 的結果」時才設 barrier，例如：
- 對全體 finding 先做跨 item 去重，再進昂貴的驗證。
- 「總數為 0 就整個跳過下游」的早退。
其餘「只是想 flatten / map / filter 一下」不構成 barrier 理由——那種轉換塞進 pipeline 的某個 stage 裡做就好。

## 來源對應

- [agentic-loop.md 四、Decomposition](../agentic-loop.md) — Pipeline。
- 需要跨 item 匯總的版本：[map-reduce](16-map-reduce.md) 的 reduce 就是一個 barrier。
