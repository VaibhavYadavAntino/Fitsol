# FitSol ESG Co-Pilot - Technical Documentation

## Table of Contents

1. [System Architecture Overview](#system-architecture-overview)
2. [Agentic Architecture](#agentic-architecture)
3. [Query Processing Flow](#query-processing-flow)
4. [Agent Flows](#agent-flows)
   - [Carbon Accounting Agent](#carbon-accounting-agent)
   - [Benchmarking Agent](#benchmarking-agent)
   - [Net Zero Planner](#net-zero-planner)
5. [Tool Router Architecture](#tool-router-architecture)
6. [File Context Handling](#file-context-handling)
7. [Data Flow Diagrams](#data-flow-diagrams)
8. [Component Interactions](#component-interactions)

---

## System Architecture Overview

### High-Level Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        UI[React Frontend]
        API[FastAPI Backend]
    end
    
    subgraph "AI Processing Layer"
        L1[Layer 1: Query Classification]
        L2[Layer 2: RAG + Tool Router]
        TR[Tool Router]
    end
    
    subgraph "Agent Layer"
        CA[Carbon Accounting Agent]
        BA[Benchmarking Agent]
        NZ[Net Zero Planner]
    end
    
    subgraph "Calculation Engines"
        CCE[Carbon Calculation Engine]
        BCE[Benchmarking Calculation Engine]
        SBTI[SBTi Pathway Calculator]
    end
    
    subgraph "Data Layer"
        VS[Vector Store<br/>ChromaDB]
        MDB[(MongoDB)]
        SQL[(SQLite<br/>Emission Factors)]
        FS[File Storage]
    end
    
    subgraph "External Services"
        LLM[Google Gemini API]
        WEB[Web Search<br/>Optional]
    end
    
    UI --> API
    API --> L1
    L1 --> L2
    L2 --> TR
    TR --> CA
    TR --> BA
    TR --> NZ
    
    CA --> CCE
    BA --> BCE
    NZ --> SBTI
    
    CCE --> SQL
    BCE --> SQL
    SBTI --> SQL
    
    CA --> MDB
    BA --> MDB
    NZ --> MDB
    
    L2 --> VS
    TR --> LLM
    CA --> LLM
    BA --> LLM
    NZ --> LLM
    
    CA --> WEB
    BA --> WEB
    
    L2 --> FS
    
    style CA fill:#e3f2fd
    style BA fill:#f3e5f5
    style NZ fill:#e8f5e9
    style TR fill:#fff3e0
```

---

## Agentic Architecture

### Complete Agentic System Architecture

```mermaid
graph TB
    subgraph "Input Layer"
        UQ[User Query]
        CM[Conversation Memory]
        CTX[Document Context]
        FILES[Uploaded Files]
    end
    
    subgraph "Layer 1: Query Classification"
        L1_CLASS[Query Classifier<br/>Gemini 2.0 Flash]
        L1_DEC{Classification Type}
        GREET[Greeting Handler]
        MEM[Memory Answer]
        CLAR[Clarification UI]
        ENH[Query Enhancer]
    end
    
    subgraph "Layer 2: RAG Processing"
        FEW[Few-Shot Matcher]
        VEC[Vector Search<br/>ChromaDB]
        RET[Context Retrieval]
        TOOL_DEC{Tool Needed?}
    end
    
    subgraph "Tool Router"
        TR_LLM[Tool Router LLM<br/>Gemini 2.5 Flash]
        TR_SCHEMA[JSON Schema Validator]
        TR_DEC{Route Decision}
    end
    
    subgraph "Agent Selection"
        CA_AGENT[Carbon Accounting Agent]
        BA_AGENT[Benchmarking Agent]
        NZ_AGENT[Net Zero Planner]
        CHART[Chart Generator]
        OTHER[Other Tools]
    end
    
    subgraph "Carbon Accounting Agent Flow"
        CA_EXTRACT[Data Extractor]
        CA_CLAR[Clarification Orchestrator]
        CA_CALC[Carbon Calculation Engine]
        CA_STORE[Result Storage]
        CA_EXPL[Explanation Generator]
        CA_VIZ[Visualization Helper]
    end
    
    subgraph "Benchmarking Agent Flow"
        BA_EXTRACT[Data Extractor]
        BA_CLAR[Clarification Orchestrator]
        BA_FILE[File-Based Orchestrator]
        BA_CALC[Benchmarking Engine]
        BA_STORE[Result Storage]
        BA_EXPL[Explanation Generator]
        BA_VIZ[Visualization Helper]
    end
    
    subgraph "Net Zero Planner Flow"
        NZ_EXTRACT[Data Extractor]
        NZ_CLAR[Clarification Orchestrator]
        NZ_SBTI[SBTi Pathway Calculator]
        NZ_INIT[Initiative Scorer]
        NZ_STORE[Plan Storage]
        NZ_EXPL[Explanation Generator]
    end
    
    subgraph "Output Layer"
        RESP[Response Generator]
        PROOF[Calculation Proof]
        CHARTS[Charts/Visualizations]
        STREAM[Streamed Response]
    end
    
    UQ --> L1_CLASS
    CM --> L1_CLASS
    CTX --> L1_CLASS
    FILES --> L1_CLASS
    
    L1_CLASS --> L1_DEC
    L1_DEC -->|Greeting| GREET
    L1_DEC -->|Memory| MEM
    L1_DEC -->|Clarification| CLAR
    L1_DEC -->|Enhanced| ENH
    
    ENH --> FEW
    FEW -->|Match| RESP
    FEW -->|No Match| VEC
    VEC --> RET
    RET --> TOOL_DEC
    
    TOOL_DEC -->|Yes| TR_LLM
    TOOL_DEC -->|No| RESP
    
    TR_LLM --> TR_SCHEMA
    TR_SCHEMA --> TR_DEC
    
    TR_DEC -->|Carbon| CA_AGENT
    TR_DEC -->|Benchmark| BA_AGENT
    TR_DEC -->|Net Zero| NZ_AGENT
    TR_DEC -->|Chart| CHART
    TR_DEC -->|Other| OTHER
    
    CA_AGENT --> CA_EXTRACT
    CA_EXTRACT --> CA_CLAR
    CA_CLAR -->|Complete| CA_CALC
    CA_CLAR -->|Incomplete| CLAR
    CA_CALC --> CA_STORE
    CA_STORE --> CA_EXPL
    CA_EXPL --> CA_VIZ
    
    BA_AGENT --> BA_EXTRACT
    BA_EXTRACT --> BA_CLAR
    BA_EXTRACT --> BA_FILE
    BA_CLAR -->|Complete| BA_CALC
    BA_FILE -->|Complete| BA_CALC
    BA_CLAR -->|Incomplete| CLAR
    BA_CALC --> BA_STORE
    BA_STORE --> BA_EXPL
    BA_EXPL --> BA_VIZ
    
    NZ_AGENT --> NZ_EXTRACT
    NZ_EXTRACT --> NZ_CLAR
    NZ_CLAR -->|Complete| NZ_SBTI
    NZ_CLAR -->|Incomplete| CLAR
    NZ_SBTI --> NZ_INIT
    NZ_INIT --> NZ_STORE
    NZ_STORE --> NZ_EXPL
    
    CA_VIZ --> RESP
    BA_VIZ --> RESP
    NZ_EXPL --> RESP
    
    RESP --> PROOF
    RESP --> CHARTS
    PROOF --> STREAM
    CHARTS --> STREAM
    
    style CA_AGENT fill:#e3f2fd
    style BA_AGENT fill:#f3e5f5
    style NZ_AGENT fill:#e8f5e9
    style TR_LLM fill:#fff3e0
    style L1_CLASS fill:#fce4ec
```

---

## Query Processing Flow

### Complete Query Processing Pipeline

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant API
    participant Layer1
    participant FewShot
    participant VectorStore
    participant ToolRouter
    participant Agent
    participant CalcEngine
    participant Database
    participant LLM
    
    User->>Frontend: Submit Query
    Frontend->>API: POST /chat/query
    
    API->>FewShot: Check Match
    alt Few-Shot Match Found
        FewShot-->>API: Instant Answer
        API-->>Frontend: Stream Response
        Frontend-->>User: Display Answer
    else No Match
        API->>Layer1: Classify Query
        Layer1->>LLM: Analyze Query Type
        
        alt Greeting
            LLM-->>Layer1: Greeting Response
            Layer1-->>API: Direct Response
            API-->>Frontend: Stream Response
        else Memory Answer
            LLM-->>Layer1: Answer from Memory
            Layer1-->>API: Memory Response
            API-->>Frontend: Stream Response
        else Clarification Needed
            LLM-->>Layer1: Clarification Questions
            Layer1-->>API: Clarification UI
            API-->>Frontend: Show Questions
            Frontend-->>User: Display Questions
            User->>Frontend: Provide Answers
            Frontend->>API: Enhanced Query
        else Enhanced Query
            LLM-->>Layer1: Enhanced Query
            Layer1->>VectorStore: Semantic Search
            VectorStore-->>Layer1: Retrieved Context
            Layer1->>ToolRouter: Route Query
            
            ToolRouter->>LLM: Determine Tool
            LLM-->>ToolRouter: Tool Selection
            
            alt Tool Selected
                ToolRouter->>Agent: Execute Agent
                Agent->>LLM: Extract Data
                LLM-->>Agent: Structured Data
                
                alt Data Incomplete
                    Agent-->>ToolRouter: Clarification Needed
                    ToolRouter-->>API: Clarification
                    API-->>Frontend: Show Questions
                else Data Complete
                    Agent->>CalcEngine: Perform Calculation
                    CalcEngine->>Database: Query Data
                    Database-->>CalcEngine: Emission Factors/Benchmarks
                    CalcEngine-->>Agent: Calculation Results
                    Agent->>Database: Store Results
                    Agent->>LLM: Generate Explanation
                    LLM-->>Agent: Natural Language Response
                    Agent-->>ToolRouter: Agent Response
                    ToolRouter-->>API: Combined Response
                    API-->>Frontend: Stream Response
                    Frontend-->>User: Display Answer
                end
            else No Tool
                ToolRouter->>LLM: Direct RAG Response
                LLM-->>ToolRouter: RAG Answer
                ToolRouter-->>API: Response
                API-->>Frontend: Stream Response
            end
        end
    end
```

---

## Agent Flows

### Carbon Accounting Agent

#### Complete Carbon Accounting Agent Flow

```mermaid
graph TB
    START([User Query:<br/>Carbon Footprint]) --> INIT[Initialize Agent]
    
    INIT --> EXTRACT[Data Extraction]
    EXTRACT --> EXTRACT_METHOD{Extraction Method}
    EXTRACT_METHOD -->|Regex| REGEX[Regex Extraction]
    EXTRACT_METHOD -->|LLM| LLM_EXTRACT[LLM Extraction]
    
    REGEX --> EXTRACTED[Extracted Data]
    LLM_EXTRACT --> EXTRACTED
    
    EXTRACTED --> FILE_CHECK{File Context?}
    FILE_CHECK -->|Yes| FILE_ENHANCE[Enhance with File Data]
    FILE_CHECK -->|No| CLAR_CHECK
    
    FILE_ENHANCE --> CLAR_CHECK[Clarification Check]
    
    CLAR_CHECK --> CLAR_ORCH[Clarification Orchestrator]
    CLAR_ORCH --> GATHER_SUGG[Gather Suggestions]
    GATHER_SUGG --> DB_SUGG[Database Suggestions]
    GATHER_SUGG --> KB_SUGG[Knowledge Base Search]
    GATHER_SUGG --> WEB_SUGG[Web Search<br/>Optional]
    
    DB_SUGG --> CLAR_DECIDE{Decision}
    KB_SUGG --> CLAR_DECIDE
    WEB_SUGG --> CLAR_DECIDE
    
    CLAR_DECIDE -->|Ask| CLAR_QUEST[Generate Questions]
    CLAR_QUEST --> CLAR_RESP[Return Clarification]
    CLAR_RESP --> USER_INPUT[User Provides Info]
    USER_INPUT --> EXTRACT
    
    CLAR_DECIDE -->|Proceed| DATA_COMPLETE{Data Complete?}
    
    DATA_COMPLETE -->|No| CLAR_QUEST
    DATA_COMPLETE -->|Yes| DATA_SEL[Data Selection Disclosure]
    
    DATA_SEL --> EMISSION_OPTS[Gather Emission Factor Options]
    EMISSION_OPTS --> DB_FACTORS[Database Factors]
    EMISSION_OPTS --> WEB_FACTORS[Internet Search<br/>Optional]
    
    DB_FACTORS --> SHOW_DISCLOSURE[Show Pre-Calculation Disclosure]
    WEB_FACTORS --> SHOW_DISCLOSURE
    
    SHOW_DISCLOSURE --> USER_CONFIRM{User Confirms?}
    USER_CONFIRM -->|No| CLAR_RESP
    USER_CONFIRM -->|Yes| CALC_TYPE{Calculation Type}
    
    CALC_TYPE -->|Scope 1| SCOPE1[Calculate Scope 1]
    CALC_TYPE -->|Scope 2| SCOPE2[Calculate Scope 2]
    CALC_TYPE -->|Scope 3| SCOPE3[Calculate Scope 3]
    CALC_TYPE -->|Total| TOTAL[Calculate Total Footprint]
    
    SCOPE1 --> LOOKUP[Lookup Emission Factors]
    SCOPE2 --> LOOKUP
    SCOPE3 --> LOOKUP
    TOTAL --> LOOKUP
    
    LOOKUP --> CALC_ENGINE[Carbon Calculation Engine]
    CALC_ENGINE --> FORMULA[Apply GHG Protocol Formulas]
    FORMULA --> RESULT[Calculation Result]
    
    RESULT --> STORE[Store in MongoDB]
    STORE --> PROOF[Generate Calculation Proof]
    PROOF --> EXPLAIN[Explanation Generator]
    
    EXPLAIN --> LLM_EXPLAIN[LLM Formats Response]
    LLM_EXPLAIN --> VIZ[Visualization Helper]
    
    VIZ --> GEN_CHARTS[Generate Charts]
    GEN_CHARTS --> PIE[Pie Chart<br/>Scope Breakdown]
    GEN_CHARTS --> BAR[Bar Chart<br/>Activity Comparison]
    
    PIE --> COMBINE[Combine Response]
    BAR --> COMBINE
    LLM_EXPLAIN --> COMBINE
    PROOF --> COMBINE
    
    COMBINE --> RESPONSE([Return Response])
    
    style EXTRACT fill:#e3f2fd
    style CALC_ENGINE fill:#c8e6c9
    style CLAR_ORCH fill:#fff3e0
    style VIZ fill:#f3e5f5
```

#### Carbon Accounting Agent - Detailed Component Flow

```mermaid
graph LR
    subgraph "Input Processing"
        Q[Query] --> DE[Data Extractor]
        C[Context] --> DE
        M[Memory] --> DE
    end
    
    subgraph "Data Extraction"
        DE --> REGEX[Regex Patterns]
        DE --> LLM_EXT[LLM Extraction]
        REGEX --> MERGE[Merge Results]
        LLM_EXT --> MERGE
    end
    
    subgraph "Clarification System"
        MERGE --> CO[Clarification Orchestrator]
        CO --> ANALYZE[Analyze Completeness]
        ANALYZE --> SUGG[Gather Suggestions]
        SUGG --> DECIDE{Complete?}
        DECIDE -->|No| QUESTIONS[Generate Questions]
        DECIDE -->|Yes| CALC
    end
    
    subgraph "Calculation Engine"
        CALC[Start Calculation] --> DS[Data Selection Helper]
        DS --> OPTIONS[Emission Factor Options]
        OPTIONS --> DB_LOOKUP[Database Lookup]
        OPTIONS --> WEB_SEARCH[Web Search]
        DB_LOOKUP --> SELECT[User Selection]
        WEB_SEARCH --> SELECT
        SELECT --> CCE[Carbon Calculation Engine]
        CCE --> SCOPE1[Scope 1 Calc]
        CCE --> SCOPE2[Scope 2 Calc]
        CCE --> SCOPE3[Scope 3 Calc]
        CCE --> TOTAL[Total Calc]
    end
    
    subgraph "Output Generation"
        SCOPE1 --> STORE[Store Results]
        SCOPE2 --> STORE
        SCOPE3 --> STORE
        TOTAL --> STORE
        STORE --> EG[Explanation Generator]
        EG --> VH[Visualization Helper]
        VH --> CHARTS[Charts]
        EG --> PROOF[Calculation Proof]
        CHARTS --> RESPONSE[Final Response]
        PROOF --> RESPONSE
    end
    
    style DE fill:#e3f2fd
    style CO fill:#fff3e0
    style CCE fill:#c8e6c9
    style EG fill:#f3e5f5
```

---

### Benchmarking Agent

#### Complete Benchmarking Agent Flow

```mermaid
graph TB
    START([User Query:<br/>Benchmark Request]) --> INIT[Initialize Agent]
    
    INIT --> FILE_CHECK{File Context?}
    
    FILE_CHECK -->|Yes| FILE_ORCH[File-Based Orchestrator]
    FILE_CHECK -->|No| STD_ORCH[Standard Orchestrator]
    
    FILE_ORCH --> FILE_ANALYZE[Analyze File Content]
    FILE_ANALYZE --> EXTRACT_FILE[Extract Companies/Metrics]
    EXTRACT_FILE --> FILE_DECIDE{Action?}
    
    FILE_DECIDE -->|Ask| FILE_QUEST[Generate File Questions]
    FILE_QUEST --> FILE_RESP[Return Clarification]
    FILE_RESP --> USER_INPUT[User Provides Info]
    USER_INPUT --> FILE_ORCH
    
    FILE_DECIDE -->|Proceed| FILE_EXTRACT[Extract Comparison Params]
    FILE_EXTRACT --> FILE_DATA[File Data Ready]
    
    STD_ORCH --> EXTRACT[Data Extraction]
    EXTRACT --> EXTRACT_METHOD{Extraction Method}
    EXTRACT_METHOD -->|LLM| LLM_EXTRACT[LLM Extraction]
    EXTRACT_METHOD -->|Regex| REGEX_EXTRACT[Regex Extraction]
    
    LLM_EXTRACT --> EXTRACTED[Extracted Data]
    REGEX_EXTRACT --> EXTRACTED
    
    FILE_DATA --> EXTRACTED
    EXTRACTED --> NORMALIZE[Normalize Data]
    NORMALIZE --> VALIDATE[Validate Extraction]
    
    VALIDATE --> VALID_ERROR{Valid?}
    VALID_ERROR -->|No| ERROR_RESP[Return Error]
    VALID_ERROR -->|Yes| CLAR_CHECK[Clarification Check]
    
    CLAR_CHECK --> CLAR_ORCH[Clarification Orchestrator]
    CLAR_ORCH --> GATHER_SUGG[Gather Suggestions]
    GATHER_SUGG --> DB_SUGG[Database Suggestions]
    GATHER_SUGG --> KB_SUGG[Knowledge Base]
    
    DB_SUGG --> CLAR_DECIDE{Decision}
    KB_SUGG --> CLAR_DECIDE
    
    CLAR_DECIDE -->|Ask| CLAR_QUEST[Generate Questions]
    CLAR_QUEST --> CLAR_RESP[Return Clarification]
    CLAR_RESP --> USER_INPUT2[User Provides Info]
    USER_INPUT2 --> EXTRACT
    
    CLAR_DECIDE -->|Proceed| DATA_COMPLETE{Data Complete?}
    
    DATA_COMPLETE -->|No| CLAR_QUEST
    DATA_COMPLETE -->|Yes| BENCH_TYPE{Benchmark Type}
    
    BENCH_TYPE -->|Single Metric| SINGLE[Single Metric Benchmark]
    BENCH_TYPE -->|Peer Comparison| PEER[Peer Comparison]
    BENCH_TYPE -->|Gap Analysis| GAP[Gap Analysis]
    BENCH_TYPE -->|Multi Metric| MULTI[Multi Metric]
    
    SINGLE --> DATA_SEL[Data Selection Disclosure]
    PEER --> DATA_SEL
    GAP --> DATA_SEL
    MULTI --> DATA_SEL
    
    DATA_SEL --> BENCH_OPTS[Gather Benchmark Options]
    BENCH_OPTS --> DB_BENCH[Database Benchmarks]
    BENCH_OPTS --> WEB_BENCH[Internet Search<br/>Optional]
    
    DB_BENCH --> SHOW_DISCLOSURE[Show Pre-Calculation Disclosure]
    WEB_BENCH --> SHOW_DISCLOSURE
    
    SHOW_DISCLOSURE --> USER_CONFIRM{User Confirms?}
    USER_CONFIRM -->|No| CLAR_RESP
    USER_CONFIRM -->|Yes| CALC_ENGINE[Benchmarking Calculation Engine]
    
    CALC_ENGINE --> STATS[Statistical Calculations]
    STATS --> PERCENTILE[Percentile Rank]
    STATS --> INTERPOLATE[Linear Interpolation]
    STATS --> GAP_CALC[Gap Analysis]
    
    PERCENTILE --> RESULT[Benchmark Result]
    INTERPOLATE --> RESULT
    GAP_CALC --> RESULT
    
    RESULT --> STORE[Store in MongoDB]
    STORE --> PROOF[Generate Calculation Proof]
    PROOF --> EXPLAIN[Explanation Generator]
    
    EXPLAIN --> LLM_EXPLAIN[LLM Formats Response]
    LLM_EXPLAIN --> VIZ[Visualization Helper]
    
    VIZ --> GEN_CHARTS[Generate Charts]
    GEN_CHARTS --> BAR[Bar Chart<br/>Comparison]
    GEN_CHARTS --> RADAR[Radar Chart<br/>Multi-Metric]
    GEN_CHARTS --> PERCENT[Percentile Chart]
    
    BAR --> COMBINE[Combine Response]
    RADAR --> COMBINE
    PERCENT --> COMBINE
    LLM_EXPLAIN --> COMBINE
    PROOF --> COMBINE
    
    COMBINE --> RESPONSE([Return Response])
    
    style EXTRACT fill:#f3e5f5
    style CALC_ENGINE fill:#c8e6c9
    style CLAR_ORCH fill:#fff3e0
    style FILE_ORCH fill:#e1bee7
    style VIZ fill:#f3e5f5
```

#### Benchmarking Agent - File-Based Processing Flow

```mermaid
graph TB
    START([File Uploaded]) --> ANALYZE[File Analysis]
    ANALYZE --> EXTRACT_META[Extract Metadata]
    EXTRACT_META --> COMPANIES[Companies List]
    EXTRACT_META --> METRICS[Metrics List]
    EXTRACT_META --> SECTORS[Sectors List]
    EXTRACT_META --> REGIONS[Regions List]
    
    COMPANIES --> FILE_ANALYSIS[File Analysis Object]
    METRICS --> FILE_ANALYSIS
    SECTORS --> FILE_ANALYSIS
    REGIONS --> FILE_ANALYSIS
    
    FILE_ANALYSIS --> USER_QUERY[User Query]
    USER_QUERY --> FILE_ORCH[File-Based Orchestrator]
    
    FILE_ORCH --> INTENT[Determine Intent]
    INTENT --> CLEAR{Intent Clear?}
    
    CLEAR -->|Yes| EXTRACT_PARAMS[Extract Comparison Params]
    CLEAR -->|No| GEN_QUEST[Generate Questions]
    
    GEN_QUEST --> USER_RESP[User Response]
    USER_RESP --> EXTRACT_PARAMS
    
    EXTRACT_PARAMS --> USER_COMPANY[User Company]
    EXTRACT_PARAMS --> COMPARE_WITH[Compare With]
    EXTRACT_PARAMS --> METRICS_LIST[Metrics List]
    
    USER_COMPANY --> MATCH_FILE[Match with File Data]
    COMPARE_WITH --> MATCH_FILE
    METRICS_LIST --> MATCH_FILE
    
    MATCH_FILE --> EXTRACT_VALUES[Extract Metric Values]
    EXTRACT_VALUES --> NORMALIZE[Normalize Metric Names]
    NORMALIZE --> READY[Data Ready for Calculation]
    
    READY --> BENCH_CALC[Benchmarking Calculation]
    
    style FILE_ORCH fill:#e1bee7
    style MATCH_FILE fill:#f3e5f5
    style BENCH_CALC fill:#c8e6c9
```

---

### Net Zero Planner

#### Complete Net Zero Planner Flow

```mermaid
graph TB
    START([User Query:<br/>Net Zero Plan]) --> INIT[Initialize Agent]
    
    INIT --> EXTRACT[Data Extraction]
    EXTRACT --> EXTRACT_METHOD{Extraction Method}
    EXTRACT_METHOD -->|LLM| LLM_EXTRACT[LLM Extraction]
    EXTRACT_METHOD -->|Regex| REGEX_EXTRACT[Regex Extraction]
    
    LLM_EXTRACT --> EXTRACTED[Extracted Data]
    REGEX_EXTRACT --> EXTRACTED
    
    EXTRACTED --> FILE_CHECK{File Context?}
    FILE_CHECK -->|Yes| FILE_ENHANCE[Enhance with File Data]
    FILE_CHECK -->|No| CLAR_CHECK
    
    FILE_ENHANCE --> EXTRACT_EMISSIONS[Extract Historical Emissions]
    EXTRACT_EMISSIONS --> BASELINE[Baseline Year Emissions]
    EXTRACT_EMISSIONS --> CURRENT[Current Year Emissions]
    
    BASELINE --> EXTRACTED
    CURRENT --> EXTRACTED
    
    CLAR_CHECK[Clarification Check] --> HAS_EMISSIONS{Has Emissions?}
    
    HAS_EMISSIONS -->|No| TRY_CARBON[Try Carbon Agent]
    TRY_CARBON --> CARBON_MEMORY[Check Memory]
    CARBON_MEMORY --> FOUND{Found?}
    FOUND -->|Yes| EXTRACTED
    FOUND -->|No| KB_SEARCH[Knowledge Base Search]
    KB_SEARCH --> KB_FOUND{Found?}
    KB_FOUND -->|Yes| EXTRACTED
    KB_FOUND -->|No| CLAR_ORCH
    
    HAS_EMISSIONS -->|Yes| VALIDATE[Validate Inputs]
    VALIDATE --> VALID_ERROR{Valid?}
    VALID_ERROR -->|No| ERROR_RESP[Return Error]
    VALID_ERROR -->|Yes| SCENARIO_CHECK
    
    SCENARIO_CHECK{Scenario Selected?} -->|No| SCENARIO_DISCLOSURE[Show SBTi Scenario Options]
    SCENARIO_DISCLOSURE --> SCENARIO_QUEST[Scenario Questions]
    SCENARIO_QUEST --> USER_SELECT[User Selects Scenario]
    USER_SELECT --> SCENARIO_CHECK
    
    SCENARIO_CHECK -->|Yes| SBTI_CALC[SBTi Pathway Calculator]
    
    SBTI_CALC --> TARGET_TEMP[Target Temperature]
    TARGET_TEMP -->|1.5°C| PATHWAY_15[1.5°C Pathway]
    TARGET_TEMP -->|Well-Below 2°C| PATHWAY_2[2°C Pathway]
    
    PATHWAY_15 --> REDUCTION_RATE[7% Annual Reduction]
    PATHWAY_2 --> REDUCTION_RATE_2[5% Annual Reduction]
    
    REDUCTION_RATE --> CALC_PATHWAY[Calculate Pathway]
    REDUCTION_RATE_2 --> CALC_PATHWAY
    
    CALC_PATHWAY --> MILESTONES[Calculate Milestones]
    MILESTONES --> MIL_2030[2030: 50% Reduction]
    MILESTONES --> MIL_2040[2040: 75% Reduction]
    MILESTONES --> MIL_NZ[2050: Net Zero]
    
    MIL_2030 --> VALIDATE_PATH[Validate Pathway]
    MIL_2040 --> VALIDATE_PATH
    MIL_NZ --> VALIDATE_PATH
    
    VALIDATE_PATH --> VALID{Valid?}
    VALID -->|No| ERROR_PATH[Pathway Error]
    VALID -->|Yes| INITIATIVE_SCORER[Initiative Scorer]
    
    INITIATIVE_SCORER --> GET_TEMPLATES[Get Initiative Templates]
    GET_TEMPLATES --> SCORE[Score Initiatives]
    SCORE --> IMPACT[Impact Score]
    SCORE --> COST[Cost Score]
    SCORE --> FEASIBILITY[Feasibility Score]
    
    IMPACT --> MULTI_CRITERIA[Multi-Criteria Analysis]
    COST --> MULTI_CRITERIA
    FEASIBILITY --> MULTI_CRITERIA
    
    MULTI_CRITERIA --> RANK[Rank Initiatives]
    RANK --> RECOMMEND[Recommend Top Initiatives]
    
    RECOMMEND --> PLAN[Create Net Zero Plan]
    PLAN --> STORE[Store in MongoDB]
    STORE --> PROOF[Generate Calculation Proof]
    PROOF --> EXPLAIN[Explanation Generator]
    
    EXPLAIN --> LLM_EXPLAIN[LLM Formats Response]
    LLM_EXPLAIN --> VIZ[Visualization Helper]
    
    VIZ --> GEN_CHARTS[Generate Charts]
    GEN_CHARTS --> PATHWAY_CHART[Pathway Chart]
    GEN_CHARTS --> INITIATIVE_CHART[Initiative Chart]
    
    PATHWAY_CHART --> COMBINE[Combine Response]
    INITIATIVE_CHART --> COMBINE
    LLM_EXPLAIN --> COMBINE
    PROOF --> COMBINE
    
    COMBINE --> RESPONSE([Return Response])
    
    CLAR_ORCH[Clarification Orchestrator] --> CLAR_QUEST[Generate Questions]
    CLAR_QUEST --> CLAR_RESP[Return Clarification]
    CLAR_RESP --> USER_INPUT[User Provides Info]
    USER_INPUT --> EXTRACT
    
    style EXTRACT fill:#e8f5e9
    style SBTI_CALC fill:#c8e6c9
    style INITIATIVE_SCORER fill:#fff3e0
    style CLAR_ORCH fill:#fff3e0
    style VIZ fill:#f3e5f5
```

#### Net Zero Planner - SBTi Pathway Calculation Flow

```mermaid
graph LR
    subgraph "Input"
        CE[Current Emissions]
        BYE[Base Year Emissions]
        TY[Target Year]
        TT[Target Temperature]
    end
    
    subgraph "Pathway Calculation"
        CE --> SBTI[SBTi Calculator]
        BYE --> SBTI
        TY --> SBTI
        TT --> SBTI
        
        SBTI --> RATE{Reduction Rate}
        RATE -->|1.5°C| R7[7% Annual]
        RATE -->|2°C| R5[5% Annual]
        
        R7 --> CALC[Calculate Yearly Targets]
        R5 --> CALC
        
        CALC --> Y2030[2030 Milestone]
        CALC --> Y2040[2040 Milestone]
        CALC --> Y2050[2050 Net Zero]
    end
    
    subgraph "Validation"
        Y2030 --> VAL[Validate Pathway]
        Y2040 --> VAL
        Y2050 --> VAL
        VAL --> COMPLIANCE{SBTi Compliant?}
        COMPLIANCE -->|Yes| PATHWAY[Pathway Data]
        COMPLIANCE -->|No| ERROR[Validation Error]
    end
    
    subgraph "Initiative Scoring"
        PATHWAY --> INIT[Initiative Scorer]
        INIT --> TEMPLATES[Get Templates]
        TEMPLATES --> SCORE[Score Each Initiative]
        SCORE --> IMPACT[Impact: tCO2e/year]
        SCORE --> COST[Cost: $]
        SCORE --> FEAS[Feasibility: 0-1]
        
        IMPACT --> MCA[Multi-Criteria Analysis]
        COST --> MCA
        FEAS --> MCA
        
        MCA --> RANK[Rank Initiatives]
        RANK --> TOP[Top 10 Recommendations]
    end
    
    subgraph "Output"
        PATHWAY --> PLAN[Net Zero Plan]
        TOP --> PLAN
        PLAN --> STORE[Store Plan]
        STORE --> RESPONSE[Return Response]
    end
    
    style SBTI fill:#c8e6c9
    style VAL fill:#fff3e0
    style INIT fill:#e3f2fd
    style MCA fill:#f3e5f5
```

---

## Tool Router Architecture

### Tool Router Decision Flow

```mermaid
graph TB
    START([Enhanced Query]) --> TR_INPUT[Tool Router Input]
    TR_INPUT --> QUERY[Query Text]
    TR_INPUT --> CONTEXT[Document Context]
    TR_INPUT --> MEMORY[Conversation Memory]
    TR_INPUT --> FILE_ANALYSIS[File Analysis Tags]
    
    QUERY --> TR_LLM[Tool Router LLM<br/>Gemini 2.5 Flash]
    CONTEXT --> TR_LLM
    MEMORY --> TR_LLM
    FILE_ANALYSIS --> TR_LLM
    
    TR_LLM --> SCHEMA_VALID[JSON Schema Validator]
    SCHEMA_VALID --> PARSE[Parse Tool Calls]
    
    PARSE --> ROUTE_DECISION{Route Decision}
    
    ROUTE_DECISION -->|Carbon Calculation| CARBON_TOOL[carbon_accounting_agent]
    ROUTE_DECISION -->|Benchmarking| BENCH_TOOL[benchmarking_agent]
    ROUTE_DECISION -->|Net Zero| NETZERO_TOOL[net_zero_planner]
    ROUTE_DECISION -->|Chart| CHART_TOOL[chart_generator]
    ROUTE_DECISION -->|Emission Factor| EF_TOOL[emission_factor_resolver]
    ROUTE_DECISION -->|Other| OTHER_TOOL[Other Tools]
    ROUTE_DECISION -->|No Tool| DIRECT_RAG[Direct RAG Response]
    
    CARBON_TOOL --> EXEC_CARBON[Execute Carbon Agent]
    BENCH_TOOL --> EXEC_BENCH[Execute Benchmarking Agent]
    NETZERO_TOOL --> EXEC_NETZERO[Execute Net Zero Planner]
    CHART_TOOL --> EXEC_CHART[Execute Chart Generator]
    EF_TOOL --> EXEC_EF[Execute Emission Factor Resolver]
    OTHER_TOOL --> EXEC_OTHER[Execute Other Tool]
    
    EXEC_CARBON --> PARALLEL[Parallel Execution]
    EXEC_BENCH --> PARALLEL
    EXEC_NETZERO --> PARALLEL
    EXEC_CHART --> PARALLEL
    EXEC_EF --> PARALLEL
    EXEC_OTHER --> PARALLEL
    
    PARALLEL --> GATHER[Gather Results]
    GATHER --> COMBINE[Combine Tool Outputs]
    DIRECT_RAG --> COMBINE
    
    COMBINE --> RESPONSE([Return Tool Results])
    
    style TR_LLM fill:#fff3e0
    style SCHEMA_VALID fill:#e3f2fd
    style PARALLEL fill:#c8e6c9
```

### Tool Router Decision Logic

```mermaid
graph TB
    QUERY[User Query] --> KEYWORDS[Extract Keywords]
    
    KEYWORDS --> CHECK_CARBON{Carbon Keywords?}
    CHECK_CARBON -->|calculate carbon<br/>carbon footprint<br/>emissions| CARBON_AGENT
    
    KEYWORDS --> CHECK_BENCH{Benchmark Keywords?}
    CHECK_BENCH -->|benchmark<br/>compare<br/>peer comparison| BENCH_AGENT
    
    KEYWORDS --> CHECK_NETZERO{Net Zero Keywords?}
    CHECK_NETZERO -->|net zero<br/>SBTi<br/>pathway| NETZERO_AGENT
    
    KEYWORDS --> CHECK_CHART{Chart Keywords?}
    CHECK_CHART -->|chart<br/>graph<br/>visualize| CHART_AGENT
    
    KEYWORDS --> CHECK_EF{Emission Factor Keywords?}
    CHECK_EF -->|emission factor<br/>factor value| EF_RESOLVER
    
    KEYWORDS --> CHECK_FILE{File Analysis?}
    CHECK_FILE -->|FILE_ANALYSIS tags| FILE_ROUTE[File-Aware Routing]
    
    FILE_ROUTE --> FILE_CARBON{File has<br/>Carbon Data?}
    FILE_ROUTE --> FILE_BENCH{File has<br/>Multiple Companies?}
    FILE_ROUTE --> FILE_NETZERO{File has<br/>Historical Emissions?}
    
    FILE_CARBON -->|Yes| CARBON_AGENT
    FILE_BENCH -->|Yes| BENCH_AGENT
    FILE_NETZERO -->|Yes| NETZERO_AGENT
    
    CARBON_AGENT --> ROUTE_CARBON[Route to Carbon Agent]
    BENCH_AGENT --> ROUTE_BENCH[Route to Benchmarking Agent]
    NETZERO_AGENT --> ROUTE_NETZERO[Route to Net Zero Planner]
    CHART_AGENT --> ROUTE_CHART[Route to Chart Generator]
    EF_RESOLVER --> ROUTE_EF[Route to Emission Factor Resolver]
    
    ROUTE_CARBON --> EXECUTE[Execute Tool]
    ROUTE_BENCH --> EXECUTE
    ROUTE_NETZERO --> EXECUTE
    ROUTE_CHART --> EXECUTE
    ROUTE_EF --> EXECUTE
    
    CHECK_CARBON -->|No| CHECK_BENCH
    CHECK_BENCH -->|No| CHECK_NETZERO
    CHECK_NETZERO -->|No| CHECK_CHART
    CHECK_CHART -->|No| CHECK_EF
    CHECK_EF -->|No| CHECK_FILE
    CHECK_FILE -->|No| DIRECT_RAG[Direct RAG Response]
    
    style CARBON_AGENT fill:#e3f2fd
    style BENCH_AGENT fill:#f3e5f5
    style NETZERO_AGENT fill:#e8f5e9
    style FILE_ROUTE fill:#fff3e0
```

---

## File Context Handling

### File Processing and Context Flow

```mermaid
graph TB
    START([File Upload]) --> UPLOAD[File Upload Handler]
    UPLOAD --> DETECT[Detect File Type]
    
    DETECT --> CSV[CSV File]
    DETECT --> PDF[PDF File]
    DETECT --> EXCEL[Excel File]
    DETECT --> OTHER[Other Formats]
    
    CSV --> CSV_PARSER[CSV Parser]
    PDF --> PDF_PARSER[PDF Parser + OCR]
    EXCEL --> EXCEL_PARSER[Excel Parser]
    OTHER --> TEXT_PARSER[Text Parser]
    
    CSV_PARSER --> EXTRACT[Extract Data]
    PDF_PARSER --> EXTRACT
    EXCEL_PARSER --> EXTRACT
    TEXT_PARSER --> EXTRACT
    
    EXTRACT --> ANALYZE[File Analysis]
    ANALYZE --> COMPANIES[Extract Companies]
    ANALYZE --> METRICS[Extract Metrics]
    ANALYZE --> SECTORS[Extract Sectors]
    ANALYZE --> REGIONS[Extract Regions]
    ANALYZE --> VALUES[Extract Values]
    
    COMPANIES --> FILE_ANALYSIS[File Analysis Object]
    METRICS --> FILE_ANALYSIS
    SECTORS --> FILE_ANALYSIS
    REGIONS --> FILE_ANALYSIS
    VALUES --> FILE_ANALYSIS
    
    FILE_ANALYSIS --> TAG[Wrap in FILE_ANALYSIS Tags]
    TAG --> STORE_CONTEXT[Store in Context]
    
    STORE_CONTEXT --> QUERY[User Query]
    QUERY --> FILE_HANDLER[File Context Handler]
    
    FILE_HANDLER --> CHECK{File Context Present?}
    CHECK -->|Yes| EXTRACT_ANALYSIS[Extract File Analysis]
    CHECK -->|No| STANDARD[Standard Processing]
    
    EXTRACT_ANALYSIS --> ROUTE{Agent Type?}
    
    ROUTE -->|Carbon| CARBON_EXTRACT[Extract Carbon Data]
    ROUTE -->|Benchmarking| BENCH_EXTRACT[Extract Benchmarking Data]
    ROUTE -->|Net Zero| NETZERO_EXTRACT[Extract Net Zero Data]
    
    CARBON_EXTRACT --> ENHANCE_CARBON[Enhance Extraction]
    BENCH_EXTRACT --> ENHANCE_BENCH[Enhance Extraction]
    NETZERO_EXTRACT --> ENHANCE_NETZERO[Enhance Extraction]
    
    ENHANCE_CARBON --> AGENT_CARBON[Carbon Agent]
    ENHANCE_BENCH --> AGENT_BENCH[Benchmarking Agent]
    ENHANCE_NETZERO --> AGENT_NETZERO[Net Zero Planner]
    
    STANDARD --> AGENT_CARBON
    STANDARD --> AGENT_BENCH
    STANDARD --> AGENT_NETZERO
    
    style FILE_ANALYSIS fill:#fff3e0
    style FILE_HANDLER fill:#e1bee7
    style EXTRACT fill:#e3f2fd
```

---

## Data Flow Diagrams

### Complete Data Flow Architecture

```mermaid
graph TB
    subgraph "User Input"
        USER[User]
        QUERY[Query]
        FILES[Files]
    end
    
    subgraph "Processing Layer"
        L1[Layer 1]
        L2[Layer 2]
        TR[Tool Router]
    end
    
    subgraph "Agent Layer"
        CA[Carbon Agent]
        BA[Benchmarking Agent]
        NZ[Net Zero Planner]
    end
    
    subgraph "Calculation Engines"
        CCE[Carbon Engine]
        BCE[Benchmarking Engine]
        SBTI[SBTi Calculator]
    end
    
    subgraph "Data Sources"
        SQL[(SQLite<br/>Emission Factors)]
        MDB[(MongoDB<br/>Results Storage)]
        VS[(Vector Store<br/>Documents)]
        WEB[Web Search]
    end
    
    subgraph "Output"
        RESP[Response]
        CHARTS[Charts]
        PROOF[Proof]
    end
    
    USER --> QUERY
    USER --> FILES
    
    QUERY --> L1
    FILES --> L1
    L1 --> L2
    L2 --> TR
    
    TR --> CA
    TR --> BA
    TR --> NZ
    
    CA --> CCE
    BA --> BCE
    NZ --> SBTI
    
    CCE --> SQL
    CCE --> WEB
    BCE --> SQL
    BCE --> WEB
    SBTI --> SQL
    
    CA --> MDB
    BA --> MDB
    NZ --> MDB
    
    L2 --> VS
    
    CCE --> RESP
    BCE --> RESP
    SBTI --> RESP
    
    CA --> CHARTS
    BA --> CHARTS
    NZ --> CHARTS
    
    CCE --> PROOF
    BCE --> PROOF
    SBTI --> PROOF
    
    RESP --> USER
    CHARTS --> USER
    PROOF --> USER
    
    style CA fill:#e3f2fd
    style BA fill:#f3e5f5
    style NZ fill:#e8f5e9
    style TR fill:#fff3e0
```

---

## Component Interactions

### Agent Interaction Sequence

```mermaid
sequenceDiagram
    participant User
    participant API
    participant Layer1
    participant Layer2
    participant ToolRouter
    participant CarbonAgent
    participant BenchAgent
    participant NetZeroAgent
    participant CalcEngine
    participant Database
    participant LLM
    
    User->>API: Submit Query
    API->>Layer1: Classify Query
    Layer1->>LLM: Analyze Query Type
    LLM-->>Layer1: Classification Result
    
    alt Enhanced Query
        Layer1->>Layer2: Enhanced Query + Context
        Layer2->>ToolRouter: Route Query
        
        ToolRouter->>LLM: Determine Tool
        LLM-->>ToolRouter: Tool Selection
        
        alt Carbon Accounting
            ToolRouter->>CarbonAgent: Execute Agent
            CarbonAgent->>LLM: Extract Data
            LLM-->>CarbonAgent: Structured Data
            CarbonAgent->>CalcEngine: Calculate Emissions
            CalcEngine->>Database: Query Emission Factors
            Database-->>CalcEngine: Factors
            CalcEngine-->>CarbonAgent: Results
            CarbonAgent->>Database: Store Results
            CarbonAgent->>LLM: Generate Explanation
            LLM-->>CarbonAgent: Response
            CarbonAgent-->>ToolRouter: Agent Response
        else Benchmarking
            ToolRouter->>BenchAgent: Execute Agent
            BenchAgent->>LLM: Extract Data
            LLM-->>BenchAgent: Structured Data
            BenchAgent->>CalcEngine: Calculate Benchmarks
            CalcEngine->>Database: Query Benchmarks
            Database-->>CalcEngine: Benchmark Data
            CalcEngine-->>BenchAgent: Results
            BenchAgent->>Database: Store Results
            BenchAgent->>LLM: Generate Explanation
            LLM-->>BenchAgent: Response
            BenchAgent-->>ToolRouter: Agent Response
        else Net Zero Planning
            ToolRouter->>NetZeroAgent: Execute Agent
            NetZeroAgent->>LLM: Extract Data
            LLM-->>NetZeroAgent: Structured Data
            NetZeroAgent->>CalcEngine: Calculate Pathway
            CalcEngine->>Database: Query SBTi Data
            Database-->>CalcEngine: Pathway Data
            CalcEngine-->>NetZeroAgent: Pathway
            NetZeroAgent->>CalcEngine: Score Initiatives
            CalcEngine-->>NetZeroAgent: Recommendations
            NetZeroAgent->>Database: Store Plan
            NetZeroAgent->>LLM: Generate Explanation
            LLM-->>NetZeroAgent: Response
            NetZeroAgent-->>ToolRouter: Agent Response
        end
        
        ToolRouter-->>Layer2: Tool Results
        Layer2->>LLM: Generate Final Response
        LLM-->>Layer2: Combined Response
        Layer2-->>API: Response
        API-->>User: Stream Response
    else Direct Response
        Layer1-->>API: Direct Response
        API-->>User: Stream Response
    end
```

---

## Summary

This technical documentation provides a comprehensive overview of the FitSol ESG Co-Pilot agentic architecture, including:

1. **System Architecture**: High-level view of all components and their interactions
2. **Agentic Architecture**: Complete flow from user query to agent execution
3. **Query Processing Flow**: Detailed sequence of query handling
4. **Agent Flows**: Individual flows for each agent (Carbon, Benchmarking, Net Zero)
5. **Tool Router**: Decision-making logic for routing queries to appropriate agents
6. **File Context Handling**: How uploaded files are processed and integrated
7. **Data Flow**: Complete data flow through the system
8. **Component Interactions**: Sequence diagrams showing component communication

Each agent follows a consistent pattern:
- **Data Extraction**: Using LLM and regex patterns
- **Clarification**: Intelligent clarification orchestrators
- **Calculation**: Dedicated calculation engines
- **Storage**: MongoDB for results
- **Explanation**: LLM-generated natural language responses
- **Visualization**: Automatic chart generation

The system uses a two-layer AI architecture:
- **Layer 1**: Query classification and simple responses
- **Layer 2**: RAG processing with tool routing and agent execution

All agents support file context awareness, allowing them to extract data from uploaded files and enhance their calculations accordingly.

