# until threshold（自評分數達標才停）

- **族別**：一、迴圈類
- **控制骨架**：`repeat { 產出; 用 rubric 自評分數 s } until s ≥ X`
- **停止條件**：可量化的品質指標（覆蓋率、可讀性分數…）達到門檻 X。

## Prompt 範例

```text
任務：把下面函式的 docstring 補到「品質達標」。

評分量表（每項 0–2，滿分 10）：
- 參數說明完整（每個 arg 都有型別與意義）
- 回傳值說明
- 例外／錯誤情況說明
- 一個可執行的使用範例
- 邊界條件註記

迴圈規則：
1. [DRAFT] 產出 docstring。
2. [SCORE] 依上面 5 項逐項打分並加總，列出「扣分項具體是哪裡不足」。
3. 若總分 < 9：針對「扣分項」定向修正，回到步驟 1。
4. 若總分 ≥ 9：輸出 [PASS @ score] 並停。

不准為了達標而虛報分數；扣分理由必須指向文字裡具體的缺漏。
```

## 為什麼這樣做可以

- **把「夠好了嗎」從模糊感覺變成逐項量表**。單純問「這樣可以嗎」模型幾乎永遠說可以；給它一張 rubric 並要求「扣分理由要指到具體位置」，自評才有鑑別力（這是 rubric-based judge 的精神，見 [rubric 範例](frontier/33-rubric-based-judge.md)）。
- **要求「針對扣分項定向修正」**避免每輪全篇重寫——只補不足的地方，收斂更快也更穩。
- 比 [until budget](02-until-budget.md) 更貼近「品質」目標：停止點綁在可衡量的分數而非輪數。

## 什麼時候用 / 不要用

- ✅ 有可量化品質面向的任務：測試覆蓋率、可讀性 checklist、文件完整度。
- ❌ 分數本身沒有外部依據時要小心——見下方陷阱。

## 陷阱：自評分數會膨脹

門檻法的死穴是**模型替自己打分容易灌水**。三個緩解：
1. rubric 越具體、越可指認位置，灌水空間越小。
2. 讓「打分的」和「產出的」分成兩個角色/兩次呼叫（見 [evaluator-optimizer](frontier/23-evaluator-optimizer.md)）。
3. 只要能換成客觀 oracle（真的跑 coverage 工具、真的跑測試），就別靠自評——見 [until oracle passes](05-until-oracle-passes.md)。

## 來源對應

- [agentic-loop.md 一、迴圈類](../agentic-loop.md) — until threshold（「有可量化品質指標」）。
- 雙 agent 評改版：[evaluator-optimizer](frontier/23-evaluator-optimizer.md)。
