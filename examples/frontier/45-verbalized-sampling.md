# Verbalized Sampling（口頭化抽樣：把多樣性生出來，再進 select）

- **族別**：二之補（generate-then-select 的**生成端**）
- **控制骨架**：`要求「N 個答案＋各自的機率」→ 逼模型吐出分佈的尾巴 → 候選才有多樣性可挑`
- **解決的問題**：對齊後的模型有 **mode collapse**——同一題就算 temperature 調高、狂 sample 十次，答案仍高度趨同。整個 generate-then-select 族（[06](../06-self-consistency.md)/[07](../07-best-of-n.md)/[08](../08-tournament.md)/[32](32-alphacode-style.md)）的前提「候選夠多樣」因此默默失效。

## Prompt 範例

```text
針對 {{TASK，例：這款產品的 tagline}}，不要直接給我「最好的一個」。

改成：生成 5 個彼此顯著不同的候選，並為每個標注你估計的機率
（＝如果從你的完整分佈隨機抽樣，抽到這類答案的比例）。

要求：
- 5 個裡至少 2 個要來自分佈的「尾巴」——冷門但合理的方向，
  不是最典型答案的改寫。
- 輸出：[{candidate, probability, 為什麼這個方向不同}]
```

下游接 select（完整的 generate-then-select）：

```text
（把 5 個候選餵給 judge / tournament / 人挑——見 07 / 08）
```

## 為什麼這樣做可以

- **Mode collapse 的成因是 typicality bias**：偏好標註時人類系統性偏愛「熟悉的文本」，RLHF 於是把機率質量壓到眾數上。但模型內部其實仍「知道」整個分佈——直接問只會得到眾數，**要求口頭化分佈**則繞過了坍縮，訓練不用改。
- **實測**：創意寫作多樣性提升 1.6–2.1×、人評 +25.7%，事實準確性與安全性不降。
- **接上本集 meta 洞察**：「多樣性 > 規模」——與其同一個 prompt 狂 sample 20 次（20 個眾數的近親），不如一次要 5 個「被強制拉開的」候選。

## 什麼時候用 / 不要用

- ✅ 創意生成、brainstorm、命名、設計方向——解空間寬的任務。
- ✅ 任何 best-of-N / tournament 的前置：先確保 N 個候選真的不同。
- ✅ 合成資料生成（多樣性直接決定資料品質）。
- ❌ 有唯一正解的題——多樣性沒有意義，直接用 [06 self-consistency](../06-self-consistency.md)。
- ⚠️ 與 [46 一致性棄答閘](46-consistency-abstention.md) 恰好相反：46 把「答案發散」當警報，45 把它當資源。分清楚你的任務要哪一種。

## 陷阱

- **只拿第一個候選**：等於沒做——第一個通常仍是眾數。價值在尾巴。
- **把口頭機率當校準信心**：那些機率是模型的口頭估計，不是校準過的統計量。拿來「拉開候選」有效，拿來做風險決策不行。
- **N 開太大**：品質隨 N 稀釋，5–10 是論文的甜蜜點。

## 來源對應

- [Verbalized Sampling: How to Mitigate Mode Collapse（arXiv:2510.01171）](https://arxiv.org/abs/2510.01171)。
- [agentic-loop.md 統合框架](../../agentic-loop.md)——「多樣性 > 純粹重複取樣」的關鍵洞察；本篇是把該洞察落到生成端的工具。
- 下游接：[07 best-of-N](../07-best-of-n.md)、[08 tournament](../08-tournament.md)、[25 MoA](25-mixture-of-agents.md)。
