# 方法論導入候選研究（2026-07-09）

> **狀態：已執行（2026-07-09）。** Tier 1 十招已落地為 [examples/frontier/](examples/frontier/) 44–53；Tier 2 七項已併入對應範例（40/43/41/30/36/16）；[index.md 第七族](index.md#七四個盲區的補完2026-07-導入4453)、[agentic-loop.md 補充三](agentic-loop.md)、composer 的 [catalog](skills/agentic-prompt-composer/references/catalog.md) 與 [SKILL.md](skills/agentic-prompt-composer/SKILL.md) 均已同步。本文件保留為缺口分析的依據與 Tier 3 的「勿重複評估」清單。

研究「除了現有 43 招之外，還有哪些方法論可以導入本範例集」。方法：先盤點 [index.md](index.md) 的 43 招涵蓋範圍，找出缺口；候選逐一與現有招比對排除重疊；2024–2026 的新方法用 web 搜尋逐一核實來源（下列 arXiv 編號皆已驗證，非憑記憶）。

**入選標準**（沿用本範例集既有原則）：
1. 可在 prompt / orchestration 層落地（訓練期方法照舊排除）；
2. 與現有 43 招不重疊，或重疊處有明確的新增維度；
3. 有可信來源。

---

## Tier 1：建議新增為獨立範例（10 個，建議編號 44–53）

### 44. PAL / Program-of-Thoughts（計算外包給直譯器）
- **是什麼**：推理歸 LLM，計算歸程式——讓模型把「會算錯的部分」寫成 code 丟給直譯器執行，而不是在腦內心算。CRITIC 是它的批判版：用工具（搜尋、直譯器、API）來 ground 每一輪批判。
- **掛哪族**：五之補（oracle 家族的「事前」形態）。現有 [05 until oracle passes](examples/05-until-oracle-passes.md) 是「做完後用 oracle 驗」；PAL 是「一開始就把不可靠的子步驟外包給確定性引擎」，缺口明確。
- **為什麼值得**：本範例集反覆強調「外部訊號 > 自我感覺」，但目前所有招的外部訊號都在「驗證端」；PAL 把外部確定性拉到「生成端」，是同一主線的另一半。
- **來源**：[PAL（arXiv:2211.10435）](https://arxiv.org/abs/2211.10435)、[Program-of-Thoughts（arXiv:2211.12588）](https://arxiv.org/abs/2211.12588)、[CRITIC（arXiv:2305.11738）](https://arxiv.org/abs/2305.11738)——CRITIC 在 [agentic-loop.md](agentic-loop.md) 一之補已被點名（「即 CRITIC 精神」）但從未有範例。

### 45. Verbalized Sampling（口頭化抽樣，恢復多樣性）
- **是什麼**：對齊後的模型有 mode collapse——同一題狂 sample 也趨同。VS 是 training-free 解法：不叫模型「給一個答案」，改叫它「給 5 個答案＋各自的機率」，直接口頭化一個分佈。創意任務多樣性提升 1.6–2.1×。
- **掛哪族**：二之補。整個 generate-then-select 族（06/07/08/32）的前提是「候選夠多樣」，而 [agentic-loop.md](agentic-loop.md) 的關鍵洞察正是「多樣性 > 規模」——但現有招沒有任何一個處理「怎麼把多樣性生出來」。VS 補上這個生成端缺口。
- **來源**：[Verbalized Sampling（arXiv:2510.01171）](https://arxiv.org/abs/2510.01171)。

### 46. 一致性不確定度 + 棄答閘（SelfCheckGPT / semantic entropy）
- **是什麼**：沒有 oracle 也能估「這答案多可疑」：同一題獨立 sample N 次，把答案**按語意分群**，群越散＝不確定度越高 → 高於門檻就**棄答 / 升級 / 掛「未驗證」標籤**，而不是硬答。Semantic entropy 登上 Nature 2024；SelfCheckGPT 是逐句版。
- **掛哪族**：三之補。與 [06 self-consistency](examples/06-self-consistency.md) 用同一個機制（sample + 比對），但用途相反：06 拿一致性**挑答案**，46 拿**不一致性當警報**。「答 vs 不答」是現有 43 招完全沒有的輸出策略——所有招都假設最後要交出一個答案。
- **來源**：[Detecting hallucinations using semantic entropy（Nature 2024）](https://www.nature.com/articles/s41586-024-07421-0)、[SelfCheckGPT（arXiv:2303.08896）](https://arxiv.org/abs/2303.08896)、[Semantic Entropy Probes（arXiv:2406.15927）](https://arxiv.org/abs/2406.15927)。

### 47. 關係式驗證（round-trip + metamorphic relations）
- **是什麼**：沒有 ground truth 時，用「輸入輸出之間必須成立的關係」當 pseudo-oracle：
  - **Round-trip**：正向做完再反向重建，比對是否等價（code→自然語言描述→重新生成 code→行為比對；翻譯回譯；摘要→擴寫→事實比對）。
  - **Metamorphic**：擾動輸入（改寫、換序、加無關句），檢查輸出是否按預期關係變化（該不變的不變）。
- **掛哪族**：三之補。現有驗證招（12/24/33）全靠「另一個 LLM 的判斷」；關係式驗證的訊號來自**任務結構本身**，客觀性介於 LLM-judge 與硬 oracle 之間——正好填「沒測試可跑、又不想全信 judge」的空隙。
- **來源**：[Round-Trip Correctness（ICML 2024，arXiv:2402.08699）](https://arxiv.org/abs/2402.08699)、[Metamorphic Testing of LLMs for NLP（arXiv:2511.02108）](https://arxiv.org/abs/2511.02108)、[Metamorphic Prompt Testing（arXiv:2406.06864）](https://arxiv.org/html/2406.06864v1)。

### 48. Quote-grounded generation（先摘引、再作答）
- **是什麼**：RAG / 長文任務的反幻覺骨架：第一步只准從來源**逐字抽引文**（含出處定位），第二步只准根據抽出的引文作答，答案中每個 claim 都要指回引文編號。抽不到引文的 claim 就不准寫。
- **掛哪族**：三之補，[27 agentic RAG](examples/frontier/27-agentic-rag.md) 的姊妹。27 管「何時查、查什麼」；48 管「查回來之後怎麼防止模型摻私貨」——檢索迴圈的最後一哩。
- **為什麼值得**：把「幻覺驗證」從事後（CoVe 式）提前到結構上不可能發生（沒有引文就沒有 claim），是 guardrail 精神的生成端版本。Anthropic 官方文件的招。
- **來源**：[Anthropic — Reduce hallucinations（官方 docs）](https://docs.anthropic.com/en/docs/test-and-evaluate/strengthen-guardrails/reduce-hallucinations)、[Anthropic — Citations API](https://docs.anthropic.com/en/docs/build-with-claude/citations)。

### 49. Clarify-before-act（動手前的消歧閘）
- **是什麼**：接到需求先偵測模糊度，模糊就**先問再做**，不模糊直接做。ClarifyGPT 的偵測器很漂亮：對需求生 n 份實作跑同一組測資，**行為不一致 = 需求有歧義** → 針對分歧點生成澄清問題。GPT-4 在 MBPP 上 Pass@1 70.96%→80.80%。
- **掛哪族**：五（互動類）。現有 43 招全部假設「輸入是明確的」，沒有任何一招處理「輸入本身就是錯誤來源」——而實務上 agent 做錯最常見的原因就是需求歧義。
- **加分**：偵測機制直接復用 [06 self-consistency](examples/06-self-consistency.md) 的積木（sample 多份看散度），與 46 是同一機制的第三種用途（挑答案 / 拉警報 / 測歧義）。
- **來源**：[ClarifyGPT（arXiv:2310.10996，FSE 2024）](https://arxiv.org/abs/2310.10996)。

### 50. Dry-run / simulate-before-act（不可逆動作前先模擬 + pre-mortem）
- **是什麼**：對不可逆 / 高成本動作（刪資料、發 email、部署、大額 API 呼叫），執行前先叫模型**當世界模型**：模擬「執行後會發生什麼」＋pre-mortem（「假設三個月後這件事失敗了，最可能的死因是什麼」），模擬結果不過關就改計畫。RAP 證明 LLM-as-world-model + 規劃搜尋在 prompt 層可行。
- **掛哪族**：新族 D 的**可 prompt 版**。[index.md](index.md) 目前把 world models 整族歸在「不在本範例集內」；50 是它的工程即時版——如同 Constitutional AI（訓練期）之於 [34 guardrail loop](examples/frontier/34-guardrail-feedback-loop.md)（工程版），同一個「訓練期概念可萃取出 prompt 層版本」的先例。
- **來源**：[RAP: Reasoning via Planning（arXiv:2305.14992）](https://arxiv.org/abs/2305.14992)；pre-mortem 為決策科學經典（Gary Klein, HBR 2007）。

### 51. Sandbox-verify-commit（交易式執行：檢查點與回滾）
- **是什麼**：把 agent 的檔案/系統操作包進交易語意：**隔離環境執行（git worktree / sandbox）→ oracle 驗證 → 通過才 commit / merge，失敗整包回滾**。多 agent 並行改檔時各給一個 worktree，合併前各自過驗證。
- **掛哪族**：五。它是 [05 oracle](examples/05-until-oracle-passes.md) 與 [21 backtracking](examples/21-tree-search.md) 的**副作用版**：21 的回溯只能撤銷「想法」，51 讓「已執行的動作」也可撤銷——現有招都假設動作發生在無副作用的文字空間。2025–2026 生產級 agent 的標配模式（Claude Code worktree isolation、Anthropic 的 agent 安全實務都內建這件事）。
- **來源**：[Anthropic — Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)（sandboxing / guardrails 節）；2026 生產模式綜述如 [Agentic Design Patterns 2026 catalog](https://www.augmentcode.com/guides/agentic-design-patterns)（bounded execution / 分層安全控制）。

### 52. Dual-LLM / CaMeL（不可信輸入的特權隔離）
- **是什麼**：agent 要讀不可信內容（網頁、email、外部文件）時的防 prompt-injection 架構：**特權 LLM** 只看使用者的可信指令、負責規劃與呼叫工具、永不接觸不可信資料；**隔離 LLM** 只讀不可信資料、吐結構化欄位、沒有工具權限。CaMeL 再加 capability 標籤追蹤每個值的來源與流向，AgentDojo 上 67% 任務達到可證明安全。
- **掛哪族**：三之補，[34 guardrail loop](examples/frontier/34-guardrail-feedback-loop.md) 的縱深版。34 是「檢查內容」（regex / 分類器，可被繞過）；52 是「架構上讓注入無效」（控制流與資料流分離）——質的差異，不是程度差異。本範例集目前完全沒有安全向的控制結構。
- **來源**：[CaMeL: Defeating Prompt Injections by Design（arXiv:2503.18813，DeepMind 2025）](https://arxiv.org/abs/2503.18813)、[Simon Willison — Dual LLM pattern（2023）](https://simonwillison.net/2023/Apr/25/dual-llm-pattern/)、[Willison 評 CaMeL（2025）](https://simonwillison.net/2025/Apr/11/camel/)。

### 53. Judge 去偏協定（position swap、長度偏誤、self-preference）
- **是什麼**：LLM-as-judge 有系統性偏誤：位置偏誤（偏好先出現的候選）、長度偏誤（偏好長答案）、self-preference（偏好自己生成的答案）。去偏協定：pairwise 比較必做**兩次換位**（不一致判平手）、judge 與 generator 用不同模型、盲評（剝除來源標記）、rubric 錨定分數。
- **掛哪族**：二/三之補——不是新招，是 [07](examples/07-best-of-n.md)/[08](examples/08-tournament.md)/[09](examples/09-judge-panel.md)/[33](examples/frontier/33-rubric-based-judge.md) 全體的**衛生規範**。本範例集大量依賴 judge（至少 6 招），但沒有任何一頁講「judge 本身會壞掉、怎麼校正」——這是 meta 層的洞察 2（回饋訊號品質決定一切）在 judge 上的直接應用。
- **來源**：[Judging LLM-as-a-Judge / MT-Bench（arXiv:2306.05685）](https://arxiv.org/abs/2306.05685)。

---

## Tier 2：不開新檔，併入現有範例的補強（7 項）

| 補強 | 併入哪裡 | 內容 | 來源 |
|---|---|---|---|
| Chain-of-Draft | [40 reasoning-effort](examples/frontier/40-reasoning-effort.md) | effort 之外的第二顆旋鈕：限制每步推理 ≤5 詞，token 降到 CoT 的 7.6% 而準度相當。40 調「想多久」，CoD 調「寫多短」 | [arXiv:2502.18600](https://arxiv.org/abs/2502.18600) |
| ACE（agentic context engineering） | [43 context curation](examples/frontier/43-context-curation.md) + [42](examples/frontier/42-prompt-optimization-loop.md) | 43 的升級：整份重寫 context 會 context collapse（越縮越空洞），ACE 改用**增量 delta 更新**（grow-and-refine），playbook 只長不塌。AppWorld 上 +10.6% | [arXiv:2510.04618](https://arxiv.org/abs/2510.04618) |
| Buffer of Thoughts | [41 skill library](examples/frontier/41-skill-library.md) | 41 沉澱的是「任務級 recipe」；BoT 沉澱「推理模板」（thought-template），檢索後實例化。同一精神、不同粒度，一段對照即可 | [arXiv:2406.04271](https://arxiv.org/abs/2406.04271) |
| Step-back prompting | [30 plan-and-solve](examples/frontier/30-plan-and-solve.md) 或 [31](examples/frontier/31-self-discover.md) | 解題前先問「這題背後的一般性原理是什麼」，先抽象再具體。規劃譜系上加一個點 | [arXiv:2310.06117](https://arxiv.org/abs/2310.06117) |
| Speculative draft-verify | [36 cascade routing](examples/frontier/36-cascade-routing.md) | 36 是「整題路由」；變體是「便宜模型出草稿、貴模型只做審與修」——同預算下另一種分工拓撲 | 工程模式（speculative decoding 的 orchestration 類比） |
| 失敗庫（anti-pattern library） | [41 skill library](examples/frontier/41-skill-library.md) | 41 的鏡像:除了沉澱「驗證通過的作法」，也沉澱「驗證失敗的死法」餵給後續任務當反面 few-shot。ACE 的 reflection 內建此事 | 同 ACE |
| Blackboard 拓撲 | [16 map-reduce](examples/16-map-reduce.md) 或 [35](examples/frontier/35-subagent-context-isolation.md) | orchestrator 之外的第三種多 agent 拓撲：共享黑板，誰有進展誰寫。2026 各 taxonomy 均列為 canonical pattern | [2026 orchestration patterns](https://beam.ai/agentic-insights/multi-agent-orchestration-patterns-production) |

## Tier 3:評估後**不建議**導入（附理由，防止之後重複評估）

- **Self-Ask**（arXiv:2210.03350）：「自問子問題+檢索」的精神已被 [29 least-to-most](examples/frontier/29-least-to-most.md) + [27 agentic RAG](examples/frontier/27-agentic-rag.md) 完整覆蓋。
- **Chain-of-Density**（arXiv:2309.04269）：只是 [03 until threshold](examples/03-until-threshold.md) 在摘要任務上的特例（固定長度、逐輪加實體密度）。
- **Reflexion**：已內含於 [10 self-refine](examples/10-self-refine.md)（該頁明文提及）。
- **Meta-prompting**（arXiv:2401.12954）：conductor 生 expert persona ＝ orchestrator-workers + [35 context 隔離](examples/frontier/35-subagent-context-isolation.md) 的單模型版，無新增控制結構。
- **Analogical prompting**（arXiv:2310.01714）：自生 few-shot 例子，是單發 prompt 技巧而非控制結構，超出本集範圍。
- **Tree-of-Thoughts**：[21 tree search](examples/21-tree-search.md) 已含（LATS 即其 agent 化）。
- **Swarm / handoff routing**：路由精神已在 [36](examples/frontier/36-cascade-routing.md)；handoff 只是實作介面差異。
- **STaR / RLVR / test-time training 等訓練期方法**：照既有排除原則，留在 [agentic-loop.md](agentic-loop.md) 理論層。

---

## 決策樹追加建議（若 Tier 1 全數導入）

- 子步驟含算術 / 可程式化邏輯 → **44 PAL**：外包給直譯器，別心算。
- generate-then-select 但候選都長一樣 → **45 verbalized sampling** 先把多樣性生出來。
- 沒 oracle、怕幻覺、寧可不答 → **46 一致性棄答閘**；有任務結構可用 → **47 關係式驗證**。
- RAG 答案要防摻私貨 → **48 先摘引再作答**。
- 需求可能有歧義 → **49 clarify-before-act**（行為不一致 = 該問了）。
- 動作不可逆 → **50 先模擬 + pre-mortem**，再進 **51 sandbox-verify-commit**。
- agent 會讀不可信內容 → **52 dual-LLM 隔離**。
- 用了任何 judge（07/08/09/33）→ 一律套 **53 去偏協定**。

## 本次研究的一句話總結

現有 43 招把「怎麼生、怎麼挑、怎麼驗、怎麼拆、怎麼搜」蓋得很全，缺口集中在四個系統性盲區：**生成端的確定性外包與多樣性**（44/45）、**「不答 / 先問」這類非交付型輸出策略**（46/49）、**副作用世界的安全控制**（50/51/52），以及**驗證器自身的品質**（47/53）——最後這條正是本集 meta 洞察 2（回饋訊號品質決定成敗）尚未吃到自己身上的部分。
