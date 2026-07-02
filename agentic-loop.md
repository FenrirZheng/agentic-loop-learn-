AI 控制結構 Prompt 模式(除了 iterate until convergence)
「iterate until convergence」本質是一個帶停止條件的迴圈。以下把同類模式依 「控制結構的骨架」分族 — 它們其實是同一批積木的不同組合。

一、迴圈類:差別只在「停止條件」
iterate until convergence 是這族的一員,把停止條件換掉就是新模式:
模式
停止條件
適用
until convergence
這一輪輸出 ≈ 上一輪(diff 為空)
refine 文字、改 code 到穩定
until budget
跑到 N 輪 / token 用完
探索型任務,不想無限跑
until threshold
自評分數 ≥ X
有可量化品質指標(可讀性、覆蓋率)
loop-until-dry / saturation
連續 K 輪「找不到新東西」
找 bug、列 edge case、審計 — 簡單的 count < N 會漏掉長尾
until oracle passes
外部驗證通過(測試綠燈、編譯過、lint 乾淨)
最強的一種,停止條件客觀,不靠模型自我感覺

關鍵洞察:until convergence 的弱點是「收斂 ≠ 正確」,模型可能穩定地停在一個 錯誤答案上。只要能換成 until oracle passes(有外部 ground truth),幾乎都該換。

二、Generate-then-Select:不迭代,改成「一次生很多再挑」
迴圈是「序列改進」;這族是「並行發散 + 收斂挑選」,通常品質更穩、更不會卡在局部最優:
Self-consistency(取多數):同一題獨立跑 N 次,投票取最常見答案。適合有唯一解的推理題。
Best-of-N / rejection sampling:生 N 個候選,用一個 judge prompt 挑最好的。
Tournament / bracket:候選兩兩 PK,贏家晉級。當「絕對評分」很難但「相對比較」容易時用。
Judge panel:多個不同角度的評審(正確性 / 安全 / 可讀性)各打分再綜合,比單一評審穩。

三、Adversarial / Critique:引入「對抗」防止模型自我感覺良好
Self-refine / Reflexion:產出 → 自我批判 → 依批判修正,固定幾輪。是 convergence loop 的「強制每輪都先批判」變體。
Debate:兩個 agent 各持立場辯論,第三個當裁判。
Adversarial verify(最常推薦):每個 finding 派 N 個專門被指示去「推翻」它的 skeptic,多數推翻就丟掉。對抗式驗證能擋掉「聽起來合理但其實錯」的結論。
Devil's advocate / red-team:對自己剛給的建議做一次專門挑毛病的 pass。
Completeness critic:最後加一個 agent 專問「還缺什麼?哪個面向沒查、哪個說法沒驗證」,它找到的就是下一輪的工作。

四、Decomposition:把「大」拆成「可控的小」
Plan-then-execute:先出計畫,再逐項執行(分離規劃與執行)。
Map-reduce / fan-out-fan-in:切片 → 平行處理 → 合併。
Pipeline:多階段流水線,每個 item 獨立流過所有 stage(不在 stage 間設 barrier)。
Divide-and-conquer / 遞迴:問題太大就遞迴拆解再合併(例:summarize 超長文 → 分段 summary → summary of summaries)。

五、Search / 動態策略:控制流由「結果」決定
ReAct:reason → act → observe 迴圈,每步根據觀察決定下一步(agentic 的基礎骨架)。
Escalation ladder:先用便宜方法,失敗才升級到貴的(便宜模型 → 貴模型;簡單 prompt → 完整 prompt)。
Tree search / backtracking:展開多個分支,評估,走不通就回溯(LLM 版的 MCTS)。
Hill-climbing:每步只接受「比現在好」的修改,否則丟棄。

怎麼選(決策樹)
有客觀 oracle(測試/編譯) → 迴圈用 until oracle passes,最可靠。
沒 oracle 但解空間窄、有唯一解 → self-consistency 取多數。
解空間寬(設計、寫作) → Best-of-N / tournament,別硬迭代。
怕「合理但錯」 → 加 adversarial verify。
怕漏 → loop-until-dry + completeness critic。
任務太大 → 先 decompose 再對每塊套上面任一種。
這些是積木,實務上會疊著用 — 例如 fan-out 找 bug → loop-until-dry → 每個 bug 用 3 票對抗驗證 → judge panel 綜合。

