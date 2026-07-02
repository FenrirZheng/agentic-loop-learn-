# Adversarial verify（每個發現派 N 個 skeptic 專門推翻它）★ 最常推薦

- **族別**：三、Adversarial / Critique
- **控制骨架**：`對每個 finding → 派 N 個被指示去「推翻」它的 skeptic → 多數推翻就丟掉`
- **重點**：驗證者的預設立場是「這條是錯的，除非你能證明它對」。

## Prompt 範例

對每一個候選 finding，獨立跑 3 個 skeptic（各自獨立）：

```text
[SKEPTIC k/3]
有人聲稱以下這條是一個真實的 bug：
  「{{finding：檔案:行 + 描述}}」

你的任務是「推翻它」。預設它是假警報，除非你找到不可辯駁的理由說明它為真。
請具體：
- 給出一條「這條其實不成立」的具體反駁（例如：該路徑不可達 / 已有防護 / 輸入不可能是那個值）。
- 只有當你真的無法推翻時，才回報 refuted=false，並附上「它為真」的最小重現條件。
輸出：{ refuted: true|false, reason: "..." }
```

裁決（程式或彙整呼叫）：3 票裡 ≥2 票 `refuted=true` → 丟棄該 finding。

## 為什麼這樣做可以

- **擋掉「聽起來合理但其實錯」的結論**——這是來源文件說它「最常推薦」的核心價值。LLM 找 bug 時很會產出貌似有理的假警報；把驗證者的預設立場反轉成「先假定是假的」，等於用舉證責任逼真問題現形、讓假問題被淘汰。
- **多個獨立 skeptic + 多數決**讓單一 skeptic 的失誤不致誤殺或漏放；獨立性確保它們不互相汙染。
- **「預設 refuted=true，除非…」的框架**是關鍵——若只是中性地問「這對嗎」，模型會傾向附和原始 finding。反轉預設值才有對抗力。

## 什麼時候用 / 不要用

- ✅ 任何「發現型」產出需要把關：bug 清單、安全風險、事實宣稱、審計發現。
- ✅ 想壓低 false positive（寧可漏放邊緣案例也不要一堆假警報）時。
- ⚠️ 若你更怕「漏抓」，這招會偏保守——搭配 [loop-until-dry](04-loop-until-dry.md) 先廣撒網再用它篩。

## 進階：視角多元化的驗證

當一個 finding 可能以「不只一種方式」出錯時，別派 N 個一模一樣的推翻者，改給每個 skeptic **不同的鏡頭**（正確性 / 安全 / 能不能真的重現）——多樣性能抓到冗餘抓不到的失效模式。

## 來源對應

- [agentic-loop.md 三、Adversarial / Critique](../agentic-loop.md) — Adversarial verify（「最常推薦」）。
- 常見組合：fan-out 找 bug → [loop-until-dry](04-loop-until-dry.md) → 每個 bug 用 3 票對抗驗證 → [judge panel](09-judge-panel.md) 綜合。
