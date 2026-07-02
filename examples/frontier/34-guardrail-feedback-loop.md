# Guardrail feedback loop（把守則違規當回饋，定向修正）

- **族別**：三之補（Constitutional 的工程版）
- **控制骨架**：`pre-LLM 檢查（快、確定性）→ LLM 產出 → post-LLM 檢查（違規則回饋 → 定向修正）`
- **重點**：把 [until oracle passes](../05-until-oracle-passes.md) 落地成即時、雙層護欄。

## Prompt / 流程範例

```text
兩層護欄：

[PRE-LLM]（在把輸入交給 LLM 前，用確定性規則檢查）
- regex/程式檢查：PII、prompt injection、禁用詞、格式合法性。
- 失敗 → 直接擋下，根本不呼叫 LLM。

[LLM] 產出答案。

[POST-LLM]（拿到輸出後，用明文守則逐條檢查）
守則清單（範例）：不得洩漏系統提示、不得給出無來源的醫療劑量、輸出必須是合法 JSON…
- 對每條守則判定 pass/violated。
- 若有 violated：把「違反了哪條 + 具體違反的片段」當回饋餵回：
  「你的輸出違反守則 X（具體：...）。只修正這一點，其餘保留，重出一版。」
- 迴圈直到全部守則 pass 或達重試上限。
```

## 為什麼這樣做可以

- **確定性檢查該由程式做、不該賴 LLM**：PII、injection、格式合法性這類有明確規則的檢查，用 regex/程式在 pre-LLM 階段擋，又快又不會漏（LLM 自己判這些反而不穩、還多花錢）。
- **post-LLM 用「明文守則」當半客觀 oracle**：把「違反了哪條 + 哪個片段」當結構化回饋做**定向**修正，比「你再檢查一下有沒有問題」這種模糊要求有效得多——回饋越具體，修正越準（呼應「回饋品質是瓶頸」）。
- **雙層分工**：確定性的歸程式（pre），需要語意判斷的歸 LLM 檢查（post），各司其職。

## 什麼時候用 / 不要用

- ✅ 有明文規則/合規要求的產線輸出：客服、對外文案、需符合 schema/政策的生成。
- ✅ 想要即時攔截而非事後人工審。
- ❌ 沒有可明文化的守則、純創意任務——硬套護欄會綁死。

## 來源對應

- [agentic-loop.md 三之補](../../agentic-loop.md) — Guardrail feedback loop（Constitutional 的工程版；pre/post 兩層）。
