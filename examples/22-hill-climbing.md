# Hill-climbing（只接受「比現在好」的修改，否則丟棄）

- **族別**：五、Search / 動態策略
- **控制骨架**：`repeat { 提一個修改 → 評分 → 若比現況好就採納，否則丟棄 }`
- **與 self-refine 差別**：self-refine 每輪都接受新版本；hill-climbing 只在「確實變好」時才接受，變差就退回。

## Prompt 範例

```text
任務：把這個函式的效能改到更好，但不能破壞行為。
評分器（oracle）：`pytest`（必須全綠）+ `python bench.py`（回報 ops/sec）。

爬山規則：
- 維護 [CURRENT BEST]：目前最好的版本 + 它的 benchmark 分數（初始 = 原版）。
- 每一輪：
  1. [PROPOSE] 對 CURRENT BEST 提「一個」具體優化改動。
  2. 我會跑 oracle 回填：測試是否全綠 + 新的 ops/sec。
  3. [DECIDE] 若「測試全綠 且 ops/sec > CURRENT BEST」→ 採納為新的 CURRENT BEST。
     否則 → 丟棄這個改動，CURRENT BEST 不變，換一個「不同方向」的改動再試。
- 連續 3 次提案都無法超越 CURRENT BEST → 停，輸出 CURRENT BEST。
```

## 為什麼這樣做可以

- **保證單調不退步**：self-refine 的風險是「改著改著把好的改壞了」。Hill-climbing 用「只接受更好的、變差就退回」把每一步都鎖成「不會比現在差」，特別適合有明確評分的優化任務。
- **必須有客觀評分器才成立**：「比現在好」要能被 oracle 客觀判定（這裡是 benchmark + 測試）。這也讓它避開了 self-refine「無外部訊號會退化」的陷阱——它的接受判準本來就綁在外部 oracle 上。
- **失敗就換方向**避免在同一個無效改動上鬼打牆。

## 什麼時候用 / 不要用

- ✅ 有客觀、可反覆量測的評分：效能優化、壓縮率、可量測的品質指標。
- ✅ 想要「只進不退」的保證。
- ❌ 沒有客觀評分器——「比現在好」變成模型自評，就退化成不可靠的 self-refine。
- ⚠️ 會卡在**局部最優**（只接受鄰近的小改進，跳不出當前山頭）。想跳出要靠：偶爾接受變差的走法（模擬退火思路）、或改用能回溯探索的 [tree search](21-tree-search.md)。

## 來源對應

- [agentic-loop.md 五、Search / 動態策略](../agentic-loop.md) — Hill-climbing。
- 需要跳出局部最優、可回溯的版本：[tree search / LATS](21-tree-search.md)。
