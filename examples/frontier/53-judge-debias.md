# Judge 去偏協定（換位、盲評、judge≠generator：先修裁判，再信裁決）

- **族別**：二 / 三之補——不是新招，是所有用到 judge 的招（[07](../07-best-of-n.md)/[08](../08-tournament.md)/[09](../09-judge-panel.md)/[23](23-evaluator-optimizer.md)/[33](33-rubric-based-judge.md)/[38](38-weighted-voting.md)）的**衛生規範**
- **控制骨架**：`pairwise 必換位跑兩次（結果不一致 = 平手）→ judge 與 generator 用不同模型 → 盲評（剝除來源特徵）→ rubric 錨定`
- **為什麼存在**：本集 meta 洞察 2 說「回饋訊號品質決定一切」——judge 是至少 6 招的回饋源，而 judge 本身有**系統性、方向固定**的偏誤。裁判壞了，上面疊的每一招都在放大錯誤訊號。

## Prompt / 流程範例

換位對消（position bias 是最大宗）：

```text
[PASS 1] 比較以下兩個回答，哪個更好？
回答一：{{A}}   回答二：{{B}}
輸出：WINNER: 一|二|平手 ＋ 理由

[PASS 2，fresh context] （同一 prompt，但 A、B 對調位置）
```

```python
# 程式裁決：兩次都指向同一個真實候選才算勝，否則記平手
verdict = "A" if (p1 == "一" and p2 == "二") else \
          "B" if (p1 == "二" and p2 == "一") else "TIE"
```

其他三條紀律（寫進 judge prompt / 流程）：

```text
- 長度偏誤：rubric 裡明示「長度與品質無關；冗長而無增量內容應扣分」。
- self-preference：judge 模型 ≠ 生成候選的模型（至少換家族）。
- 盲評：餵給 judge 前剝除模型名、系統簽名、可辨識的格式慣性。
- rubric 錨定：每個分數段給一個具體範例，防止分數在批次間漂移
  （見 33 rubric-based judge）。
```

## 為什麼這樣做可以

- **偏誤方向固定，所以可以用對稱化程序抵消**。MT-Bench 系統性測出：position bias（偏好先出現的候選，連 GPT-4 都有）、verbosity bias（偏好長答案）、self-enhancement bias（偏好自己生成的文本）。隨機誤差靠多 sample 平均；**系統性偏誤靠結構對消**——換位取交集正是如此。
- **「不一致 = 平手」比「取平均」誠實**：兩次換位給出矛盾判決，代表訊號本身不可靠，硬取一邊是在把雜訊當訊號。
- **成本效益極高**：換位只是 ×2 呼叫，卻能消掉最大宗的偏誤來源——相比之下，在有偏 judge 上疊更多輪 refine 是負收益。

## 什麼時候用 / 不要用

- ✅ 任何 pairwise judge：[08 tournament](../08-tournament.md) 每一場 PK、[07 best-of-N](../07-best-of-n.md) 的決選。
- ✅ 評測 / 迴歸場景（[42 prompt optimization loop](42-prompt-optimization-loop.md) 的 eval set 打分）——偏誤會直接汙染優化方向。
- ❌ 有硬 oracle 時——直接用 oracle，別用 judge（judge 是沒 oracle 時的替代品，不是升級品）。
- ⚠️ 大規模 tournament 全面換位成本 ×2——可對關鍵場次（決賽圈）全換位、初賽抽樣稽核。

## 陷阱

- **單次 pairwise 就宣布勝者**：position bias 足以翻轉相近候選的勝負。
- **同一個模型自產自評**：self-preference 讓分數整體虛高且偏向自家文風。
- **把 judge 分數當絕對品質**：它是相對偏好的近似，跨批次、跨 rubric 不可比。
- **rubric 沒有錨定範例**：同一份 rubric 兩天打出兩種分佈。

## 來源對應

- [Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena（arXiv:2306.05685）](https://arxiv.org/abs/2306.05685)——position / verbosity / self-enhancement bias 的系統性實測。
- 被本篇保護的招：[07](../07-best-of-n.md)、[08](../08-tournament.md)、[09](../09-judge-panel.md)、[23](23-evaluator-optimizer.md)、[33](33-rubric-based-judge.md)、[38](38-weighted-voting.md)。
- 本質：[meta 洞察 2](../../index.md#三個要記住的-meta-洞察)（回饋訊號品質決定成敗）應用在回饋源自身。
