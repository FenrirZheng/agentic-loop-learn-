# AI 控制結構 Prompt 範例集

把 [agentic-loop.md](agentic-loop.md) 裡的每一種控制結構招數，各做成一個「可直接複製的 prompt 範例 + 為什麼這樣做可以」的檔案。這頁是總聚合。

- **怎麼讀**：先看下面的[決策樹](#怎麼選決策樹)決定用哪一族，再點進對應範例。
- **每個檔案的格式**：族別 → 控制骨架 → Prompt 範例 → **為什麼這樣做可以** → 何時用/不用 → 陷阱 → 來源對應。
- **一句話貫穿全書**：這些是積木，實務會疊著用；而**回饋 / 驗證的客觀性**是決定成敗的主線——沒有外部訊號的自我修正會退化。

---

## 一、迴圈類（差別只在「停止條件」）

| 範例 | 停止條件 | 一句話 |
|---|---|---|
| [01 until convergence](examples/01-until-convergence.md) | 本輪 ≈ 上一輪 | 改到穩定；但「收斂 ≠ 正確」 |
| [02 until budget](examples/02-until-budget.md) | 跑滿 N 輪 / token | 探索型任務的成本保險絲 |
| [03 until threshold](examples/03-until-threshold.md) | 自評分數 ≥ X | 有可量化品質指標時 |
| [04 loop-until-dry](examples/04-loop-until-dry.md) | 連續 K 輪無新發現 | 找 bug / 列 edge case，防長尾遺漏 |
| [05 until oracle passes](examples/05-until-oracle-passes.md) ★ | 外部驗證通過 | **最可靠**；測試綠燈 / 編譯過 |

## 二、Generate-then-Select（一次生多個再挑）

| 範例 | 挑選方式 | 一句話 |
|---|---|---|
| [06 self-consistency](examples/06-self-consistency.md) | 投票取眾數 | 有唯一解的推理題 |
| [07 best-of-N](examples/07-best-of-n.md) | judge 擇優 | 解空間寬的生成任務 |
| [08 tournament](examples/08-tournament.md) | 兩兩 PK 晉級 | 絕對評分難、相對比較易 |
| [09 judge panel](examples/09-judge-panel.md) | 多角度評審綜合 | 評價標準是多維的 |

## 三、Adversarial / Critique（引入對抗防自我感覺良好）

| 範例 | 對抗方式 | 一句話 |
|---|---|---|
| [10 self-refine](examples/10-self-refine.md) | 產出→自我批判→修正 | ⚠️ 無外部訊號會越改越差 |
| [11 debate](examples/11-debate.md) | 兩方辯論 + 裁判 | ⚠️ 常輸給 self-consistency，要引入不對稱 |
| [12 adversarial verify](examples/12-adversarial-verify.md) ★ | N 個 skeptic 推翻 | **最常推薦**；擋「合理但錯」 |
| [13 devil's advocate](examples/13-devils-advocate.md) | 對自己建議紅隊 | 採納建議前的壓力測試 |
| [14 completeness critic](examples/14-completeness-critic.md) | 專問「還缺什麼」 | 怕「漏」的任務 |

## 四、Decomposition（把大拆成可控的小）

| 範例 | 拆法 | 一句話 |
|---|---|---|
| [15 plan-then-execute](examples/15-plan-then-execute.md) | 先計畫再執行 | 分離規劃與執行 |
| [16 map-reduce](examples/16-map-reduce.md) | 切片→平行→合併 | 大輸入、子任務獨立 |
| [17 pipeline](examples/17-pipeline.md) | item 各自流過各 stage | 階段間不設 barrier，省等待 |
| [18 divide-and-conquer](examples/18-divide-and-conquer.md) | 遞迴拆解再合併 | 超長文摘要的摘要 |

## 五、Search / 動態策略（控制流由結果決定）

| 範例 | 策略 | 一句話 |
|---|---|---|
| [19 ReAct](examples/19-react.md) | reason→act→observe | 邊看邊做的互動任務 |
| [20 escalation ladder](examples/20-escalation-ladder.md) | 便宜失敗才升級 | 成本敏感、難度不均 |
| [21 tree search / LATS](examples/21-tree-search.md) | 展開→評分→回溯 | 早期選擇會鎖死後續 |
| [22 hill-climbing](examples/22-hill-climbing.md) | 只接受更好的改動 | 有客觀評分、要只進不退 |

## 進階 / 前沿（2024–2026，可用 prompt 表達的）

| 範例 | 掛回哪一族 |
|---|---|
| [23 evaluator-optimizer](examples/frontier/23-evaluator-optimizer.md) | 一之補（雙 agent 評改） |
| [24 chain-of-verification](examples/frontier/24-chain-of-verification.md) | 三之補（定位第一個錯只修那步） |
| [25 mixture-of-agents](examples/frontier/25-mixture-of-agents.md) | 二之補（分層聚合，綜合>挑一個） |
| [26 skeleton-of-thought](examples/frontier/26-skeleton-of-thought.md) | 四之補（骨架→並行展開，降延遲） |
| [27 agentic RAG](examples/frontier/27-agentic-rag.md) | 五之補（檢索進迴圈，Self-RAG/CRAG） |
| [28 ReWOO](examples/frontier/28-rewoo.md) | 四之補(2)（先規劃整條再一次執行） |
| [29 least-to-most](examples/frontier/29-least-to-most.md) | 四之補(2)（由易到難的依賴鏈） |
| [30 plan-and-solve](examples/frontier/30-plan-and-solve.md) | 四之補(2)（先計畫治跳步） |
| [31 self-discover](examples/frontier/31-self-discover.md) | 四之補(2)（自組推理結構） |
| [32 AlphaCode 式](examples/frontier/32-alphacode-style.md) | 二之補(2)（狂生+oracle過濾+分群） |
| [33 rubric-based judge](examples/frontier/33-rubric-based-judge.md) | 三之補(2)（逐項量表評分） |
| [34 guardrail feedback loop](examples/frontier/34-guardrail-feedback-loop.md) | 三之補（pre/post 雙層護欄） |
| [35 subagent context 隔離](examples/frontier/35-subagent-context-isolation.md) | 四之補 / 新族C（context engineering） |
| [36 cascade routing](examples/frontier/36-cascade-routing.md) | 五之補（成本量化的升級階梯） |
| [37 universal self-consistency](examples/frontier/37-universal-self-consistency.md) | 二之補（開放式答案挑最一致） |
| [38 weighted voting](examples/frontier/38-weighted-voting.md) | 二之補（信心加權多數決） |
| [39 graph-of-thoughts](examples/frontier/39-graph-of-thoughts.md) | 四之補（思路成圖，可合併綜效節點） |

## 六、Meta / 系統層（算力配置與跨任務——在 agent runtime 可落地的）

原本歸在「非 prompt 可表達」的概念中，有四個在現代 agent runtime（如 Claude Code：`effort` 參數、skills 目錄、eval 迴圈、subagent 隔離）其實**可以在 orchestration 層落地**，各補一個可操作範例：

| 範例 | 掛回哪一族 | 一句話 |
|---|---|---|
| [40 reasoning-effort 旋鈕](examples/frontier/40-reasoning-effort.md) | 新族 A | 先調內建迴圈深度，別急著手疊；外層仍包 oracle |
| [41 skill library](examples/frontier/41-skill-library.md) | 新族 E | oracle 認證過的產物沉澱成 recipe，跨任務複利 |
| [42 prompt optimization loop](examples/frontier/42-prompt-optimization-loop.md) | 新族 B | 迭代 prompt 而非答案；凍結的 eval set 當 oracle |
| [43 context curation](examples/frontier/43-context-curation.md) | 新族 C+F | 每輪蒸餾狀態防 context rot；長迴圈的地基 |

## 七、四個盲區的補完（2026-07 導入，44–53）

前 43 招把「怎麼生、怎麼挑、怎麼驗、怎麼拆、怎麼搜」蓋得很全；[缺口研究](import-candidates.md)找出四個系統性盲區，各補上可操作範例：

**盲區 1：生成端的確定性與多樣性**（既有招的外部訊號全在驗證端）

| 範例 | 掛回哪一族 | 一句話 |
|---|---|---|
| [44 PAL / Program-of-Thoughts](examples/frontier/44-pal-program-of-thoughts.md) | 五之補（oracle 事前形態） | 會算錯的子步驟外包給直譯器，別心算 |
| [45 verbalized sampling](examples/frontier/45-verbalized-sampling.md) | 二之補（生成端） | 要「N 個答案＋機率」破 mode collapse，候選才有多樣性可挑 |

**盲區 2：「不答 / 先問」的輸出策略**（既有招全假設要交出一個答案、輸入是明確的）

| 範例 | 掛回哪一族 | 一句話 |
|---|---|---|
| [46 一致性棄答閘](examples/frontier/46-consistency-abstention.md) | 三之補 | sample 發散＝警報：棄答 / 升級 / 標「未驗證」 |
| [49 clarify-before-act](examples/frontier/49-clarify-before-act.md) | 五之補（互動） | 行為不一致＝需求有歧義，先問再做 |

**盲區 3：副作用世界的安全控制**（既有招全活在無副作用的文字空間）

| 範例 | 掛回哪一族 | 一句話 |
|---|---|---|
| [50 dry-run + pre-mortem](examples/frontier/50-dry-run-premortem.md) | 新族 D 可 prompt 版 | 不可逆動作前先模擬＋「假設已失敗，死因？」 |
| [51 sandbox-verify-commit](examples/frontier/51-sandbox-verify-commit.md) | 五之補 | 交易式執行：隔離跑 → oracle 驗 → 過了才 commit |
| [52 dual-LLM / CaMeL](examples/frontier/52-dual-llm-quarantine.md) | 三之補（guardrail 縱深） | 特權/隔離分離，讓 prompt injection 在架構上無效 |

**盲區 4：驗證器自身的品質**（meta 洞察 2 吃到自己身上）

| 範例 | 掛回哪一族 | 一句話 |
|---|---|---|
| [47 關係式驗證](examples/frontier/47-relation-based-verification.md) | 三之補 | round-trip / metamorphic：沒 ground truth 也有客觀訊號 |
| [48 quote-grounded](examples/frontier/48-quote-grounded.md) | 三之補（27 姊妹） | 先逐字摘引再作答，幻覺在結構上寫不出來 |
| [53 judge 去偏協定](examples/frontier/53-judge-debias.md) | 二/三之補（衛生規範） | 換位 ×2、盲評、judge≠generator——先修裁判再信裁決 |

同積木三用途的家族線：[06](examples/06-self-consistency.md) 挑答案 / [46](examples/frontier/46-consistency-abstention.md) 拉警報 / [49](examples/frontier/49-clarify-before-act.md) 測歧義。
不可逆動作的閘門鏈：[49](examples/frontier/49-clarify-before-act.md)（題目對嗎）→ [50](examples/frontier/50-dry-run-premortem.md)（後果可接受嗎）→ [51](examples/frontier/51-sandbox-verify-commit.md)（真執行也能回滾）。

---

## 怎麼選（決策樹）

- 有客觀 oracle（測試 / 編譯）→ [until oracle passes](examples/05-until-oracle-passes.md)（能逐步就用 PRM）。**最可靠，也是 self-refine 唯一安全的前提。**
- 想 self-refine 但沒 oracle → 先想辦法弄外部訊號（工具 / 檢索 / 測試），否則別做——純內省會[越改越差](examples/10-self-refine.md)。
- 解空間窄、有唯一解 → [self-consistency](examples/06-self-consistency.md)（+ [信心加權](examples/frontier/38-weighted-voting.md)）；**別急著上 [debate](examples/11-debate.md)**。
- 解空間寬 → [best-of-N](examples/07-best-of-n.md) / [MoA](examples/frontier/25-mixture-of-agents.md)（多樣策略 > 狂 sample 同一個）；[tournament](examples/08-tournament.md)。
- 怕「合理但錯」 → [adversarial verify](examples/12-adversarial-verify.md)。
- 怕「漏」 → [loop-until-dry](examples/04-loop-until-dry.md) + [completeness critic](examples/14-completeness-critic.md)。
- 要探索且可回溯的 agent 任務 → [tree search / LATS](examples/21-tree-search.md)。
- 成本敏感 → [cascade routing](examples/frontier/36-cascade-routing.md)。
- 需要外部知識 → [agentic RAG](examples/frontier/27-agentic-rag.md)。
- 路徑可事先寫死 → 用 **workflow**（[plan-then-execute](examples/15-plan-then-execute.md) / [map-reduce](examples/16-map-reduce.md) / [pipeline](examples/17-pipeline.md)），別上 agent。
- 可事先規劃、不需邊做邊看 → [ReWOO](examples/frontier/28-rewoo.md) / [plan-and-solve](examples/frontier/30-plan-and-solve.md)，別用 [ReAct](examples/19-react.md) 逐步往返。
- 任務太大 → 先 [decompose](examples/15-plan-then-execute.md) 再對每塊套上面任一種。
- 想手疊 self-refine / self-consistency，而模型有思考預算旋鈕 → 先試 [reasoning-effort](examples/frontier/40-reasoning-effort.md) 單發調高當基線。
- 迴圈會超過 ~5 輪 → 必配 [context curation](examples/frontier/43-context-curation.md)（每輪蒸餾），否則 context rot 吃掉一切增益。
- 同類任務會重複出現 → [skill library](examples/frontier/41-skill-library.md)，oracle 認證過才入庫。
- prompt 本身是長期資產 → [prompt optimization loop](examples/frontier/42-prompt-optimization-loop.md)。
- 子步驟含算術 / 可程式化邏輯 → [44 PAL](examples/frontier/44-pal-program-of-thoughts.md)：外包給直譯器，別心算。
- generate-then-select 但候選都長一樣 → [45 verbalized sampling](examples/frontier/45-verbalized-sampling.md) 先把多樣性生出來。
- 沒 oracle、怕幻覺、寧可不答 → [46 一致性棄答閘](examples/frontier/46-consistency-abstention.md)；任務有可逆/等變結構 → [47 關係式驗證](examples/frontier/47-relation-based-verification.md)。
- RAG 答案要防摻私貨 → [48 先摘引再作答](examples/frontier/48-quote-grounded.md)。
- 需求可能有歧義 → [49 clarify-before-act](examples/frontier/49-clarify-before-act.md)（行為不一致＝該問了）。
- 動作不可逆 → [50 先模擬 + pre-mortem](examples/frontier/50-dry-run-premortem.md)，再進 [51 sandbox-verify-commit](examples/frontier/51-sandbox-verify-commit.md)。
- agent 讀不可信內容且有工具權 → [52 dual-LLM 隔離](examples/frontier/52-dual-llm-quarantine.md)。
- 用了任何 judge（07/08/09/33）→ 一律套 [53 去偏協定](examples/frontier/53-judge-debias.md)。

## 三個要記住的 meta 洞察

1. 這些招數都是 **test-time compute 的不同花法**（序列 / 並行 / 內化進權重 / 離線）。
2. **最該記的一條**：沒有外部回饋的自我修正會**退化**——迴圈的價值取決於回饋訊號品質，不是迴圈本身。
3. 三個「預設選較樸素那邊」的準則：**多樣性 > 規模**、**workflow > agent**（能用就用）、**逐步獎勵（PRM）> 結果獎勵（ORM）**。

---

## 不在本範例集內：系統/訓練層概念（非單一 prompt 可表達）

來源文件也涵蓋了一些**不是「寫一個 prompt」就能落地**的概念——它們是訓練方法、系統架構或研究方向，硬編成「prompt 範例」會誤導。這裡誠實列出並指回來源，而不是為湊數而虛構範例：

- **Constitutional AI / RLAIF**：把 self-refine 搬到**訓練期**，回饋來自明文憲法。（[三之補(2)](agentic-loop.md)）— 其**工程即時版**才是可 prompt 的 [guardrail feedback loop](examples/frontier/34-guardrail-feedback-loop.md)。
- **World models / co-evolving**：agent 自帶能想像後果的模擬器，訓練期形態尚在研究期。（[新族 D](agentic-loop.md)）— 其**工程即時版**已可落地：[50 dry-run + pre-mortem](examples/frontier/50-dry-run-premortem.md)（讓模型在不可逆動作前當一次世界模型）。
- **PRM / process reward、generative verifier**：屬驗證器/獎勵模型的訓練與設計；其精神已體現在 [until oracle passes](examples/05-until-oracle-passes.md)（逐步驗證）與 [rubric judge](examples/frontier/33-rubric-based-judge.md)。

四個原本列在這裡的概念（reasoning models 的 effort 旋鈕、DSPy/OPRO 式自動優化、Voyager skill library、sleep-time/context 策展）已各自有**orchestration 層的可操作版**，移入上方[第六族](#六meta--系統層算力配置與跨任務在-agent-runtime-可落地的)——訓練期的原始形態仍以 [agentic-loop.md](agentic-loop.md) 為準。

## 來源

所有範例的理論依據與延伸閱讀連結，見 [agentic-loop.md](agentic-loop.md)。