---

# 補充:世界前沿的控制結構(2024–2026)

先給一個把你原本五族收攏起來的統合框架,再把新模式掛回原本的族,最後補四個原文完全沒有的新族。

## 統合框架:Test-Time Scaling(推論期算力擴展)

你原本的「迴圈 vs generate-then-select」其實是同一件事的兩大軸——**在推論期多花算力換品質**:

- **Sequential scaling(序列)**:同一條線上反覆修 → self-refine / reflexion / verifier-guided revision。就是你的「一、迴圈類」。
- **Parallel scaling(並行)**:一次生很多再挑 → best-of-N / self-consistency / MoA。就是你的「二、generate-then-select」。
- **第三軸(新)**:把整個迴圈「內化」進模型權重 → reasoning models,見下面〈新族 A〉。

關鍵洞察(新):**算力不是灑越多越好**。研究一致發現「花在哪、怎麼花」比「花多少」重要;而且**多樣性 > 純粹重複取樣**——用不同策略/角度的異質 agents,比對同一個最強模型狂 sample,準度更高、成本更低。

## 一之補:迴圈類的新成員與新警訊

- **Evaluator-Optimizer**(Anthropic 命名):一個 LLM 生,另一個 LLM 評+給回饋,循環。就是你 until threshold 的「雙 agent」版,重點在**評審與產出分離**。
- **Process Reward Model(PRM)/ verifier-guided revision**:不是等整份做完才評,而是**逐步**打分。對比:**ORM**(outcome reward,只看最終對不對)vs **PRM**(process reward,每一步都給訊號)。PRM + 樹搜尋(如 ReST-MCTS*)是目前數學/推理最強的一類——「逐步獎勵 > 結果獎勵」。
- **Generative verifier**:讓「裁判」自己先寫一段 CoT 推理再判對錯,比只吐一個分數的 discriminative verifier 準。呼應你原本 judge panel,但裁判會思考。
- ⚠️ **重大警訊(強化你「收斂 ≠ 正確」)**:**沒有外部訊號時,LLM 的 intrinsic self-correction 通常會越改越差**。純自我反省會出現 "degeneration-of-thought"(反覆反省反而收斂到更糟);研究指出「回饋品質」才是瓶頸,不是迴圈次數。→ 實務結論:self-refine 迴圈一定要接**外部 oracle / 工具 / 檢索**(即 CRITIC 精神),否則不如不改。這正是你 until oracle passes 最該用、最該優先的理由。

## 二之補:generate-then-select 的新招

- **Weighted / confidence-weighted voting**:majority 的加權版,用各 agent 自報信心加權。研究顯示 self-consistency 的「多數決」本身就吃掉了大部分增益。
- **Universal self-consistency**:開放式回答沒有唯一解、無法精確比對時,用一個 LLM 讀完所有候選挑「最一致」的那個。
- **Mixture-of-Agents(MoA)**:分層架構——每層多個 agent 讀上一層**全部**輸出再重寫,最後 aggregator 綜合。研究:aggregator 綜合 > 單純 ranker 挑一個。
- ⚠️ **Multi-agent debate 的現實**:很多 MAD 框架**贏不過單純的 self-consistency**;增益大多來自「多次取樣 + 多數決」而非「辯論」本身。要讓辯論有用,得刻意引入**不對稱**(confidence-weighting、oracle-locking、角色差異)。→ 別為辯論而辯論。

## 三之補:對抗 / 批判族

- **Chain-of-Verification(CoVe / LogiCoT)**:產出後,自動生一串驗證問題逐一查,定位**第一個**出錯的節點、保留前面對的、只重修那一步。比整份重寫又省又準。
- **Guardrail feedback loop(Constitutional 的工程版)**:把守則違規(或確定性檢查失敗)當回饋,餵回去做**定向**修正。分兩層:pre-LLM(regex / PII / prompt-injection 檢查,快且確定性)與 post-LLM(檢查輸出 → 失敗就回修)。這是把你「until oracle passes」落地成即時護欄。

