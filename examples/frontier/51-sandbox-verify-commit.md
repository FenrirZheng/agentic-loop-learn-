# Sandbox-verify-commit（交易式執行：隔離跑 → oracle 驗 → 過了才 commit）

- **族別**：五之補（副作用世界的控制結構）
- **控制骨架**：`隔離環境執行（git worktree / sandbox）→ oracle 驗證 → pass 才 commit / merge → fail 整包丟棄，主線零汙染`
- **與 21 的關係**：[21 tree search / backtracking](../21-tree-search.md) 的回溯只能撤銷「想法」；本篇讓**已執行的動作**也可撤銷——現有各招都活在無副作用的文字空間，這是把回溯能力帶進「會改檔案、會碰系統」的真實世界。

## Prompt / 流程範例

單 agent 交易式改檔：

```bash
# 1. 隔離：開一個 worktree，agent 只准在裡面動手
git worktree add /tmp/agent-tx-1 -b agent/fix-parser

# 2. 執行：agent 在隔離區完成修改

# 3. 驗證（oracle，不是模型自評）
cd /tmp/agent-tx-1 && go test ./... && golangci-lint run

# 4a. 綠燈 → commit / merge 回主線
# 4b. 紅燈 → 整個 worktree 丟棄，主線從頭到尾沒被碰過
git worktree remove --force /tmp/agent-tx-1
```

Claude Code Workflow 中這是一個參數：

```js
// 多 agent 並行改檔：各給一個 worktree，互不衝突；
// 各自過驗證後才合併，failed agent 的殘骸自動清掉
const results = await parallel(tasks.map(t => () =>
  agent(t.prompt, { isolation: 'worktree' })))
```

Agent prompt 裡把交易邊界講死：

```text
你在隔離的 worktree 裡工作。規則：
1. 完成後必須跑 {{ORACLE_CMD}}，把完整輸出貼回來。
2. 綠燈才報告成功；紅燈就修，修不好就報告失敗——
   失敗的一切都會被丟棄，所以不要為了「看起來完成」而繞過驗證。
```

## 為什麼這樣做可以

- **交易語意讓失敗變便宜**：失敗的成本從「收拾殘局」降到「rm -rf 一個目錄」。失敗便宜，agent 才敢嘗試、你才敢放手。
- **驗證在隔離中進行**：驗證不過的中間態永遠不會汙染主線——對照「先改主線再想辦法驗」的裸奔模式。
- **並行的前提**：N 個 agent 改同一個 repo 唯一安全的方式就是各給一個隔離副本，合併點做衝突與驗證裁決。
- 這是 [05 until oracle passes](../05-until-oracle-passes.md) 的副作用版：oracle 不只決定「停不停」，還決定「這些副作用要不要被保留」。

## 什麼時候用 / 不要用

- ✅ agent 改檔案 / 改設定 / 跑遷移——任何有副作用且有 oracle 的任務。
- ✅ 多 agent 並行寫同一個 codebase。
- ✅ 配 [50 dry-run](50-dry-run-premortem.md)：模擬過了，真執行仍走交易。
- ❌ 純文字任務——沒有副作用就不需要交易。
- ⚠️ 副作用出得了 sandbox 的（已發送的 email、外部 API 呼叫）**無法回滾**——這類動作 sandbox 保護不了，只能靠 [50](50-dry-run-premortem.md) 的事前閘。

## 陷阱

- **驗證放在 commit 之後**：本末倒置——merge 進主線才發現紅燈，交易語意全失。
- **sandbox 與真實環境 drift**：隔離區缺 env var / 服務 / 資料，過了 sandbox 掛在真環境。定期校準兩者。
- **commit 邊界含糊**：「哪個 oracle 綠了才算數」要寫死在流程裡，不能讓 agent 自由心證。
- **忘記清理**：失敗的 worktree / sandbox 堆積吃磁碟——清理是交易的一部分。

## 來源對應

- [Anthropic — Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)（sandboxing 與 guardrails）；2026 生產模式綜述（bounded execution）如 [Agentic Design Patterns catalog](https://www.augmentcode.com/guides/agentic-design-patterns)。Claude Code 的 worktree isolation 即此模式的產品化。
- 閘門鏈：[49](49-clarify-before-act.md) → [50](50-dry-run-premortem.md) → **51**；oracle 精神：[05](../05-until-oracle-passes.md)。
