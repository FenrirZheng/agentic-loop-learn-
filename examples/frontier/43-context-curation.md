# Context curation（長迴圈不崩的地基：主動策展 context）

- **族別**：新族 C（記憶與 context engineering）的**可操作版**
- **控制骨架**：`每 N 輪（或每階段收尾）: 蒸餾狀態 → 丟棄原始過程 → 只帶結論進下一輪`
- **核心問題**：任何 loop-until-dry / 長時程迴圈，不做 context 管理就會 **context rot**——歷史越堆越長，訊號被稀釋，越跑越笨。瓶頸不是記憶容量，是「對的證據有沒有在對的時間點出現在 context 裡」。

## Prompt / 流程範例

長迴圈的每輪收尾加一個壓縮步驟：

```text
[COMPRESS — 每輪結束執行]
把本輪過程壓縮成狀態摘要，之後只帶摘要進下一輪：

1. VERIFIED：本輪確認的事實（含證據位置 file:line / URL / 測試名）。
2. DEAD-ENDS：試過且排除的路徑 + 一句為什麼（防下一輪重試）。
3. OPEN：still open 的問題,按優先級排。
4. NEXT：下一輪的第一步。

規則：
- 原始 tool output / 長引文一律丟棄，只留結論與「去哪裡重新取得」的指標。
- VERIFIED 不得覆寫：新證據與舊結論衝突時,兩者並列標 [CONFLICT],不要靜默取代。
```

搭配三個結構性手法：

- **Subagent 隔離**（最重要）：大搜尋/大驗證丟給 subagent 在獨立 context 跑，主線只收結論——見 [35](35-subagent-context-isolation.md)。
- **外部化**：狀態寫進檔案（`notes.md`、TODO list），context 裡只留指標；要用時再讀回來（memory-as-action）。
- **Sleep-time 預消化**：可預讀的持久 context（codebase、歷史對話）趁閒置先蒸餾成 learned context，熱路徑直接用——排程 agent / cron 就是它的落地形式。

## 為什麼這樣做可以

- **注意力是稀釋性資源**：context 裡每多一段無用歷史,關鍵證據分到的注意力就少一分。壓縮不是省 token 的小氣,是**維持訊號密度**——這是長迴圈品質衰退的第一因。
- **DEAD-ENDS 與 VERIFIED 同等重要**：沒記「排除了什麼」,長迴圈會在第 8 輪重試第 3 輪失敗的路徑;沒記證據位置,壓縮就變成傳話遊戲,錯誤逐輪放大。
- **蒸餾要趁熱**：每輪收尾時做,細節還在;等 context 快爆了才壓,已經分不清哪些是事實哪些是猜測。

## 什麼時候用 / 不要用

- ✅ 預期超過 ~5 輪的任何迴圈（loop-until-dry、長 debug、大遷移、深度研究）。
- ✅ 多 agent 接力的 pipeline——交接物就是壓縮後的狀態摘要,不是完整歷史。
- ❌ 三輪內會結束的短任務——壓縮的成本與失真反而虧。
- ❌ 需要逐字保真的內容（法條、合約原文）——外部化存檔+指標,不要壓縮改寫。

## 陷阱

- **壓縮時把猜測升級成事實**：摘要最常見的失真。強制 VERIFIED 欄位帶證據位置,無證據的進 OPEN。
- **丟了「去哪裡重新取得」的指標**：壓縮後發現需要細節卻回不去了。丟內容可以,丟指標不行。
- **一直壓同一份摘要**：摘要的摘要的摘要,三代之後只剩口號。以「原始證據指標」為錨,定期從源頭重建,而不是無限迭代摘要。

## 來源對應

- [agentic-loop.md 新族 C](../../agentic-loop.md) — context pruning、trajectory compression、memory-as-action、外部記憶;「主動策展 context」。
- [agentic-loop.md 新族 F](../../agentic-loop.md) — sleep-time compute:閒置時把 raw context 消化成 learned context（~1/5 token、+15% 正確率）。
- 互補:[subagent context 隔離](35-subagent-context-isolation.md)(結構性隔離)、[map-reduce](../16-map-reduce.md)(digest 而非原文往上傳)。
