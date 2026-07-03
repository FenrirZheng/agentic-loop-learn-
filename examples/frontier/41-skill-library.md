# Skill library（Voyager 式：把 oracle 認證過的產物沉澱成可重用資產）

- **族別**：新族 E（跨任務經驗累積）的**可操作版**
- **控制骨架**：`任務內迴圈照常跑 → 通過 oracle 的產物「蒸餾」成 recipe/skill 存檔 → 下一次同類任務先檢索 library 再開工`
- **核心轉變**：其他招數都在「單一任務內」打轉，每題從零開始；skill library 讓迴圈**跨任務**累積——上一輪驗證過的解法是下一輪的起點。

## Prompt / 流程範例

任務收尾時加一個「蒸餾」步驟：

```text
[DISTILL] 本次任務已通過驗證（{{ORACLE_EVIDENCE}}：測試綠燈 / 審核通過）。
把可重用的部分蒸餾成一份 recipe，存到 {{LIBRARY_DIR}}/<kebab-case-名稱>.md：

1. WHEN：什麼樣的任務該用這份 recipe（觸發特徵，寫給未來的檢索者）。
2. STEPS：去除本次特定細節後的通用步驟。
3. ORACLE：這份 recipe 的產出要怎麼驗證（沿用本次的驗證方式）。
4. PITFALLS：本次踩過、修過的坑（只寫真的發生過的，不寫想像的）。

只蒸餾「oracle 認證過」的內容；沒驗證過的觀察寫進 PITFALLS 並標 [UNVERIFIED]。
```

下一次任務開頭：

```text
[RETRIEVE] 開工前先列出 {{LIBRARY_DIR}} 中 WHEN 條件符合本任務的 recipe（最多 3 份）。
有命中 → 以它的 STEPS 為初始計畫、PITFALLS 為 checklist；
沒命中 → 照常從頭做，做完記得 DISTILL。
```

Claude Code 的落地形式：`.claude/skills/` 目錄本身就是 skill library——把驗證過的工作流寫成 SKILL.md；Workflow 腳本則在 return 前多跑一個 distill agent，把 recipe 寫進 repo，下次以 `args` 餵回。

## 為什麼這樣做可以

- **入庫門檻 = oracle**，這是 Voyager 最漂亮的設計：技能能不能存，由「環境跑得動」裁決，不是模型自評。庫裡因此只有被認證過的資產，檢索到就能信。
- **把 test-time 花的算力變成資產**：一次 R2（until-oracle-passes）迴圈花掉的修錯過程，蒸餾後讓同類任務直接跳過那些坑——這是唯一會**複利**的招數。
- **PITFALLS 是最高價值欄位**：通用步驟網路上到處有，「這個 codebase / 這類任務真實踩過的坑」只有你的迴圈知道。

## 什麼時候用 / 不要用

- ✅ 同類任務會重複出現（部署、遷移、審計、同框架的 CRUD……）。
- ✅ 任務有明確 oracle，「認證過才入庫」執行得起來。
- ❌ 一次性任務——蒸餾成本收不回。
- ❌ 沒有 oracle 就入庫——庫會被「聽起來對」的內容汙染，之後每次檢索都在放大錯誤。

## 陷阱

- **庫只進不出**：環境變了 recipe 會過期。每次「照 recipe 做卻失敗」就是它的過期訊號——當場修訂或標記棄用，別靜默繞過。
- **蒸餾寫成流水帳**：把本次所有細節照抄不是蒸餾；WHEN 寫不清楚,未來檢索不到,等於沒存。
- **跳過檢索**：有庫不查、每次重新發明，是最常見的失敗模式——RETRIEVE 步驟要寫進任務模板的第一步，不是靠記憶。

## 來源對應

- [agentic-loop.md 新族 E](../../agentic-loop.md) — Voyager:自動課程、skill library、環境回饋迭代;「存進 library 的都是被 oracle 認證過的」。
- 互補:[until oracle passes](../05-until-oracle-passes.md)(入庫門檻)、[agentic RAG](27-agentic-rag.md)(檢索端)。
