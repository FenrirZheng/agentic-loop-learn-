# PAL / Program-of-Thoughts（把會算錯的子步驟外包給直譯器）

- **族別**：五之補（oracle 家族的**事前**形態）
- **控制骨架**：`推理歸 LLM → 可計算的子步驟寫成 code → 直譯器真的執行 → 結果代回推理鏈`
- **核心轉變**：[05 until oracle passes](../05-until-oracle-passes.md) 是「做完之後用 oracle 驗」；PAL 是「一開始就不讓不可靠的子步驟由模型執行」——外部確定性從驗證端提前到生成端。

## Prompt 範例

基本形（PAL / Program-of-Thoughts）：

```text
解 {{PROBLEM}}。

規則：
1. 用自然語言推理「該算什麼、為什麼」。
2. 凡是算術、日期、單位換算、計數、排序、統計——一律寫成 Python 交給直譯器，
   不准心算，不准在腦內模擬執行結果。
3. 拿到直譯器的真實輸出後，才能繼續下一步推理。
4. 最後輸出 FINAL: <答案>，並列出哪些中間值來自程式執行（可稽核）。
```

CRITIC 變體（工具 ground 的批判迴圈）：

```text
你剛給出以下答案：{{DRAFT}}

現在批判它——但每一條批判都必須先用工具驗證過才准寫：
- 事實宣稱 → 檢索查證
- 數值 → 寫 code 重算
- 程式行為 → 真的執行
沒有工具證據的批判不要寫。輸出：[{claim, tool_used, evidence, verdict}]
```

## 為什麼這樣做可以

- **LLM 的心算是機率性的，直譯器是確定性的**。多位數乘法、日期差、字串計數是 LLM 的經典失誤區；把這些子步驟外包，等於把錯誤率最高的環節換成錯誤率為零的引擎——分工各用其長。
- **code 是可稽核的中間產物**：心算錯了無從發現，code 錯了看得見、測得了。錯誤從「隱形」變「可檢查」。
- **CRITIC 的補充**：本集反覆強調「無外部訊號的自我批判會退化」（[10 self-refine](../10-self-refine.md) 的警訊）；CRITIC 就是把每一條批判都強制接上工具訊號——批判品質不再取決於模型的自我感覺。

## 什麼時候用 / 不要用

- ✅ 題目含算術、日期、單位、計數、統計、字串處理等可程式化子步驟。
- ✅ 資料分析類任務（讓模型寫 pandas 而不是「看著表格心算」）。
- ✅ 自我批判需要 ground（CRITIC 形態）。
- ❌ 純語意判斷、無可程式化部分的任務——外包沒有著力點。
- ⚠️ code 本身也會寫錯——但 code bug 可測可稽核，比心算錯誤好抓一個量級。

## 陷阱

- **模型「假裝執行」**：最危險的失效——模型自己編一個「執行結果」。直譯器必須真的存在於 loop 裡（tool call / sandbox），不能讓模型自報輸出。
- **整題塞給 code**：語意推理也硬寫成程式，可讀性與正確率反而下降。分工原則：判斷歸 LLM、計算歸 code。
- **直譯器沙箱**：執行模型生成的 code 必須在沙箱裡（配 [51 sandbox-verify-commit](51-sandbox-verify-commit.md)）。

## 來源對應

- [PAL: Program-aided Language Models（arXiv:2211.10435）](https://arxiv.org/abs/2211.10435)、[Program-of-Thoughts（arXiv:2211.12588）](https://arxiv.org/abs/2211.12588)、[CRITIC（arXiv:2305.11738）](https://arxiv.org/abs/2305.11738)。
- [agentic-loop.md 一之補](../../agentic-loop.md) 早已點名「CRITIC 精神」（self-refine 必接外部工具），本篇是它的完整範例。
- 互補：[05 until oracle passes](../05-until-oracle-passes.md)（事後驗證）——事前外包 + 事後驗證是同一主線的兩半。
