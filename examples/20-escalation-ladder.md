# Escalation ladder（先用便宜方法，失敗才升級到貴的）

- **族別**：五、Search / 動態策略
- **控制骨架**：`先試便宜的 → 若信心/結果不足 → 才升級到貴的`
- **重點**：成本階梯——便宜模型 → 貴模型；簡單 prompt → 完整 prompt。

## Prompt 範例

第一階（便宜、快）：

```text
[TIER 1 · 便宜]
用最簡潔的方式回答：{{QUESTION}}
答完後自評：CONFIDENCE = high | low，並說明「哪裡讓你不確定」。
若你判斷這題需要查資料/多步推理才能可靠回答，直接輸出 ESCALATE。
```

升級判準（由程式或外層依 TIER 1 輸出決定）：
- `CONFIDENCE = high` → 採用，結束。
- `CONFIDENCE = low` 或 `ESCALATE` → 丟到第二階：

```text
[TIER 2 · 貴]
（TIER 1 對這題信心不足）
用完整方法回答：{{QUESTION}}
允許多步推理、可調用工具/檢索、給出依據。務求可靠。
```

## 為什麼這樣做可以

- **大部分請求其實很簡單**，用最強最貴的設定去打所有題目是浪費。階梯法讓便宜路徑先接住多數簡單題，只有真正難的才付昂貴成本——在總體品質幾乎不降的前提下大幅省錢。
- **關鍵是 gate 的訊號要可靠**：讓 tier 1 自報信心、或用「多個小模型不一致才升級」（agreement-based）比讓模型隨口說「我很確定」更穩。
- **「哪裡讓你不確定」**這句逼模型定位不確定來源，讓升級決策不是拍腦袋。

## 什麼時候用 / 不要用

- ✅ 成本敏感、請求量大、難度分布不均（多數簡單、少數困難）。
- ✅ 有明確的便宜/昂貴兩檔（或多檔）可切換。
- ❌ 每題都難、或錯誤代價極高不容便宜路徑冒險——直接上最強檔。

## 進階：cascade routing

這招的成本量化版是 **model cascade / routing**：用信心/一致性/conformal-prediction 門檻決定升級點，甚至能給出形式化的成本保證。見 [cascade routing 範例](frontier/36-cascade-routing.md)。

## 來源對應

- [agentic-loop.md 五、Search / 動態策略](../agentic-loop.md) — Escalation ladder；量化版見〈五之補〉Model cascade / routing。
