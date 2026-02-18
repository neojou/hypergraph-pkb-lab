# Hypergraph PKB Lab 專案概述  
（供新對話中的 AI Agent 快速理解背景）

## 專案基本資訊
- Repository: https://github.com/neojou/hypergraph-pkb-lab  
- 作者：Neo Jou (@NeoJou)  
- 建立時間：約 2026 年 2 月  
- 目前階段：**Phase 0 – Research & Design**（幾乎沒有實作程式碼）  
- 目標使用者：個人長期知識管理（Personal Knowledge Base）  
- 核心技術方向：**超圖（Hypergraph） + 多智能體（Multi-Agent）AI**

## 核心理念
傳統筆記工具（如 Obsidian、Logseq、Roam）以「單篇文件／雙向連結」為主，表達能力有限。  
本專案希望用 **超圖（Hypergraph）** 來更自然地表達複雜知識：

- 一個 hyperedge 可以同時連接 **多個節點**（n-ary relation），而非只有二元關係  
- 更適合表示：事件、因果、多方參與、命題、情境、策略等高階結構  
- 最終目標：讓個人知識庫能支援更強的跨領域推理與自動化發現

## 主要靈感來源
1. **MIT 2026 論文**  
   - Higher-Order Knowledge Representations for Agentic Scientific Reasoning  
   - arXiv: 2601.04848  
2. MIT 相關開源實作  
   - lamm-mit/HyperGraphReasoning  
   - SciAgentsDiscovery  
3. 其他重要論文  
   - SciAgents (2024)  
   - Topological Deep Learning (2022)

## 規劃中的核心架構
- **三智能體協作系統**（參考 SciAgents 架構）
  - **GraphAgent**（導航者／Navigator）：負責超圖探索、路徑規劃
  - **Engineer**（分析者／Analyst）：執行具體計算、驗證、模式提取
  - **Hypothesizer**（假設生成者）：提出新假設、連結可能性
- **Orchestrator**：總控智能體，負責任務分解、路由、協調與迭代
- **Local Hypergraph Storage Layer**（本地優先）
  - 儲存：Nodes、Hyperedges、Embeddings、原始文件 chunk、metadata
  - 設計原則：local-first，外部搜尋為輔助（用完回寫本地）
- **資料流主要路徑**
  1. 攝取（Ingestion）：文字／PDF → LLM → 結構化超圖元素
  2. 儲存：寫入本地超圖資料庫
  3. 查詢／推理：多智能體協作在超圖上進行高階推理

## 目前專案狀態（2026 年 2 月）
- 只有 **1 commit**（初始化）
- 主要內容集中在 `docs/` 資料夾：
  - design/ → schema 草稿、agent 角色定義
  - notes/ → 論文閱讀筆記、靈感整理
  - roadmap/ → 里程碑規劃
- `src/` 目前只有空目錄結構，尚未有實作程式碼

## 預期目錄結構
```
hypergraph-pkb-lab/
├── docs/
│   ├── design/         # schema、agent prompt、流程圖
│   ├── notes/          # 論文筆記、概念驗證
│   └── roadmap/        # 階段性目標
├── src/
│   ├── agents/         # 各智能體邏輯
│   ├── storage/        # 超圖資料庫層（可能用 NetworkX / HyperNetX / 自訂）
│   ├── ingestion/      # 文字 → 超圖元素的轉換
│   ├── query/          # 查詢與多輪推理
│   └── orchestrator/   # 總控邏輯
├── examples/           # 範例知識庫、測試案例
└── tests/
```

## Roadmap 主要里程碑（E1–E4）
- **E1**：徹底解構 MIT Hypergraph 論文 + 相關實作，內化核心概念  
- **E2**：設計適合個人的知識 Schema（Node 類型、Hyperedge 類型、屬性）  
- **E3**：打造最小可行原型（Hypergraph Storage + LLM Ingestion pipeline）  
- **E4**：實作三智能體 + Orchestrator，開始跑端到端實驗

# MIT HyperGraphReasoning 論文快速總覽（arXiv:2601.04878）

**論文標題**：Higher-Order Knowledge Representations for Agentic Scientific Reasoning  
**作者**：Isabella A. Stewart, Markus J. Buehler (MIT Laboratory for Atomistic and Molecular Mechanics, LAMM)  
**發布日期**：2026 年 1 月 8 日  
**應用領域示範**：biocomposite scaffolds（生物複合支架材料），從約 1,100 篇論文建構 hypergraph  
**官方實作 repo**：https://github.com/lamm-mit/HyperGraphReasoning  
**論文連結**：https://arxiv.org/abs/2601.04878

## 核心問題與貢獻

傳統 pairwise Knowledge Graph (KG) 在科學領域的痛點：
- 無法直接捕捉 higher-order（多於兩個實體）的交互關係 → 導致 combinatorial explosion（成對展開時爆炸性增長）
- LLM 在 RAG 時缺乏結構深度，容易 hallucinate 或遺漏跨域機制連結

