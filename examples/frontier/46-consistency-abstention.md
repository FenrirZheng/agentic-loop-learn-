# 一致性棄答閘（sample 發散 = 警報：棄答 / 升級 / 標「未驗證」）

- **族別**：三之補（沒有 oracle 時的不確定度估計）
- **控制骨架**：`同題獨立 sample N 次 → 按語意分群 → 群太散 → 棄答或升級；夠聚 → 交付`
- **與 06 的關係**：和 [06 self-consistency](../06-self-consistency.md) 用**同一個積木**（sample + 比對），用途相反——06 拿一致性**挑答案**，本篇拿**不一致性當警報**。「答 vs 不答」是一個現有各招都沒有的輸出策略：所有招都假設最後要交出一個答案。

## Prompt 範例

第一步：獨立 sample（互不可見）：

```text
[RUN k/5，fresh context]
{{QUESTION}}
只輸出答案本身，不要解釋。
```

第二步：語意分群（注意：是語意等價，不是字面相同）：

```text
以下是同一個問題的 5 份獨立回答：
{{ANSWERS}}

把它們按「語意等價」分群——說法不同但意思相同的算同一群。
輸出：[{group_answer, members: [...], size}]
```

第三步：程式裁決（不是模型裁決）：

```text
最大群 ≥ 4/5 → 交付該答案
最大群 = 3/5 → 交付但標註 [低信心]
最大群 ≤ 2/5 → 棄答：輸出「無法可靠回答」＋升級（人工 / 更強模型 / 檢索）
```

逐句版（SelfCheckGPT，適用長文）：對草稿的**每一句**，問「其他 4 份 sample 是否支持這句」——不被支持的句子逐句標記或刪除。

## 為什麼這樣做可以

- **幻覺（confabulation）的行為特徵是不穩定**：模型真的知道時，N 次回答聚成一群；不知道而編造時，答案在語意空間亂散。**發散度是「模型不知道」的可觀察 proxy**——不需要 ground truth 就能量。
- **語意層級是關鍵**：Nature 2024 證明按意義分群後算熵（semantic entropy），比 token 層級的熵準得多——「巴黎」和「法國的首都巴黎」是同一個答案。
- **棄答的價值**：高風險場景中，錯答的成本 >> 不答的成本。一個會說「我不確定」的系統，比一個永遠自信的系統可信。

## 什麼時候用 / 不要用

- ✅ 事實型 QA、無 oracle 又高風險的輸出、「自動答 vs 升級人工」的分流閘。
- ✅ 給答案掛信心標籤（呼應 CLAUDE.md 式的 [oracle]/[判斷]/[未驗證] 紀律）。
- ❌ 創意任務——答案發散是 feature 不是 bug（那是 [45](45-verbalized-sampling.md) 的地盤）。
- ⚠️ **偵測不到「一致的錯」**：模型自信地系統性答錯時，5 份全聚在同一個錯誤上。一致只是必要條件，不是正確的證明。

## 陷阱

- **字面比對代替語意分群**：同義答案被算成不同群，熵虛高、狂棄答。
- **N 太小**：3 份估不出散度；5 是底線，重要決策用 10。
- **sample 之間互相可見**：後面的 run 抄前面的，發散度假性收斂——必須 fresh context。
- **把「通過閘門」當「驗證過」**：這是不確定度過濾，不是 oracle；能配 [05](../05-until-oracle-passes.md)/[47](47-relation-based-verification.md) 就再驗。

## 來源對應

- [Detecting hallucinations in LLMs using semantic entropy（Nature 2024）](https://www.nature.com/articles/s41586-024-07421-0)、[SelfCheckGPT（arXiv:2303.08896）](https://arxiv.org/abs/2303.08896)、[Semantic Entropy Probes（arXiv:2406.15927）](https://arxiv.org/abs/2406.15927)。
- 同積木三用途：[06](../06-self-consistency.md) 挑答案 / **46 拉警報** / [49](49-clarify-before-act.md) 測歧義。
- 下游：散度太高時升級 → [36 cascade routing](36-cascade-routing.md)。
