# until oracle passes（外部驗證通過才停）★ 最可靠

- **族別**：一、迴圈類
- **控制骨架**：`while oracle(產出) 不通過: 讀失敗訊息 → 定向修正`
- **停止條件**：**外部**客觀驗證通過——測試綠燈、編譯過、lint 乾淨、schema 驗證過。

## Prompt 範例

```text
任務：實作 `parse_duration(s: str) -> int`（把 "1h30m" 這種字串轉成秒數）。

Oracle（唯一的停止判準，不是你的自我感覺）：
  執行 `pytest tests/test_duration.py -q`

迴圈規則：
1. [PATCH] 寫出／修改實作。
2. 我會幫你跑 oracle，把「完整的失敗輸出」貼回給你。
   （若你在有工具的環境，請自己跑並貼出真實輸出，不准腦補測試結果。）
3. [DIAGNOSE] 只針對「第一個 failing test 的實際報錯」定位根因，不要一次亂改一堆。
4. 回到步驟 1，直到 oracle 全綠。
5. 全綠後輸出 [ORACLE PASS] 並附上最後一次真實的測試輸出。

硬規則：在看到「真實的綠燈輸出」前，絕不宣稱完成。
```

## 為什麼這樣做可以

- **停止條件客觀，不靠模型自我感覺**——這是來源文件說「最強的一種」的理由。收斂/門檻/自評都可能停在「看起來對其實錯」；oracle 是外部 ground truth，模型騙不了編譯器和測試。
- 這也是**唯一讓 self-refine 迴圈安全的前提**：來源文件的重大警訊指出，沒有外部訊號時純內省會「越改越差」。接上 oracle（測試/工具/檢索）等於給迴圈一個誠實的梯度。
- **「只修第一個 failing test 的實際報錯」**避免散彈式亂改：逐一擊破比一次改一堆更快收斂，也更好定位是哪個改動修好了問題。
- **「看到真實綠燈才宣稱完成」**是防幻覺的護欄——模型很愛腦補「測試應該會過」。

## 什麼時候用 / 不要用

- ✅ 任何能弄到客觀驗證器的任務：寫 code（測試/編譯/型別）、產 JSON（schema）、SQL（跑得動）、格式（linter）。
- ✅ 決策樹第一順位：**「有客觀 oracle → 就用它」**。
- ❌ 真的沒有外部驗證器的開放式任務（寫作、設計）——退而求其次用 [rubric judge](frontier/33-rubric-based-judge.md) 或 [best-of-N](07-best-of-n.md)。

## 進階：能逐步就用 PRM

比起「整份做完才驗」（ORM，結果獎勵），若能**逐步**驗證（每一步都給訊號，PRM，過程獎勵）會更強——例如每寫完一個函式就跑該函式的單元測試，而不是全部寫完才一次跑。逐步獎勵 > 結果獎勵。

## 來源對應

- [agentic-loop.md 一、迴圈類](../agentic-loop.md) — until oracle passes（「最強的一種」）。
- 「無外部訊號會退化」的警訊：見 [self-refine 範例](10-self-refine.md) 與來源〈一之補〉。
- 工程化即時護欄版：[guardrail feedback loop](frontier/34-guardrail-feedback-loop.md)。
