# Weighted / confidence-weighted voting（信心加權的多數決）

- **族別**：二之補（generate-then-select 的新招）
- **控制骨架**：`獨立生 N 個答案，各自附信心 → 依信心加權計票，取加權最高`
- **與純多數決差別**：不是一票一等值，而是「較有把握的答案權重更高」。

## Prompt 範例

生成階段（每個獨立答案都要自報信心）：

```text
題目：{{PROBLEM}}
一步步推理，最後輸出：
FINAL: <答案>
CONFIDENCE: <0.0–1.0，並用一句話說明信心來源；若你其實是猜的就給低分>
```

彙整（加權計票）：

```text
以下是 N 組 (FINAL, CONFIDENCE)：{{list}}
規則：
- 把相同的 FINAL 歸為一組，該組權重 = 該組所有 CONFIDENCE 之和。
- 輸出加權總和最高的答案，附各答案的加權分佈。
- 若第一名與第二名的加權差距很小，標記 [CLOSE]。
```

## 為什麼這樣做可以

- **不是每個答案都一樣可靠**：純多數決把「瞎猜矇到的」和「有把握推出的」等值計票。信心加權讓模型自陳的把握度參與計票，理論上讓可靠答案更容易勝出。
- **但要務實看待增益**：來源文件明確指出，**self-consistency 的多數決本身就吃掉了大部分增益**——加權是錦上添花，別期待它帶來質變。它的價值在「勢均力敵時多一個判準」。
- 要求「猜的就給低分」是為了讓 confidence 有鑑別力，否則模型傾向對每個答案都報高信心，加權就退化回等值投票。

## 什麼時候用 / 不要用

- ✅ 已在用 [self-consistency](../06-self-consistency.md)、且模型的自陳信心有一定鑑別力時，當作小幅增強。
- ❌ 模型信心校準很差（總是報高分）——加權沒意義，用純多數決即可。
- ❌ 開放式答案無法歸組——用 [universal self-consistency](37-universal-self-consistency.md)。

## 來源對應

- [agentic-loop.md 二之補](../../agentic-loop.md) — Weighted / confidence-weighted voting。