## 四之補:decompose 族

- **Anthropic 五工作流**(把你的 decompose / pipeline 講清楚了):prompt chaining(串接)、routing(分流)、parallelization(切片並行:sectioning + voting)、orchestrator-workers(協調者**動態**拆、發包、綜合——subtask 不預先定義,由 orchestrator 依輸入即時決定)、evaluator-optimizer(評-改迴圈)。
- **Workflow vs Agent 的分界(很重要)**:workflow = 路徑由你寫死(predefined code path);agent = 路徑由模型自己邊做邊決定。**能用 workflow 解就別上 agent**——可預測、可測、便宜。
- **Skeleton-of-Thought**:先產「大綱骨架」,再**並行**展開每個點——主要動機是降延遲。
- **Graph-of-Thoughts**:思路是任意圖,可把多條思路「合併」成一個綜效節點(比 tree 更彈性)。
- **子代理 context 隔離**:把子任務丟給 sub-agent 在**獨立 context** 跑,避免主線 context rot / 汙染(subagent-as-tool)。

## 五之補:search / 動態策略族

- **LATS(Language Agent Tree Search)**:把 ReAct + ToT + 反思 + MCTS 合一——用 value function 對 frontier 節點打分、可 **backtrack**、把失敗經驗寫成 reflection 再探索。是你「tree search / backtracking」的完整 agent 版(HumanEval pass@1 曾達 92.7%)。
- **Model cascade / routing**(你 escalation ladder 的成本量化版):先便宜模型,信心 / 不確定性不足才升級。變體:agreement-based(多個小模型不一致才升級)、conformal-prediction 門檻、有形式成本保證的 cascading。
- **Agentic RAG(把檢索也拉進迴圈)**:
  - **Self-RAG**:模型自己決定「何時該檢索」並自評輸出。
  - **Corrective RAG(CRAG)**:輕量評估器給檢索到的文件打分,不夠好就改寫 query / 換來源 / 上網。
  - **MCTS-RAG**:把樹搜尋套在「檢索 + 推理」的交錯決策上。
  - 精神:檢索不再是一次性前處理,而是 agent 在迴圈裡「要不要查、查什麼、查夠了沒」的決策。

## 新族 A:把迴圈內化進模型——Reasoning models(o1 / o3 / DeepSeek-R1)

- 以前迴圈在 **prompt 層**(你手動 orchestrate);現在用 **RLVR(可驗證獎勵的強化學習)** 把「長 CoT + 自我檢查 + 換策略」直接訓進模型權重。DeepSeek-R1 證明:**純 RL(只給對錯、不用人標推理過程)** 就能長出自我反思 / 驗證 / 回溯等行為。
- 對你的影響:很多「self-consistency / self-refine」的手動 orchestration,對 reasoning model 已內建;你要調的是 **reasoning effort(思考預算)** 這個旋鈕,而不是自己疊迴圈。**但仍需外部 oracle 驗證**——模型內部的自評一樣會錯。

## 新族 B:自我改進 / 自動優化 pipeline

迭代的對象不再是「答案」,而是**產生答案的系統本身**(prompt / 程式):

- **DSPy**:把 LLM 呼叫寫成宣告式模組,用 optimizer(MIPROv2、GEPA)自動搜 few-shot / 指令——不手寫 prompt,讓編譯器調。
- **OPRO**:讓 LLM 看「歷史嘗試 + 分數」自己提更好的指令(把 LLM 當優化器)。
- **TextGrad**:把自然語言回饋當「梯度」在 pipeline 反向傳播——natural-language backprop。
- **自我演化 agent**(Darwin-Gödel Machine、STaR / START):agent 改寫自己的程式 / prompt、用自產資料訓自己。

## 新族 C:記憶與 context engineering(讓長迴圈不崩的地基)

- 長時程 agent 的瓶頸已從「記憶容量」轉為「**主動策展 context**」——把對的證據放在對的時間點。
- 手法:context pruning(剪掉迴圈內無用歷史)、trajectory compression(壓縮走過的路徑)、memory-as-action(把「要記什麼」變成一個 agent 動作)、外部記憶(RAG 化)。
- 為什麼重要:任何 loop-until-dry / 長迴圈,**不做 context 管理就會 context rot,越跑越笨**。

