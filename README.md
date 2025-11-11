# FitSol ESG Co-Pilot - Complete Technical Documentation

## Table of Contents

1. [System Architecture](#system-architecture)
2. [Application Flow](#application-flow)
3. [Agent Workflows](#agent-workflows)
   - [Carbon Accounting Agent](#carbon-accounting-agent)
   - [Benchmarking Agent](#benchmarking-agent)
   - [Net-Zero Planner Agent](#net-zero-planner-agent)
4. [Hybrid AI Architecture](#hybrid-ai-architecture)
5. [Data Flow & Storage](#data-flow--storage)
6. [Tool Router System](#tool-router-system)
7. [Calculation Engines](#calculation-engines)
8. [Visualization System](#visualization-system)
9. [Workflow Logging](#workflow-logging)

---

## System Architecture

### High-Level Architecture

```mermaid
graph TB
    subgraph "Frontend Layer"
        UI[React UI]
        Chat[Chat Interface]
        Dashboard[Admin Dashboard]
        Viz[Visualization Components]
    end
    
    subgraph "API Layer"
        FastAPI[FastAPI Backend]
        Routes[API Routes]
        SSE[Server-Sent Events]
    end
    
    subgraph "AI Processing Layer"
        Layer1[Layer 1: Query Classification]
        Layer2[Layer 2: RAG + Tools]
        Router[Tool Router]
        HybridRetrieval[Hybrid Retrieval System]
    end
    
    subgraph "Agent Layer"
        Carbon[Carbon Accounting Agent]
        Bench[Benchmarking Agent]
        NetZero[Net-Zero Planner Agent]
        Chart[Chart Generator]
    end
    
    subgraph "Calculation Layer"
        CarbonCalc[Carbon Calculation Engine]
        BenchCalc[Benchmarking Engine]
        NetZeroCalc[SBTi Pathway Calculator]
        InitiativeScorer[Initiative Scorer]
    end
    
    subgraph "Data Storage Layer"
        MongoDB[(MongoDB)]
        FAISS[(FAISS Vector Store)]
        SQLite[(SQLite Databases)]
    end
    
    UI --> FastAPI
    Chat --> FastAPI
    Dashboard --> FastAPI
    Viz --> FastAPI
    
    FastAPI --> Routes
    Routes --> Layer1
    Layer1 --> Layer2
    Layer2 --> HybridRetrieval
    Layer2 --> Router
    
    Router --> Carbon
    Router --> Bench
    Router --> NetZero
    Router --> Chart
    
    Carbon --> CarbonCalc
    Bench --> BenchCalc
    NetZero --> NetZeroCalc
    NetZero --> InitiativeScorer
    
    CarbonCalc --> SQLite
    BenchCalc --> SQLite
    NetZeroCalc --> MongoDB
    
    HybridRetrieval --> FAISS
    HybridRetrieval --> SQLite
    HybridRetrieval --> MongoDB
    Layer2 --> MongoDB
    Carbon --> MongoDB
    Bench --> MongoDB
    NetZero --> MongoDB
    
    FastAPI --> SSE
    SSE --> UI
```

### Component Interaction

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant API
    participant Layer1
    participant Layer2
    participant HybridRetrieval
    participant Router
    participant Agent
    participant CalcEngine
    participant DB
    
    User->>Frontend: Submit Query
    Frontend->>API: POST /chat/chat-with-llm
    API->>Layer1: Classify Query
    
    alt Clarification Needed
        Layer1->>API: Clarification Required
        API->>Frontend: Show Clarification UI
        Frontend->>User: Display Questions
        User->>Frontend: Provide Details
        Frontend->>API: Enhanced Query
    end
    
    API->>Layer2: Process Enhanced Query
    Layer2->>HybridRetrieval: Retrieve Context
    HybridRetrieval->>DB: Query SQL/Vector/MongoDB
    DB->>HybridRetrieval: Return Context
    HybridRetrieval->>Layer2: Combined Context
    
    Layer2->>Router: Check Tool Requirements
    Router->>Agent: Execute Agent
    
    Agent->>CalcEngine: Perform Calculations
    CalcEngine->>DB: Lookup Data
    DB->>CalcEngine: Return Data
    CalcEngine->>Agent: Calculation Results
    
    Agent->>DB: Store Results
    Agent->>Router: Tool Output
    Router->>Layer2: Tool Results
    Layer2->>Layer2: Generate Response
    Layer2->>API: Stream Response
    API->>Frontend: SSE Stream
    Frontend->>User: Display Response
```

---

## Application Flow

### Complete Query Processing Flow

```mermaid
graph TB
    Start([User Submits Query]) --> FewShot{Few-Shot Match?}
    FewShot -->|Yes| Instant[Instant Response]
    FewShot -->|No| Layer1[Layer 1: Classification]
    
    Layer1 --> Check1{Query Type?}
    Check1 -->|Greeting| Greeting[Direct Response]
    Check1 -->|Memory| Memory[Retrieve from History]
    Check1 -->|Clarification| Clarify[Show Clarification UI]
    Check1 -->|Enhanced| Layer2[Layer 2: RAG Processing]
    
    Clarify --> UserInput[User Provides Details]
    UserInput --> Layer2
    
    Layer2 --> HybridRetrieval[Hybrid Retrieval]
    HybridRetrieval --> SQLDB[(SQL Database)]
    HybridRetrieval --> VectorDB[(Vector Database)]
    HybridRetrieval --> Internet[Internet Search]
    
    SQLDB --> Context[Combined Context]
    VectorDB --> Context
    Internet --> Context
    
    Context --> ToolRouter{Tool Needed?}
    ToolRouter -->|No| DirectRAG[Direct RAG Response]
    ToolRouter -->|Yes| SelectTool{Which Tool?}
    
    SelectTool -->|Carbon| CarbonAgent[Carbon Accounting Agent]
    SelectTool -->|Benchmark| BenchAgent[Benchmarking Agent]
    SelectTool -->|Net-Zero| NetZeroAgent[Net-Zero Planner Agent]
    SelectTool -->|Chart| ChartGen[Chart Generator]
    
    CarbonAgent --> CarbonCalc[Calculation Engine]
    BenchAgent --> BenchCalc[Benchmarking Engine]
    NetZeroAgent --> NetZeroCalc[SBTi Calculator]
    
    CarbonCalc --> CarbonResult[Results]
    BenchCalc --> BenchResult[Results]
    NetZeroCalc --> NetZeroResult[Results]
    
    CarbonResult --> Viz[Visualization]
    BenchResult --> Viz
    NetZeroResult --> Viz
    
    DirectRAG --> Combine[Combine All Contexts]
    CarbonResult --> Combine
    BenchResult --> Combine
    NetZeroResult --> Combine
    ChartGen --> Combine
    Viz --> Combine
    
    Combine --> Layer2Gen[Layer 2: Generate Response]
    Layer2Gen --> Citations[Add Citations]
    Citations --> Stream[Stream to Frontend]
    
    Greeting --> Stream
    Memory --> Stream
    Instant --> Stream
    
    Stream --> Display[Display to User]
    Display --> End([Complete])
```

---

## Agent Workflows

### Carbon Accounting Agent

#### Complete Workflow

```mermaid
graph TD
    Start([Query Received]) --> Log1[Log: QUERY_RECEIVED]
    Log1 --> Extract[Data Extraction]
    
    Extract --> Regex[Regex Extraction]
    Regex --> LLM{Regex Success?}
    LLM -->|Yes| Extracted[Extracted Data]
    LLM -->|No| LLMExtract[LLM Extraction]
    LLMExtract --> Extracted
    
    Extracted --> Log2[Log: CARBON_DATA_EXTRACTION]
    Log2 --> ClarifyCheck{Clarification Needed?}
    
    ClarifyCheck -->|Yes| GatherSuggestions[Gather Suggestions]
    GatherSuggestions --> DBLookup[Database Lookup]
    DBLookup --> KBSearch[Knowledge Base Search]
    KBSearch --> InternetSearch[Internet Search]
    
    InternetSearch --> PreCalc[Pre-Calculation Disclosure]
    PreCalc --> Log3[Log: CARBON_CLARIFICATION_GENERATED]
    Log3 --> ReturnClarify[Return Clarification]
    
    ClarifyCheck -->|No| Normalize[Normalize Activity Types]
    Normalize --> EmissionFactor[Emission Factor Lookup]
    EmissionFactor --> Log4[Log: CARBON_EMISSION_FACTOR_LOOKUP]
    
    Log4 --> CalcStart[Start Calculation]
    CalcStart --> Log5[Log: CARBON_CALCULATION_START]
    
    Log5 --> ScopeCheck{Scope Type?}
    ScopeCheck -->|Scope 1| Scope1[Calculate Scope 1]
    ScopeCheck -->|Scope 2| Scope2[Calculate Scope 2]
    ScopeCheck -->|Scope 3| Scope3[Calculate Scope 3]
    ScopeCheck -->|Total| Total[Calculate Total]
    
    Scope1 --> Log6[Log: CARBON_CALCULATION_SCOPE1]
    Scope2 --> Log7[Log: CARBON_CALCULATION_SCOPE2]
    Scope3 --> Log8[Log: CARBON_CALCULATION_SCOPE3]
    Total --> Log9[Log: CARBON_CALCULATION_TOTAL]
    
    Log6 --> Store[Store Results]
    Log7 --> Store
    Log8 --> Store
    Log9 --> Store
    
    Store --> Log10[Log: CARBON_RESULT_STORED]
    Log10 --> GenerateProof[Generate Calculation Proof]
    GenerateProof --> Viz[Generate Visualizations]
    Viz --> Log11[Log: CARBON_VISUALIZATION_GENERATED]
    
    Log11 --> GenerateResponse[Generate Explanation]
    GenerateResponse --> Log12[Log: CARBON_RESPONSE_GENERATED]
    Log12 --> Log13[Log: WORKFLOW_COMPLETE]
    Log13 --> End([Return Response])
    
    ReturnClarify --> End
```

#### Detailed Step Descriptions

**1. Query Reception & Logging**
- User query received
- Workflow session started
- Query logged with metadata (context, memory presence)

**2. Data Extraction**
- **Regex Extraction**: Pattern matching for common formats
  - Activity types: "diesel", "electricity", "waste", etc.
  - Quantities: "1000 liters", "5000 kWh", "500 kg"
  - Locations: "India", "USA", "Brazil"
- **LLM Extraction**: Fallback for complex queries
  - Uses structured schema extraction
  - Handles ambiguous inputs

**3. Clarification Check**
- Analyzes extracted data completeness
- Checks for required fields:
  - Region (country) - **REQUIRED**
  - At least one activity with amount - **REQUIRED**
  - Company name, time period - **OPTIONAL**

**4. Pre-Calculation Data Selection (Cross-Question)**
- **Database Lookup**: Searches SQLite for emission factors
- **Knowledge Base Search**: Searches vector store for relevant data
- **Internet Search**: Searches web for missing emission factors
- **Disclosure Generation**: Shows all available options to user
  - Database options with sources (IPCC, IEA, EPA)
  - Internet options with confidence scores
  - Allows user to select or proceed automatically

**5. Emission Factor Lookup**
- Normalizes activity types (e.g., "lpg_consumption" → "lpg_combustion")
- Retrieves emission factors from database
- Applies region-specific factors
- Handles temporal grid factors for electricity

**6. Calculation Execution**
- **Scope 1**: Direct emissions from fuel combustion
  - Formula: `Emissions = Activity × Emission Factor × GWP`
  - Handles multiple fuel types
  - Applies IPCC AR6 GWP values
- **Scope 2**: Indirect emissions from purchased electricity
  - Location-based: Uses grid emission factors
  - Market-based: Uses contractual instruments (if available)
- **Scope 3**: Value chain emissions
  - Activity-based, spend-based, supplier-specific methods
  - Category-specific calculations
- **Total**: Sums all scopes

**7. Result Storage**
- Stores calculation results in MongoDB
- Includes metadata: timestamp, query, extracted data, results
- Enables trend analysis and historical tracking

**8. Visualization Generation**
- **Pie Chart**: Scope breakdown (Scope 1, 2, 3)
- **Bar Chart**: Emissions by activity type
- **Timeline Chart**: Historical trends (if available)
- Charts embedded as base64 images in response

**9. Response Generation**
- **Calculation Proof**: Step-by-step formulas and intermediate values
- **Action Points**: What was accomplished
- **Data Sources**: Verified sources used
- **Workflow Timeline**: Visual representation of steps taken
- **Recommendations**: Actionable insights

#### Example Calculation Flow

```
Query: "Calculate carbon footprint for 1000 liters diesel in India"

Step 1: Extract Data
  - Activity: diesel_combustion
  - Amount: 1000 liters
  - Region: India

Step 2: Lookup Emission Factor
  - Database: IPCC 2006 Guidelines
  - Factor: 2.68 kg CO₂ equivalent/liter
  - Source: Verified

Step 3: Calculate
  - Emissions = 1000 L × 2.68 kg CO₂ equivalent/L
  - Emissions = 2,680 kg CO₂ equivalent
  - Emissions = 2.68 tonnes CO₂ equivalent

Step 4: Generate Proof
  - Formula: E = A × EF × GWP
  - A = 1000 L
  - EF = 2.68 kg CO₂ equivalent/L
  - GWP = 1 (CO₂)
  - E = 2,680 kg CO₂ equivalent

Step 5: Store & Visualize
  - Store in MongoDB
  - Generate pie chart (Scope 1: 2.68 tonnes)
  - Generate bar chart (Diesel: 2.68 tonnes)
```

---

### Benchmarking Agent

#### Complete Workflow

```mermaid
graph TD
    Start([Query Received]) --> Log1[Log: QUERY_RECEIVED]
    Log1 --> FileCheck{File Context Available?}
    
    FileCheck -->|Yes| FileOrch[File-Based Orchestrator]
    FileOrch --> Log2[Log: BENCH_FILE_ORCHESTRATOR]
    Log2 --> ExtractFromFiles[Extract from Files]
    
    FileCheck -->|No| InitialExtract[Initial Data Extraction]
    InitialExtract --> Normalize[Normalize Data]
    Normalize --> Log3[Log: BENCH_DATA_EXTRACTION]
    
    ExtractFromFiles --> Validate[Validate Extraction]
    Log3 --> Validate
    
    Validate --> ClarifyCheck{Clarification Needed?}
    
    ClarifyCheck -->|Yes| GatherSuggestions[Gather Suggestions]
    GatherSuggestions --> DBLookup[Database Lookup]
    DBLookup --> KBSearch[Knowledge Base Search]
    KBSearch --> InternetSearch[Internet Search]
    
    InternetSearch --> PreCalc[Pre-Calculation Disclosure]
    PreCalc --> Log4[Log: BENCH_CLARIFICATION_GENERATED]
    Log4 --> ReturnClarify[Return Clarification]
    
    ClarifyCheck -->|No| NormalizeMetric[Normalize Metric Name]
    NormalizeMetric --> NormalizeSector[Normalize Sector]
    
    NormalizeSector --> PeerGroup[Identify Peer Group]
    PeerGroup --> Log5[Log: BENCH_PEER_GROUP_IDENTIFICATION]
    
    Log5 --> BenchmarkLookup[Lookup Benchmark Data]
    BenchmarkLookup --> Log6[Log: BENCH_BENCHMARK_LOOKUP]
    
    Log6 --> BenchmarkType{Benchmark Type?}
    BenchmarkType -->|Single Metric| Percentile[Calculate Percentile Rank]
    BenchmarkType -->|Gap Analysis| GapAnalysis[Calculate Gap Analysis]
    BenchmarkType -->|Peer Comparison| PeerComp[Peer Comparison]
    
    Percentile --> Log7[Log: BENCH_PERCENTILE_CALCULATION]
    GapAnalysis --> Log8[Log: BENCH_GAP_ANALYSIS]
    PeerComp --> Log9[Log: BENCH_PEER_COMPARISON]
    
    Log7 --> Store[Store Results]
    Log8 --> Store
    Log9 --> Store
    
    Store --> Log10[Log: BENCH_RESULT_STORED]
    Log10 --> GenerateProof[Generate Calculation Proof]
    GenerateProof --> Viz[Generate Visualizations]
    Viz --> Log11[Log: BENCH_VISUALIZATION_GENERATED]
    
    Log11 --> GenerateResponse[Generate Explanation]
    GenerateResponse --> Log12[Log: BENCH_RESPONSE_GENERATED]
    Log12 --> Log13[Log: WORKFLOW_COMPLETE]
    Log13 --> End([Return Response])
    
    ReturnClarify --> End
```

#### Detailed Step Descriptions

**1. Query Reception & File Context Check**
- Checks if file-based context is available
- If yes, uses File-Based Orchestrator to extract data from uploaded files
- If no, proceeds with standard extraction

**2. Data Extraction & Normalization**
- **Initial Extraction**: LLM extracts structured data
- **Normalization**:
  - Metric names: "carbon emissions" → "carbon_emissions"
  - Sectors: "Manufacturing" → "Manufacturing" (title case)
  - Company names: Title case or uppercase for acronyms
- **Validation**: Ensures required fields are present
  - Company name - **REQUIRED**
  - Metric name - **REQUIRED**
  - Metric value - **REQUIRED**
  - Sector - **REQUIRED**

**3. Clarification Check**
- Analyzes extracted data completeness
- If missing required fields, generates clarification questions
- Includes suggestions from database and knowledge base

**4. Pre-Calculation Data Selection**
- **Database Lookup**: Searches SQLite for benchmark data
- **Knowledge Base Search**: Searches vector store
- **Internet Search**: Searches web for company metric values
- **Disclosure Generation**: Shows available benchmark options

**5. Peer Group Identification**
- Identifies peer group based on:
  - Sector (primary)
  - Region (optional)
  - Geography (optional)
  - Company size (if available)

**6. Benchmark Lookup**
- Searches benchmark database for:
  - Metric name
  - Sector
  - Region/Geography (optional)
  - Year (defaults to current if not specified)
- **Fallback Logic**:
  - Tries without region/geography
  - Tries with different year
  - Tries similar metrics
  - Uses closest available benchmark

**7. Calculation Execution**
- **Percentile Rank Calculation**:
  - Uses linear interpolation between percentiles
  - Handles extrapolation for values outside range
  - Formula: `Percentile = P_lower + (P_upper - P_lower) × (value - V_lower) / (V_upper - V_lower)`
- **Gap Analysis**:
  - Calculates gap to target percentile
  - Provides reduction recommendations
- **Peer Comparison**:
  - Compares with peer group companies
  - Calculates relative performance

**8. Result Storage & Visualization**
- Stores results in MongoDB
- Generates visualizations:
  - **Bar Chart**: Company vs. peers
  - **Radar Chart**: Percentile distribution
  - **Gap Chart**: Current vs. target

**9. Response Generation**
- **Percentile Ranking**: Shows where company ranks
- **Gap Analysis**: Shows distance to targets
- **Peer Comparison**: Shows relative performance
- **Recommendations**: Actionable insights

#### Example Benchmarking Flow

```
Query: "Benchmark TATA's carbon emissions of 50,000 tons CO₂ equivalent in Manufacturing sector"

Step 1: Extract & Normalize
  - Company: TATA
  - Metric: carbon_emissions (normalized from "carbon emissions")
  - Value: 50,000 tonnes CO₂ equivalent
  - Sector: Manufacturing (normalized)

Step 2: Lookup Benchmark
  - Database: Manufacturing sector, carbon_emissions
  - Percentiles:
    - P25: 30,000 tonnes
    - P50: 45,000 tonnes
    - P75: 60,000 tonnes
    - P90: 80,000 tonnes

Step 3: Calculate Percentile Rank
  - Value: 50,000 tonnes
  - Between P50 (45,000) and P75 (60,000)
  - Interpolation: P50 + (P75 - P50) × (50,000 - 45,000) / (60,000 - 45,000)
  - Percentile Rank: 50 + (75 - 50) × 5,000 / 15,000 = 58.3%

Step 4: Gap Analysis
  - Target: 75th percentile (60,000 tonnes)
  - Gap: 60,000 - 50,000 = 10,000 tonnes
  - Reduction needed: 20%

Step 5: Generate Response
  - "TATA ranks at the 58.3rd percentile"
  - "To reach 75th percentile, reduce by 10,000 tonnes (20%)"
```

---

### Net-Zero Planner Agent

#### Complete Workflow

```mermaid
graph TD
    Start([Query Received]) --> Log1[Log: QUERY_RECEIVED]
    Log1 --> Extract[Data Extraction]
    Extract --> Log2[Log: NETZERO_DATA_EXTRACTION]
    
    Log2 --> ClarifyCheck{Clarification Needed?}
    ClarifyCheck -->|Yes| ReturnClarify[Return Clarification]
    
    ClarifyCheck -->|No| CheckEmissions{Emissions Available?}
    CheckEmissions -->|No| FetchEmissions[Fetch from Carbon Agent]
    FetchEmissions --> Log3[Log: NETZERO_EMISSIONS_FETCH]
    
    CheckEmissions -->|Yes| Validate[Validate Inputs]
    Log3 --> Validate
    
    Validate --> SBTiCalc[SBTi Pathway Calculation]
    SBTiCalc --> Log4[Log: NETZERO_SBTI_PATHWAY_CALCULATION]
    
    Log4 --> Scenario{Scenario Type?}
    Scenario -->|1.5°C| Scenario15[1.5°C Scenario]
    Scenario -->|Well-Below 2°C| Scenario2C[Well-Below 2°C Scenario]
    
    Scenario15 --> Log5[Log: NETZERO_15C_SCENARIO]
    Scenario2C --> Log6[Log: NETZERO_WELL_BELOW_2C_SCENARIO]
    
    Log5 --> InitiativeScoring[Initiative Scoring]
    Log6 --> InitiativeScoring
    
    InitiativeScoring --> Log7[Log: NETZERO_INITIATIVE_SCORING]
    Log7 --> ImpactScoring[Impact Scoring]
    ImpactScoring --> Log8[Log: NETZERO_IMPACT_SCORING]
    Log8 --> CostScoring[Cost Scoring]
    CostScoring --> Log9[Log: NETZERO_COST_SCORING]
    Log9 --> FeasibilityScoring[Feasibility Scoring]
    FeasibilityScoring --> Log10[Log: NETZERO_FEASIBILITY_SCORING]
    
    Log10 --> PathwayValidation[Pathway Validation]
    PathwayValidation --> Log11[Log: NETZERO_PATHWAY_VALIDATION]
    
    Log11 --> Store[Store Plan]
    Store --> Log12[Log: NETZERO_PLAN_STORED]
    
    Log12 --> GenerateResponse[Generate Explanation]
    GenerateResponse --> Log13[Log: NETZERO_RESPONSE_GENERATED]
    Log13 --> Log14[Log: WORKFLOW_COMPLETE]
    Log14 --> End([Return Response])
    
    ReturnClarify --> End
```

#### Detailed Step Descriptions

**1. Data Extraction**
- Extracts structured data:
  - Current emissions (tonnes CO₂ equivalent)
  - Target year (default: 2050)
  - Temperature scenario (1.5°C or well-below 2°C)
  - Budget constraints (optional)
  - Company information

**2. Emissions Fetching**
- If emissions not provided, attempts to fetch from Carbon Accounting Agent
- Uses conversation history to find previous calculations
- Falls back to asking user if not found

**3. SBTi Pathway Calculation**
- **1.5°C Scenario**: Aligned with limiting warming to 1.5°C
  - Requires 50% reduction by 2030
  - Net-zero by 2050
- **Well-Below 2°C Scenario**: Aligned with limiting warming to well-below 2°C
  - Requires 30% reduction by 2030
  - Net-zero by 2050
- Calculates annual reduction targets
- Generates milestone years

**4. Initiative Scoring**
- **Impact Scoring**: Reduction potential (tonnes CO₂ equivalent/year)
- **Cost Scoring**: Implementation cost (CAPEX + OPEX)
- **Feasibility Scoring**: Technical and organizational feasibility
- **Multi-Criteria Scoring**: Weighted combination of all factors
- Recommends top initiatives based on constraints

**5. Pathway Validation**
- Validates pathway compliance with SBTi standards
- Checks milestone targets are achievable
- Verifies initiative impacts sum to required reductions

**6. Plan Storage & Response**
- Stores complete plan in MongoDB
- Generates natural language explanation
- Includes:
  - Pathway summary
  - Milestone breakdown
  - Recommended initiatives
  - Implementation timeline

#### Example Net-Zero Planning Flow

```
Query: "Create net-zero plan for 100,000 tons CO₂ equivalent with recommended initiatives"

Step 1: Extract Data
  - Current emissions: 100,000 tonnes CO₂ equivalent
  - Target year: 2050 (default)
  - Scenario: 1.5°C (default)

Step 2: Calculate SBTi Pathway
  - 2030 target: 50,000 tonnes (50% reduction)
  - 2050 target: 0 tonnes (net-zero)
  - Annual reduction: 3,333 tonnes/year

Step 3: Score Initiatives
  - Renewable Energy: Impact=20,000, Cost=Medium, Feasibility=High
  - Energy Efficiency: Impact=15,000, Cost=Low, Feasibility=High
  - Supply Chain: Impact=10,000, Cost=High, Feasibility=Medium
  - Carbon Offsets: Impact=5,000, Cost=Low, Feasibility=High

Step 4: Recommend Initiatives
  - Top 3: Renewable Energy, Energy Efficiency, Carbon Offsets
  - Total impact: 40,000 tonnes/year
  - Exceeds required annual reduction

Step 5: Generate Plan
  - Pathway: 2024-2030: 50% reduction
  - Initiatives: 3 recommended initiatives
  - Timeline: Phased implementation
```

---

## Hybrid AI Architecture

### LLM vs. Python Functions

The application uses a **hybrid approach** combining LLM intelligence with Python-based calculations:

#### LLM Responsibilities

1. **Query Understanding**
   - Natural language processing
   - Intent classification
   - Context extraction

2. **Data Extraction**
   - Structured data extraction from queries
   - Handles ambiguous inputs
   - Regex fallback for reliability

3. **Clarification Generation**
   - Natural language questions
   - Context-aware suggestions
   - User-friendly explanations

4. **Response Formatting**
   - Natural language explanations
   - Professional formatting
   - Citation integration

#### Python Function Responsibilities

1. **Mathematical Calculations**
   - GHG Protocol formulas
   - Statistical calculations (percentiles, gaps)
   - SBTi pathway modeling

2. **Database Operations**
   - SQLite queries for emission factors
   - MongoDB storage and retrieval
   - Vector database searches

3. **Data Validation**
   - Input validation
   - Range checking
   - Type validation

4. **Visualization Generation**
   - Chart creation (matplotlib, plotly)
   - Base64 encoding
   - Chart metadata

### Hybrid Flow Example

```mermaid
graph LR
    A[User Query] --> B[LLM: Extract Data]
    B --> C[Python: Validate]
    C --> D[Python: Lookup Database]
    D --> E[Python: Calculate]
    E --> F[Python: Store Results]
    F --> G[LLM: Format Response]
    G --> H[User Response]
```

---

## Data Flow & Storage

### Data Storage Architecture

```mermaid
graph TB
    subgraph "SQLite Databases"
        CarbonDB[(Carbon Accounting DB)]
        BenchDB[(Benchmarking DB)]
    end
    
    subgraph "MongoDB Collections"
        Calculations[Calculations]
        Benchmarks[Benchmark Results]
        Plans[Net-Zero Plans]
        Chats[Chat History]
        Metrics[Usage Metrics]
    end
    
    subgraph "Vector Store"
        FAISS[FAISS Index]
        Embeddings[Document Embeddings]
    end
    
    CarbonAgent --> CarbonDB
    CarbonAgent --> Calculations
    BenchAgent --> BenchDB
    BenchAgent --> Benchmarks
    NetZeroAgent --> Plans
    HybridRetrieval --> FAISS
    HybridRetrieval --> CarbonDB
    HybridRetrieval --> BenchDB
```

### Data Flow Sequence

```mermaid
sequenceDiagram
    participant Agent
    participant CalcEngine
    participant SQLite
    participant MongoDB
    participant VectorStore
    
    Agent->>CalcEngine: Request Calculation
    CalcEngine->>SQLite: Query Emission Factors
    SQLite->>CalcEngine: Return Factors
    CalcEngine->>CalcEngine: Perform Calculation
    CalcEngine->>Agent: Return Results
    Agent->>MongoDB: Store Results
    Agent->>VectorStore: Search Knowledge Base
    VectorStore->>Agent: Return Context
    Agent->>Agent: Generate Response
```

---

## Tool Router System

### Router Architecture

```mermaid
graph TD
    Query[User Query] --> Router[Tool Router]
    Router --> LLM[Gemini LLM]
    LLM --> Schema[JSON Schema Validation]
    Schema --> ToolCalls[Tool Calls]
    
    ToolCalls --> Carbon{Carbon Agent?}
    ToolCalls --> Bench{Benchmark Agent?}
    ToolCalls --> NetZero{Net-Zero Agent?}
    ToolCalls --> Chart{Chart Generator?}
    
    Carbon -->|Yes| CarbonExec[Execute Carbon Agent]
    Bench -->|Yes| BenchExec[Execute Benchmark Agent]
    NetZero -->|Yes| NetZeroExec[Execute Net-Zero Agent]
    Chart -->|Yes| ChartExec[Execute Chart Generator]
    
    CarbonExec --> Results[Tool Results]
    BenchExec --> Results
    NetZeroExec --> Results
    ChartExec --> Results
    
    Results --> Layer2[Layer 2: Response Generation]
```

### Routing Rules

1. **Carbon Accounting**
   - Keywords: "calculate", "carbon footprint", "emissions", "CO2", "GHG"
   - Priority: High for calculation requests

2. **Benchmarking**
   - Keywords: "benchmark", "compare", "peer", "industry average", "percentile"
   - Priority: High for comparison requests

3. **Net-Zero Planning**
   - Keywords: "net zero", "net-zero", "SBTi", "pathway", "decarbonization"
   - Priority: High for planning requests

4. **Chart Generation**
   - Keywords: "chart", "graph", "visualize", "plot"
   - Priority: Medium (often automatic)

---

## Calculation Engines

### Carbon Calculation Engine

**Formulas Used:**

1. **Scope 1 (Direct Emissions)**
   ```
   Emissions = Activity × Emission Factor × GWP
   ```

2. **Scope 2 (Indirect - Electricity)**
   ```
   Location-based: Emissions = Electricity (kWh) × Grid EF (kg CO₂/kWh)
   Market-based: Emissions = Electricity (kWh) × Contract EF (kg CO₂/kWh)
   ```

3. **Scope 3 (Value Chain)**
   ```
   Activity-based: Emissions = Activity × EF
   Spend-based: Emissions = Spend × EF per currency unit
   ```

**GWP Values (IPCC AR6):**
- CO₂: 1
- CH₄: 29.8
- N₂O: 273

### Benchmarking Engine

**Formulas Used:**

1. **Percentile Rank (Linear Interpolation)**
   ```
   Percentile = P_lower + (P_upper - P_lower) × (value - V_lower) / (V_upper - V_lower)
   ```

2. **Gap Analysis**
   ```
   Gap = Target Percentile Value - Company Value
   Reduction % = (Gap / Company Value) × 100
   ```

3. **Performance Score**
   ```
   Score = (Percentile Rank / 100) × 100
   ```

### SBTi Pathway Calculator

**Pathway Models:**

1. **1.5°C Scenario**
   - 2030: 50% reduction from base year
   - 2050: Net-zero

2. **Well-Below 2°C Scenario**
   - 2030: 30% reduction from base year
   - 2050: Net-zero

**Annual Reduction Calculation:**
```
Annual Reduction = (Current Emissions - Target Emissions) / Years Remaining
```

---

## How to Run the Application

### Prerequisites

- **Docker** and **Docker Compose** installed
- **MongoDB** connection string (MongoDB Atlas or local MongoDB)
- **Google Gemini API Key** (required for AI features)

### Quick Start with Docker (Recommended)

**1. Clone the Repository**
```bash
git clone <repository-url>
cd fitsol_ant
```

**2. Create Environment File**
```bash
cp env.example .env
```

**3. Configure Environment Variables**

Edit `.env` file with your configuration:

```env
# MongoDB Connection
MONGODB_CONNECTION_STRING=mongodb://localhost:27017/fitsol
# OR for MongoDB Atlas:
# MONGODB_CONNECTION_STRING=mongodb+srv://username:password@cluster.mongodb.net/fitsol

MONGODB_DB_NAME=fitsol_esg

# Google Gemini API Key (Required)
GEMINI_API_KEY=your_gemini_api_key_here

# Frontend Configuration
VITE_SERVER_BASE_URL=
FRONTEND_PORT=80
```

**4. Start the Application**
```bash
docker-compose up --build
```

**5. Access the Application**
- **Frontend**: http://localhost:80
- **API Documentation**: http://localhost:80/docs
- **Backend API**: http://localhost:80/api/

**6. Stop the Application**
```bash
docker-compose down
```

### Local Development Setup

**Backend Setup:**
```bash
cd server
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
python main.py
```

**Frontend Setup:**
```bash
cd client
npm install
npm run dev
```

**Initialize Databases:**
```bash
cd server
python -c "from utils.db_init import initialize_databases; initialize_databases()"
```

### Environment Variables

**Required:**
- `MONGODB_CONNECTION_STRING`: MongoDB connection string
- `MONGODB_DB_NAME`: Database name
- `GEMINI_API_KEY`: Google Gemini API key

**Optional:**
- `ENABLE_SCHEDULED_SCRAPING`: Enable scheduled data scraping (true/false)
- `LANGSMITH_API_KEY`: For evaluation and monitoring
- `LANGCHAIN_PROJECT`: LangChain project name

---

## CI/CD Pipeline

### Overview

The application uses **GitHub Actions** for continuous integration and deployment. The pipeline automatically:

1. **Runs Tests** - Validates code quality and functionality
2. **Builds Docker Images** - Creates container images for backend and frontend
3. **Deploys to EC2** - Automatically deploys to AWS EC2 server on push to main/develop/uat branches

### Pipeline Stages

```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'primaryColor':'#ffffff', 'primaryTextColor':'#000000', 'primaryBorderColor':'#000000', 'lineColor':'#000000', 'secondaryColor':'#f0f0f0', 'tertiaryColor':'#e0e0e0'}}}%%
graph LR
    PUSH[Code Push] --> TEST[Run Tests]
    TEST -->|Pass| BUILD[Build Docker Images]
    TEST -->|Fail| STOP[Stop Pipeline]
    BUILD -->|Success| DEPLOY[Deploy to EC2]
    BUILD -->|Fail| STOP
    DEPLOY --> DONE[Deployment Complete]
    
    style TEST fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000
    style BUILD fill:#fff3e0,stroke:#f57c00,stroke-width:2px,color:#000
    style DEPLOY fill:#e8f5e9,stroke:#388e3c,stroke-width:2px,color:#000
    style STOP fill:#ffebee,stroke:#c62828,stroke-width:2px,color:#000
```

### Pipeline Configuration

**Location**: `.github/workflows/ci-cd.yml`

**Triggers:**
- Push to `main`, `develop`, or `uat` branches
- Pull requests to `main`, `develop`, or `uat` branches

**Jobs:**

1. **Test Job**
   - Runs on: `ubuntu-latest`
   - Steps:
     - Checkout code
     - Set up Python 3.12
     - Install Python dependencies
     - Run Python linter (flake8, black)
     - Set up Node.js 18
     - Install Node dependencies
     - Run TypeScript check

2. **Build Docker Images Job**
   - Runs on: `ubuntu-latest`
   - Depends on: Test job
   - Steps:
     - Build backend Docker image
     - Build frontend Docker image
     - Validate docker-compose configuration

3. **Deploy to EC2 Job**
   - Runs on: `self-hosted` (EC2 instance)
   - Depends on: Test and Build jobs
   - Steps:
     - Checkout code
     - Pull latest changes
     - Run deployment script (`deploy.sh`)

### Deployment Process

When code is pushed to `main`, `develop`, or `uat` branches:

1. **GitHub Actions triggers** the pipeline
2. **Tests run** to ensure code quality
3. **Docker images are built** and validated
4. **Deployment script runs** on EC2 server:
   - Pulls latest code
   - Rebuilds containers
   - Restarts services
   - Verifies deployment

**Deployment Script**: `deploy.sh`
- Stops existing containers
- Pulls latest code
- Rebuilds Docker images
- Starts containers
- Runs health checks

---


## Workflow Logging

### Log Structure

Each agent workflow is logged step-by-step:

**Log Files:**
- `app.log`: Main application logs
- `carbon_agent.log`: Carbon Accounting Agent workflows
- `benchmarking_agent.log`: Benchmarking Agent workflows
- `net_zero_agent.log`: Net-Zero Planner Agent workflows

**Log Format:**
```json
{
  "timestamp": "2025-01-15T10:30:45.123456+00:00",
  "session_id": "session_123",
  "step": "CARBON_CALCULATION_START",
  "agent": "carbon",
  "details": {
    "query": "Calculate carbon footprint...",
    "extracted_data": {...},
    "execution_time": 1.23
  }
}
```

**Workflow Steps Tracked:**
- Query reception
- Data extraction
- Clarification checks
- Database lookups
- Calculations
- Result storage
- Visualization generation
- Response generation
- Workflow completion

---

## Summary

This technical documentation provides a complete overview of:

1. **System Architecture**: Multi-layer AI system with specialized agents
2. **Application Flow**: End-to-end query processing
3. **Agent Workflows**: Detailed workflows for each agent
4. **Hybrid Architecture**: LLM + Python functions
5. **Data Management**: Storage and retrieval systems
6. **Tool Routing**: Intelligent agent selection
7. **Calculations**: Mathematical formulas and engines
8. **Visualization**: Automatic chart generation
9. **Logging**: Comprehensive workflow tracking

The system is designed to be:
- **Transparent**: Full calculation proofs and data sources
- **Accurate**: Real mathematical calculations, not LLM-generated numbers
- **User-Friendly**: Natural language interactions with automatic clarifications
- **Comprehensive**: Detailed responses with visualizations and recommendations

---

*Last Updated: January 2025*

