# Mixture-of-Agents（MoA，分層聚合）

- **族別**：二之補（generate-then-select 的新招）
- **控制骨架**：`第 1 層多個 agent 各自產出 → 第 2 層每個 agent 讀「上一層全部輸出」再重寫 → 最後 aggregator 綜合`
- **重點**：多層、每層都參考上一層的**全部**，最後由 aggregator「綜合」而非「挑一個」。

## Prompt 範例

第 1 層（多個異質 proposer，各自獨立）：

```text
[LAYER 1 · proposer {{i}}] 針對 {{TASK}} 給出你的最佳答案 + 理由。
（各 proposer 用不同角度/風格/側重，刻意求異質。）
```

第 2 層（每個 agent 讀第 1 層全部再重寫）：

```text
[LAYER 2] 以下是多個獨立答案：{{all_layer1}}
綜合它們的優點、修掉各自的缺點，重寫出一個更好的版本。
不要只挑一個——要吸收多方長處。
```

Aggregator（最終綜合）：

```text
[AGGREGATOR] 以下是第 2 層各版本：{{all_layer2}}
產出最終答案：整合最強論點、去除彼此矛盾、補齊各版都漏的點。
```

## 為什麼這樣做可以

- **aggregator 綜合 > 單純 ranker 挑一個**：來源文件明確指出，讓一個彙整者「讀完全部再重寫綜合」比只從候選裡選最好的一個效果好——因為好答案的不同優點常分散在不同候選裡，綜合能把它們嫁接起來。
- **異質 proposer + 逐層參考全部**體現「多樣性 > 純規模」：多個角度不同的 agent，勝過對同一最強模型狂 sample。
- 分層讓「發散（第 1 層）→ 融合（第 2 層）→ 定稿（aggregator）」逐步收斂。

## 什麼時候用 / 不要用

- ✅ 開放式、解空間寬、值得多方觀點融合的任務（設計、寫作、複雜分析）。
- ✅ 願意付多層呼叫成本換品質。
- ❌ 有唯一解 → [self-consistency](../06-self-consistency.md) 投票更省。
- ❌ 成本敏感的簡單任務——分層太貴。

## 來源對應

- [agentic-loop.md 二之補](../../agentic-loop.md) — Mixture-of-Agents（MoA）。
- 單層多角度版：[judge panel](../09-judge-panel.md)。