## 新族 D:World models / 自我演化(2026 前沿,尚在研究期)

- Agent 不只回顧過去經驗,還用「世界模型」**預測候選動作的未來結果**再選(類似 model-based RL 的 planning)。
- Co-evolving:文字世界模型與 agent policy 一起訓、互相強化(如 COMAP)。方向是「agent 自帶一個能想像後果的模擬器」。

## 更新版決策樹(把新知識掛進你原本的決策樹)

- 有客觀 oracle → until oracle passes / verifier-guided(**能逐步就用 PRM**)。最可靠,且是 self-refine 唯一安全的前提。
- 想 self-refine 但沒 oracle → **先想辦法弄外部訊號(工具 / 檢索 / 測試)**,否則別做——純內省會越改越差。
- 解空間窄、有唯一解 → self-consistency(+ 信心加權);**別急著上 debate**。
- 解空間寬 → best-of-N / **MoA**(多樣策略 > 狂 sample 同一個);tournament。
- 要探索且可回溯的 agent 任務 → **LATS**(MCTS + value + reflection)。
- 成本敏感 → **cascade routing**(便宜 → 貴,以信心 / 一致性 gate)。
- 任務要外部知識 → **agentic RAG**(檢索進迴圈,Self-RAG / CRAG)。
- 路徑可預先寫死 → 用 **workflow**(五工作流),別上 agent;只有 subtask 無法預測才用 orchestrator-workers。
- 長時程 → 一定要配 **context engineering**(pruning / compression / 子代理隔離)。
- 有能力用 reasoning model → 先讓模型內建的長 CoT 做、只調 reasoning effort,再在外面包 oracle 驗證。

## 三個要記住的 meta 洞察

1. **你原本五族 = test-time compute 的不同花法**;新前沿是把同樣的積木「往內化進模型(reasoning models)」「往上自動優化(DSPy / TextGrad)」「往下打地基(memory / context)」三個方向長出來。
2. **最反直覺、最該記的一條**:沒有外部回饋的自我修正會**退化**——所有迴圈的價值都取決於回饋訊號的品質,不是迴圈本身。
3. **三個「預設選較樸素那邊」的工程準則**:多樣性 > 規模、workflow > agent(能用就用)、逐步獎勵(PRM)> 結果獎勵(ORM)。

---

## 來源(2024–2026)