論文提出解決方案：
- 使用 **hypergraph**（超圖）來表示知識：一個 hyperedge 可同時連接多個 nodes（實體），自然保留科學配方/機制中的多體關係
- 建構出具 **scale-free topology** 的全球 hypergraph（power law exponent ≈ 1.23），有明顯的 conceptual hubs（概念樞紐）
- 實現 **teacherless**（無需額外微調/訓練）的 agentic reasoning 系統：hypergraph 拓撲本身作為可驗證的 guardrail（護欄）

主要統計（biocomposite scaffolds 語料）：
- Nodes：161,172
- Hyperedges：320,201

## 整體系統架構 / 流程（input → output）

1. **Input**：大量科學論文（PDFs，例如 ~1,100 篇 biocomposite scaffolds 相關論文）
2. **Preprocessing**：
    - PDF → Markdown（使用 marker 工具）
    - 實體/關係提取（LLM 驅動，例如 Llama 系列）
3. **Hypergraph Construction**：
    - Nodes：科學概念、材料、方法、屬性等
    - Hyperedges：從文本中提取的多實體共現/機制關係（higher-order）
    - Cleaning：移除噪音、確保品質（細節在 Methods）
    - 輸出：HypernetX 相容的 hypergraph（.pkl 格式），附 embeddings（nomic-embed-text-v1.5）
4. **Hypergraph Analysis**：
    - 驗證 scale-free 特性、找出 hubs
    - 計算 degree、centrality 等
5. **Agentic Reasoning（多代理推理）**：
    - 使用 **hypergraph traversal** 工具：以 node-intersection size (IS) 作為約束（例如 IS=1 或 IS=2）
    - 參數：K = 最短路徑數量（常設 K=1）
    - 代理角色示例：
        - GraphAgent：執行 traversal，找出 hyperedge paths
        - Engineer Agent：解讀 intersection nodes 作為 mechanistic bridge
        - Hypothesizer Agent：根據路徑提出假說與實驗設計
    - 工具：HypernetX（traversal）、AutoGen 或類似框架（multi-agent）
    - 範例 query 與輸出：
        - “cerium oxide 如何與 PCL scaffolds 相關？” → 透過 chitosan 作為橋接（intersection node）
        - “grass 如何與 PCL 相關？” → 透過 biomass → methanol → PCL precipitation 的機制路徑
6. **Output**：grounded mechanistic hypotheses（有根據的機制假說），包含實驗 workflow 建議

## 最關鍵的 3–5 個設計決策（Key Design Choices）

1. **採用 hypergraph 而非 pairwise graph**：直接捕捉 multi-entity 關係，避免展開爆炸並保留科學上下文
2. **Node-intersection 約束的 traversal**：以共享節點數 (IS) 控制路徑品質，避免無意義跳躍，實現 semantically distant concepts 的可靠橋接
3. **Teacherless agentic 框架**：不需額外訓練 LLM，只靠 hypergraph 拓撲 + traversal 工具提供可驗證的 reasoning guardrail
4. **單一 global hypergraph + domain-specific agents**：集中知識於一個結構化圖譜，再由多代理（graph-aware + domain experts）協作
5. **LLM-driven construction + embedding**：用強大 LLM 提取 higher-order 關係，結合 sentence embeddings 支援語義搜尋/相似度

## 相關工具與技術棧（來自官方 repo）

- **Hypergraph 處理**：HypernetX
- **Embeddings**：nomic-ai/nomic-embed-text-v1.5
- **LLM**：Llama-4-Maverick-17B、Llama-3.3-70B 等（Together API 或 local）
- **PDF 處理**：marker (PDF → Markdown)
- **代理框架**：AutoGen（推測，多代理對話式推理）
- **分析 Notebook**：make_hypergraph.ipynb、analyze_hypergraph.ipynb、Agents.ipynb

## 待辦 / 我們專案的對應目標（hypergraph-pkb-lab）

- 粗讀論文主要章節（Intro + Method + Results）
- 畫出完整流程圖（Mermaid）：input documents → preprocessing → hypergraph construction → cleaning → traversal → multi-agent reasoning → hypothesis output
- 列出 3–5 個 key design choices（如上）
- 筆記存放：`docs/notes/E1-mit-paper-overview.md`
- Issue：https://github.com/neojou/hypergraph-pkb-lab/issues/1 （目前 open，高優先）

## 其他重要特性／立場
- 強調**跨領域整合**（交易策略、技術架構、哲學、日常生活皆可放入同一超圖）
- **本地優先**，隱私優先，外部知識為可選增強
- 不開放外部貢獻（目前為個人研究實驗專案）
- 歡迎討論、想法碰撞、平行參考

## 一句總結
這是一個極早期、研究導向的個人專案，目標是把 MIT 最新的超圖推理＋多智能體科學發現框架，轉化成適合個人長期使用、本地優先的下一代知識庫系統。目前仍處於大量閱讀、思考、設計階段，尚未進入 coding 階段。

歡迎新 Agent 基於以上背景，繼續推進討論、提問、建議 schema、prompt、技術選型等。