- [Anthropic — Building Effective Agents(五工作流、workflow vs agent)](https://www.anthropic.com/research/building-effective-agents)
- [Scaling Test-time Compute for LLM Agents(TTS 四面向)](https://arxiv.org/html/2506.12928v1)
- [ReST-MCTS*:Process Reward 引導的樹搜尋(NeurIPS 2024)](https://openreview.net/forum?id=8rcFOqEud5)
- [When Can LLMs Actually Correct Their Own Mistakes?(自我修正的極限,survey)](https://arxiv.org/html/2406.01297v3)
- [LLMs Cannot Self-Correct Reasoning Yet](https://arxiv.org/pdf/2310.01798)
- [Multi-LLM-Agents Debate:效能與擴展的挑戰(ICLR 2025 blogpost)](https://d2jud02ci9yv69.cloudfront.net/2025-04-28-mad-159/blog/mad/)
- [DeepSeek-R1:純 RL 激發推理能力(RLVR)](https://arxiv.org/abs/2501.12948) · [Nature 版](https://www.nature.com/articles/s41586-025-09422-z)
- [Mixture-of-Agents(MoA,分層聚合)](https://arxiv.org/html/2406.04692v1)
- [Demystifying Chains, Trees, and Graphs of Thoughts(CoT/ToT/GoT/SoT 拓撲比較)](https://arxiv.org/pdf/2401.14295)
- [Language Agent Tree Search(LATS)](https://arxiv.org/abs/2310.04406)
- [Reasoning RAG via System 1/2(Agentic RAG survey,含 Self-RAG / CRAG)](https://arxiv.org/html/2506.10408v1)
- [Cluster, Route, Escalate:成本感知的 cascade 服務](https://arxiv.org/html/2606.27457)
- [Context Engineering in 2025(記憶與 context 策展)](https://mem0.ai/blog/context-engineering-ai-agents-guide)
- [Darwin-Gödel Machine:自我改進 agent 的開放式演化](https://arxiv.org/pdf/2505.22954)
- [A Survey of Self-Evolving Agents](https://arxiv.org/pdf/2507.21046)
- [COMAP:Co-Evolving World Models 與 Agent Policy](https://arxiv.org/pdf/2606.02372)
- [TUMIX:工具混用的多代理 test-time scaling](https://arxiv.org/pdf/2510.01279)

---

# 補充二:再一層前沿(規劃法、經驗累積、對齊迴圈、離線算力)

第一批補充把重點放在「迴圈的內化 / 上層自動化 / 下層地基」。這一批補四件原文與補充一都還沒涵蓋的事:**更精緻的 prompt 層規劃法**、**跨任務的經驗累積**、**對齊/評估層的迴圈**,以及算力的**第四軸——離線(sleep-time)**。

## 四之補(2):prompt 層的規劃法譜系(ReAct 之外的選擇)

你原文把 ReAct 當 agentic 骨架,但 ReAct 的 think-act-observe 逐步來回很耗 token。以下是同一光譜上的其他點:

- **Least-to-Most**:把難題拆成**由易到難**的子問題,前一個答案餵給後一個。適合「後面步驟依賴前面結果」的推理。
- **Plan-and-Solve**:先叫模型「訂一個計畫」再「照計畫逐步解」——專治 zero-shot CoT 會**跳步**的毛病。
- **ReWOO(Reasoning WithOut Observation)**:把 agent 拆成 **Planner / Worker / Solver**,**先一次把整條推理藍圖(含要呼叫哪些工具)規劃好,再一次性執行**,不走 ReAct 的逐步往返 → 大幅省 token、少來回。是 ReAct 最重要的對照替代。
- **Self-Discover**:讓模型從約 39 個「推理模組」(分解、批判、逐步、換角度…)裡**自己組出**這題專用的推理結構,再照它解。等於「meta 層的 plan-then-execute」——先設計解法骨架,再填。

一句話:**ReAct = 邊看邊想邊做(互動任務強);ReWOO / Plan-and-Solve = 先想好再做(可規劃任務省又穩)**。

## 二之補(2):generate-then-select 的極端規模範例——AlphaCode

你原文的 best-of-N 講的是「生 N 個挑一個」。AlphaCode 把它推到極致,是一個值得記住的模板:

1. 對每題**生成上百萬份**候選程式;
2. 用題目附的範例測試**過濾掉約 99%**(← oracle 過濾);
3. 對剩下的,用「另一個模型生成的測試輸入」跑、依**輸出分群(clustering)** 去重;
4. 每個大群挑一份,submit 10 份。

關鍵洞察:**用客觀 oracle(測試)過濾 + 用行為分群去重,遠比讓模型自評挑答案準**。這正是你 until oracle passes 的精神放到 parallel 軸上——**生成靠取樣要多樣,挑選靠 oracle 要客觀**。

## 三之補(2):把 self-refine 迴圈搬進訓練期,與 rubric 化的裁判

- **Constitutional AI / RLAIF**:模型依一份「憲法(明文原則清單)」**自我批判 → 自我修正**產生資料,再用 **AI(而非人)** 產生的偏好做 RL。本質是你的 self-refine 迴圈被搬到**訓練期**,且回饋來自「明文原則」這個半客觀 oracle,而非模型的憑感覺。
- **Rubric-based reward / agent-as-judge**:給裁判一份**明確評分量表(rubric)**,逐項打分再加權(chain-of-rubrics,如 RM-R1),比「給個籠統分數」穩、可解釋。**對沒有硬 oracle 的開放式任務特別有用**——把主觀品質變成半客觀的逐項檢查表。這是你 judge panel 的標準化升級。

## 新族 E:跨任務的經驗累積——Voyager 與 skill library

你原文所有迴圈都在「單一任務內」打轉;前沿的一大方向是讓迴圈**跨任務累積**,每題不從零開始:

- **Voyager**(Minecraft lifelong agent)三件事:①**自動課程(automatic curriculum)** 決定下一步該學什麼、②把學會的技能寫成**可重用的程式碼存進 skill library**、③用「環境回饋 + 執行錯誤 + 自我驗證」迭代改程式。
- 為什麼是漂亮的 `until oracle passes` 實例:技能能不能用,由**環境執行**當裁判(程式在遊戲裡跑得動 = pass),不是模型自評 → 存進 library 的都是被 oracle 認證過的。
- 對你的啟發:loop-until-dry / self-refine 若能把「驗證通過的產物」沉澱成**可檢索的技能/範例庫**,下次同類任務就是站在上一輪的肩膀上。

## 新族 F:算力的第四軸——Sleep-time compute(離線預算)

前面說算力有 sequential / parallel / 內化三軸;還有第四軸:**別只在「被問的當下」(test-time)才算**。

- **Sleep-time compute**(Letta, arXiv:2504.13171):讓 agent 在**閒置時**先把 raw context 消化成 learned context——預讀 codebase、整理歷史對話、重寫記憶狀態。雙 agent 設計:一個負責即時服務,一個「睡眠 agent」在 downtime 整理記憶。
- 效果:回答時用約 **1/5 token**、同預算下多約 **15% 正確率**。
- 定位:把「思考」從熱路徑(latency 敏感)移到冷路徑(離線)。與〈新族 C 記憶/context〉互補——sleep-time 是「何時做 context 整理」的答案:趁閒置做。

## 決策樹追加

- 任務**可事先規劃**(不需邊做邊看環境)→ 用 **ReWOO / Plan-and-Solve**,別用 ReAct 的逐步往返(省 token)。
- 需要**依賴鏈**的多步推理 → **Least-to-Most**。
- 不知道用哪種推理結構 → **Self-Discover** 讓模型自己組。
- 有便宜的客觀 oracle 且可大量生成 → **AlphaCode 式**:狂生 + oracle 過濾 + 分群去重。
- 開放式、沒硬 oracle → **rubric-based judge**(把主觀拆成逐項量表)。
- 同類任務會**重複出現** → 建 **skill library**(Voyager 式),把 oracle 認證過的產物沉澱重用。
- 有**閒置時間 / 可預讀的持久 context** → **sleep-time compute**,離線把 raw context 轉成 learned context。

## 補充二的一句話總結

算力有**四軸**(序列 / 並行 / 內化進權重 / 離線 sleep-time);而「規劃在前 vs 邊做邊看」(ReWOO ↔ ReAct)、「單任務 vs 跨任務累積」(self-refine ↔ Voyager skill library)是兩條讓你在既有積木上再往外長的維度。無論哪一種,**回饋/驗證的客觀性**仍是決定成敗的那條主線。

## 來源(補充二)

- [Least-to-Most Prompting](https://arxiv.org/pdf/2205.10625)
- [Plan-and-Solve Prompting(arXiv:2305.04091)](https://arxiv.org/abs/2305.04091)
- [ReWOO:Decoupling Reasoning from Observations](https://arxiv.org/pdf/2305.18323) · [IBM 解說](https://www.ibm.com/think/topics/rewoo)
- [Self-Discover:自組推理結構(NeurIPS 2024)](https://arxiv.org/abs/2402.03620)
- [AlphaCode:Competition-Level Code Generation](https://arxiv.org/abs/2203.07814) · [Science 版](https://www.science.org/doi/10.1126/science.abq1158)
- [Constitutional AI:Harmlessness from AI Feedback(RLAIF)](https://arxiv.org/abs/2212.08073)
- [Rubric-Based Rewards(概念)](https://www.emergentmind.com/topics/rubric-based-rewards) · [LongTraceRL(rubric reward 實例)](https://arxiv.org/abs/2605.31584)
- [Voyager:Open-Ended Embodied Agent(skill library)](https://arxiv.org/abs/2305.16291)
- [Sleep-time Compute:Beyond Inference Scaling at Test-time](https://arxiv.org/abs/2504.13171) · [Letta blog](https://www.letta.com/blog/sleep-time-compute/)
