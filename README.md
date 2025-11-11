# FitSol ESG Co-Pilot - Technical Documentation

**Client-Facing Technical Documentation for Carbon Emissions Experts**

---

## 📋 Table of Contents

1. [Executive Summary](#executive-summary)
2. [What We Built](#what-we-built)
3. [System Architecture](#system-architecture)
4. [How It Works - Agentic Architecture](#how-it-works---agentic-architecture)
5. [The Three AI Agents](#the-three-ai-agents)
6. [Sample Queries & Use Cases](#sample-queries--use-cases)
7. [Sample Data Files](#sample-data-files)
8. [How to Run the Application](#how-to-run-the-application)
9. [CI/CD Pipeline](#cicd-pipeline)
10. [Technical Implementation Details](#technical-implementation-details)

---

## Executive Summary

**FitSol ESG Co-Pilot** is an AI-powered platform that helps organizations calculate, benchmark, and plan their carbon emissions reduction strategies. The system uses three specialized AI agents that work together to provide comprehensive ESG intelligence:

1. **Carbon Accounting Agent** - Calculates carbon footprints using verified emission factors
2. **Benchmarking Agent** - Compares your company's ESG metrics against industry peers
3. **Net Zero Planner** - Creates science-based pathways to achieve net-zero emissions

The platform intelligently understands natural language queries, extracts data from uploaded files (CSV, PDF, Excel), and performs real mathematical calculations (not AI-generated numbers) using industry-standard formulas and verified databases.

---

## What We Built

### Core Capabilities

**1. Intelligent Query Understanding**
- The system understands natural language questions about carbon emissions
- Example: "Calculate carbon footprint for 1000 liters of diesel in India"
- Automatically extracts: activity type, amount, region, and other parameters

**2. Real Mathematical Calculations**
- Uses **GHG Protocol Standard** formulas
- Accesses verified emission factor databases (IPCC, IEA, EPA)
- Performs actual calculations, not AI-generated estimates
- Provides step-by-step calculation proofs

**3. File-Based Intelligence**
- Upload CSV/Excel files with company ESG data
- System automatically extracts companies, metrics, sectors, regions
- Enables bulk comparisons and benchmarking across multiple companies

**4. Industry Benchmarking**
- Compares your metrics against industry peers
- Calculates percentile rankings using statistical methods
- Identifies performance gaps and improvement opportunities

**5. Science-Based Net Zero Planning**
- Creates SBTi (Science Based Targets initiative) aligned pathways
- Supports 1.5°C and well-below 2°C scenarios
- Recommends decarbonization initiatives with impact scoring

### Technology Stack

- **Frontend**: React with TypeScript
- **Backend**: FastAPI (Python)
- **AI/LLM**: Google Gemini 2.0/2.5 Flash
- **Vector Database**: ChromaDB (for document search)
- **Databases**: 
  - MongoDB (results storage)
  - SQLite (emission factors and benchmarks)
- **Deployment**: Docker containers on AWS EC2

---

## System Architecture

### High-Level Overview

```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'primaryColor':'#ffffff', 'primaryTextColor':'#000000', 'primaryBorderColor':'#000000', 'lineColor':'#000000', 'secondaryColor':'#f0f0f0', 'tertiaryColor':'#e0e0e0'}}}%%
graph TB
    subgraph "User Interface"
        UI[React Frontend<br/>User Interface]
    end
    
    subgraph "Backend Services"
        API[FastAPI Backend<br/>REST API]
    end
    
    subgraph "AI Processing"
        L1[Layer 1: Query Classification<br/>Understands user intent]
        L2[Layer 2: RAG + Tool Router<br/>Searches documents & routes to agents]
    end
    
    subgraph "AI Agents"
        CA[Carbon Accounting Agent<br/>Calculates emissions]
        BA[Benchmarking Agent<br/>Compares performance]
        NZ[Net Zero Planner<br/>Creates pathways]
    end
    
    subgraph "Calculation Engines"
        CCE[Carbon Calculation Engine<br/>GHG Protocol formulas]
        BCE[Benchmarking Engine<br/>Statistical calculations]
        SBTI[SBTi Pathway Calculator<br/>Science-based targets]
    end
    
    subgraph "Data Sources"
        SQL[(SQLite<br/>Emission Factors & Benchmarks)]
        MDB[(MongoDB<br/>Results Storage)]
        VS[(Vector Store<br/>Document Search)]
    end
    
    UI -->|User Queries| API
    API -->|Process Query| L1
    L1 -->|Enhanced Query| L2
    L2 -->|Route to Agent| CA
    L2 -->|Route to Agent| BA
    L2 -->|Route to Agent| NZ
    
    CA -->|Perform Calculation| CCE
    BA -->|Perform Calculation| BCE
    NZ -->|Calculate Pathway| SBTI
    
    CCE -->|Query Data| SQL
    BCE -->|Query Data| SQL
    SBTI -->|Query Data| SQL
    
    CA -->|Store Results| MDB
    BA -->|Store Results| MDB
    NZ -->|Store Results| MDB
    
    L2 -->|Search Documents| VS
    
    style CA fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000
    style BA fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style NZ fill:#e8f5e9,stroke:#388e3c,stroke-width:2px,color:#000
    style L1 fill:#fff3e0,stroke:#f57c00,stroke-width:2px,color:#000
    style L2 fill:#fff3e0,stroke:#f57c00,stroke-width:2px,color:#000
```

**Key Components Explained:**

1. **User Interface (React Frontend)**: Where users interact with the system
2. **Backend API (FastAPI)**: Handles all requests and coordinates processing
3. **Layer 1 (Query Classification)**: Understands if the query is a greeting, needs clarification, or should be processed
4. **Layer 2 (RAG + Tool Router)**: 
   - Searches uploaded documents for relevant information
   - Routes queries to the appropriate AI agent
5. **AI Agents**: Specialized agents that handle specific tasks
6. **Calculation Engines**: Perform actual mathematical calculations using verified formulas
7. **Data Sources**: Store emission factors, benchmarks, results, and documents

---

## How It Works - Agentic Architecture

### Complete Query Processing Flow

```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'primaryColor':'#ffffff', 'primaryTextColor':'#000000', 'primaryBorderColor':'#000000', 'lineColor':'#000000', 'secondaryColor':'#f0f0f0', 'tertiaryColor':'#e0e0e0'}}}%%
graph TD
    START([User Submits Query]) --> CLASSIFY{Query Type?}
    
    CLASSIFY -->|Simple Greeting| GREET[Direct Response]
    CLASSIFY -->|Can Answer from Memory| MEMORY[Answer from History]
    CLASSIFY -->|Needs More Info| CLARIFY[Ask Clarification Questions]
    CLASSIFY -->|Complex Query| PROCESS[Process Query]
    
    CLARIFY --> USER_INPUT[User Provides Info]
    USER_INPUT --> PROCESS
    
    PROCESS --> FEW_SHOT{Pre-defined Answer?}
    FEW_SHOT -->|Yes| INSTANT[Instant Response]
    FEW_SHOT -->|No| SEARCH[Search Documents]
    
    SEARCH --> ROUTE{Which Agent?}
    
    ROUTE -->|Carbon Calculation| CARBON[Carbon Accounting Agent]
    ROUTE -->|Benchmarking| BENCH[Benchmarking Agent]
    ROUTE -->|Net Zero Planning| NETZERO[Net Zero Planner]
    ROUTE -->|General Question| RAG[Direct RAG Response]
    
    CARBON --> EXTRACT1[Extract: Activity, Amount, Region]
    BENCH --> EXTRACT2[Extract: Company, Metric, Value, Sector]
    NETZERO --> EXTRACT3[Extract: Current Emissions, Target Year]
    
    EXTRACT1 --> CHECK1{Data Complete?}
    EXTRACT2 --> CHECK2{Data Complete?}
    EXTRACT3 --> CHECK3{Data Complete?}
    
    CHECK1 -->|No| CLARIFY
    CHECK2 -->|No| CLARIFY
    CHECK3 -->|No| CLARIFY
    
    CHECK1 -->|Yes| CALC1[Calculate Emissions]
    CHECK2 -->|Yes| CALC2[Calculate Benchmarks]
    CHECK3 -->|Yes| CALC3[Calculate Pathway]
    
    CALC1 --> RESULT1[Emission Results + Charts]
    CALC2 --> RESULT2[Benchmark Results + Charts]
    CALC3 --> RESULT3[Net Zero Plan + Charts]
    
    RESULT1 --> RESPONSE[Generate Natural Language Response]
    RESULT2 --> RESPONSE
    RESULT3 --> RESPONSE
    RAG --> RESPONSE
    INSTANT --> RESPONSE
    MEMORY --> RESPONSE
    GREET --> RESPONSE
    
    RESPONSE --> STREAM[Stream Response to User]
    
    style CARBON fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#000
    style BENCH fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000
    style NETZERO fill:#e8f5e9,stroke:#388e3c,stroke-width:2px,color:#000
    style ROUTE fill:#fff3e0,stroke:#f57c00,stroke-width:2px,color:#000
    style RESPONSE fill:#c8e6c9,stroke:#388e3c,stroke-width:2px,color:#000
```

**Flow Explanation:**

1. **User submits a query** (e.g., "Calculate carbon footprint for 1000 liters of diesel")
2. **System classifies the query** - Is it simple? Does it need clarification? Should it be processed?
3. **If processing needed:**
   - Checks for pre-defined answers (few-shot matching)
   - Searches uploaded documents for context
   - Routes to appropriate AI agent
4. **Agent extracts structured data** from the query
5. **If data incomplete**, asks clarification questions
6. **If data complete**, performs actual calculations
7. **Generates response** with results, charts, and explanations
8. **Streams response** back to user in real-time

---

## The Three AI Agents

### 1. Carbon Accounting Agent

**Purpose**: Calculate carbon footprints for various activities (fuel consumption, electricity, waste, etc.)

**How It Works**:
1. Extracts activity type, amount, and region from your query
2. Looks up verified emission factors from databases (IPCC, IEA, EPA)
3. Performs calculations using GHG Protocol Standard formulas
4. Generates detailed results with calculation proofs

**Key Features**:
- Supports Scope 1, 2, and 3 emissions
- Handles multiple activities simultaneously
- Shows emission factor sources and confidence levels
- Generates visualizations (pie charts, bar charts)
- Stores results for tracking over time

**Calculation Example**:
```
Input: "Calculate carbon footprint for 1000 liters of diesel in India"
Process:
  1. Extract: activity=diesel, amount=1000 liters, region=India
  2. Lookup emission factor: 2.68 kg CO2e/liter (IPCC 2006)
  3. Calculate: 1000 × 2.68 = 2,680 kg CO2e = 2.68 tonnes CO2e
  4. Return: Detailed breakdown with proof
```

### 2. Benchmarking Agent

**Purpose**: Compare your company's ESG metrics against industry peers and benchmarks

**How It Works**:
1. Extracts company name, metric, value, and sector from query or uploaded files
2. Queries benchmark database for industry averages and percentiles
3. Calculates percentile rank using statistical interpolation
4. Performs gap analysis to identify improvement opportunities

**Key Features**:
- Supports single metric, multi-metric, and peer comparison
- Works with uploaded CSV/Excel files containing multiple companies
- Calculates statistical percentiles (P10, P25, P50, P75, P90, P95)
- Generates comparison charts (bar charts, radar charts)
- Identifies best-in-class performers

**Calculation Example**:
```
Input: "Compare TATA's carbon intensity of 0.15 tCO2e/revenue against technology sector in India"
Process:
  1. Extract: company=TATA, metric=carbon_intensity, value=0.15, sector=Technology, region=India
  2. Query benchmarks: P25=0.12, P50=0.18, P75=0.25, P90=0.35
  3. Calculate percentile rank: 0.15 falls between P25 and P50
  4. Interpolate: ~42nd percentile
  5. Return: "TATA performs better than 42% of technology companies in India"
```

### 3. Net Zero Planner

**Purpose**: Create science-based pathways to achieve net-zero emissions by 2050

**How It Works**:
1. Extracts current emissions, target year, and scenario preference
2. Calculates SBTi-aligned reduction pathway (1.5°C or well-below 2°C)
3. Scores and ranks decarbonization initiatives
4. Generates year-by-year emission targets and recommendations

**Key Features**:
- Supports SBTi 1.5°C and well-below 2°C scenarios
- Calculates interim targets (2030, 2040, 2050)
- Scores initiatives by impact, cost, and feasibility
- Recommends top initiatives with multi-criteria analysis
- Validates pathway compliance with SBTi standards

**Calculation Example**:
```
Input: "Create net zero plan for 10,000 tonnes CO2e, target 2050, 1.5°C scenario"
Process:
  1. Extract: current_emissions=10,000 tCO2e, target_year=2050, scenario=1.5°C
  2. Calculate pathway: 7% annual reduction rate
  3. Milestones:
     - 2030: 5,000 tCO2e (50% reduction)
     - 2040: 2,500 tCO2e (75% reduction)
     - 2050: 0 tCO2e (net-zero)
  4. Score initiatives: Renewable energy, Energy efficiency, etc.
  5. Return: Complete plan with pathway and recommendations
```

---

## Sample Queries & Use Cases

### Carbon Accounting Agent Queries

**Basic Calculations:**
- "Calculate carbon footprint for 1000 liters of diesel fuel in India"
- "What is the carbon emissions for 50,000 kWh of electricity consumption in USA?"
- "Calculate Scope 1 emissions for 500 kg of waste generation in Germany"

**Multiple Activities:**
- "Calculate total carbon footprint: 2000 liters diesel, 30,000 kWh electricity, 1000 kg waste in India"
- "What are the emissions for our fuel consumption (500L diesel) and electricity (20,000 kWh) in USA?"

**Scope-Specific:**
- "Calculate Scope 2 emissions for 25,000 kWh grid electricity in India"
- "What are our Scope 1 emissions from diesel (1000L) and gasoline (500L) consumption?"

**With Company Context:**
- "TATA, Q1 2024: Calculate carbon footprint for 1000 liters diesel in India"
- "Calculate emissions for Reliance: 50,000 kWh electricity, 2000 liters diesel, India"

### Benchmarking Agent Queries

**Single Metric Benchmarking:**
- "Compare TATA's carbon intensity of 0.15 tCO2e/revenue against technology sector in India for 2024"
- "How does our water usage of 1.7 million m³ compare to manufacturing sector peers in Asia?"
- "Benchmark our renewable energy percentage of 38% against energy sector in India"

**Peer Comparison:**
- "Compare TATA's carbon emissions against Reliance, Infosys, and TCS in India"
- "How do we compare to our competitors on carbon intensity, water usage, and renewable energy %?"

**File-Based Benchmarking:**
- Upload CSV file with multiple companies, then: "Compare all companies in the file on carbon emissions"
- "Benchmark TATA against all other companies in the file for water usage"

**Gap Analysis:**
- "What percentile is our carbon intensity of 0.15 tCO2e/revenue in technology sector?"
- "How far are we from the 75th percentile in water usage for manufacturing sector?"

### Net Zero Planner Queries

**Basic Net Zero Planning:**
- "Create net zero plan for 10,000 tonnes CO2e, target year 2050"
- "What is the pathway to net zero for 50,000 tonnes CO2e by 2050 using 1.5°C scenario?"

**With Scenario Selection:**
- "Create net zero plan for 25,000 tonnes CO2e, 1.5°C scenario, target 2050"
- "Plan net zero pathway for 100,000 tonnes CO2e using well-below 2°C scenario"

**With Historical Data:**
- "Create net zero plan: current emissions 10,000 tCO2e, base year 2020 emissions 12,000 tCO2e, target 2050"
- "Plan pathway to net zero: we emit 50,000 tonnes CO2e now, baseline was 60,000 in 2020"

**Initiative-Focused:**
- "What initiatives should we prioritize to achieve net zero by 2050 for 10,000 tonnes CO2e?"
- "Recommend top 5 decarbonization initiatives for 25,000 tonnes CO2e reduction pathway"

### File-Based Queries

**After uploading a CSV file with company ESG data:**
- "Compare all companies in the file on carbon emissions"
- "Benchmark TATA against all others for water usage and energy consumption"
- "What is the net zero pathway for TATA based on the file data?"
- "Compare TATA, Reliance, and Infosys on all metrics in the file"

---

## Sample Data Files

The system includes sample CSV files in the `sample_data/` folder that demonstrate the expected format:

### Sample File: `indian_companies_esg_benchmark_2024.csv`

**Format:**
```csv
Company Name,Sector,Region,Country,Year,Carbon Emissions (tonnes CO2e),Water Usage (m³),Energy Consumption (MWh),Waste Generated (tonnes),Renewable Energy %,Employee Count
TATA,Manufacturing,Asia,India,2024,260000,1700000,1200000,26000,38,450000
Reliance,Energy,Asia,India,2024,5200000,35000000,25000000,520000,15,250000
Infosys,Technology,Asia,India,2024,125000,850000,450000,12500,65,350000
```

**Required Columns:**
- **Company Name**: Name of the company
- **Sector**: Industry sector (Technology, Manufacturing, Energy, etc.)
- **Region/Country**: Geographic location
- **Year**: Data year
- **Metrics**: Carbon Emissions, Water Usage, Energy Consumption, Waste Generated, Renewable Energy %, etc.

**How to Use:**
1. Upload the CSV file through the web interface
2. System automatically extracts companies, metrics, sectors, and regions
3. Ask questions like:
   - "Compare all companies on carbon emissions"
   - "Benchmark TATA against others for water usage"
   - "What is the net zero pathway for Reliance?"

**Other Sample Files Available:**
- `energy_sector_esg_2024.csv` - Energy sector companies
- `technology_companies_esg_2024.csv` - Technology companies
- `manufacturing_companies_esg_2024.csv` - Manufacturing companies
- `multi_sector_esg_comparison_2024.csv` - Multi-sector comparison
- `scope3_emissions_comparison_2024.csv` - Scope 3 emissions data
- `water_intensive_industries_2024.csv` - Water usage data

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

## Technical Implementation Details

### Two-Layer AI Architecture

**Layer 1: Query Classification**
- **Purpose**: Quickly handle simple queries and classify complex ones
- **Technology**: Google Gemini 2.0 Flash
- **Functions**:
  - Detects greetings → Direct response
  - Answers from conversation memory → Memory response
  - Identifies need for clarification → Shows clarification UI
  - Enhances queries with ESG context → Passes to Layer 2

**Layer 2: RAG + Tool Router**
- **Purpose**: Process complex queries with document search and agent routing
- **Technology**: Google Gemini 2.5 Flash with JSON schema validation
- **Functions**:
  - Searches uploaded documents (vector search)
  - Routes queries to appropriate AI agents
  - Combines agent results with document context
  - Generates comprehensive responses

### Calculation Engines

**Carbon Calculation Engine**
- Uses **GHG Protocol Standard** formulas
- Emission factors from: IPCC 2006, IEA, EPA databases
- Supports: Scope 1 (direct), Scope 2 (indirect energy), Scope 3 (value chain)
- Calculates: Activity Data × Emission Factor = Emissions

**Benchmarking Calculation Engine**
- Statistical methods: Percentile rank, linear interpolation
- Benchmark data: Industry averages, percentiles (P10, P25, P50, P75, P90, P95)
- Calculations: Compare company value against benchmark distribution
- Gap analysis: Identifies distance to target percentiles

**SBTi Pathway Calculator**
- Implements **Science Based Targets initiative** methodology
- Scenarios: 1.5°C (7% annual reduction) and well-below 2°C (5% annual reduction)
- Calculates: Year-by-year emission targets
- Validates: Pathway compliance with SBTi standards

### Data Storage

**MongoDB**
- Stores: Calculation results, benchmark results, net zero plans
- Collections: `carbon_calculations`, `benchmark_results`, `net_zero_plans`
- Enables: Historical tracking, trend analysis, report generation

**SQLite**
- Stores: Emission factors, benchmark data, initiative templates
- Tables: `emission_factors`, `benchmarks`, `initiatives`
- Enables: Fast lookups, offline capability, easy updates

**ChromaDB (Vector Store)**
- Stores: Document embeddings for semantic search
- Enables: Finding relevant information from uploaded documents
- Technology: Vector similarity search

### File Processing

**Supported Formats**: CSV, Excel (.xlsx), PDF

**Processing Flow**:
1. File uploaded → Detects file type
2. Parses file → Extracts structured data
3. Analyzes content → Identifies companies, metrics, sectors, regions
4. Wraps in tags → `[FILE_ANALYSIS]` tags for agent processing
5. Agents extract → Use file data to enhance calculations

**CSV Format Requirements**:
- Headers: Company Name, Sector, Region, Year, Metrics (Carbon Emissions, Water Usage, etc.)
- Example: See `sample_data/indian_companies_esg_benchmark_2024.csv`

### Agent Communication

**Tool Router**
- Uses JSON schema validation to ensure correct tool selection
- Routes based on keywords and file context
- Supports parallel execution of multiple tools
- Handles errors gracefully with fallback mechanisms

**Agent Response Format**:
```json
{
  "status": "success",
  "explanation": "Natural language explanation",
  "calculation_result": { ... },
  "charts": { ... },
  "calculation_proof": { ... },
  "timestamp": "2024-01-01T00:00:00Z"
}
```

### Error Handling

**Clarification System**
- Detects missing required information
- Generates intelligent clarification questions
- Gathers suggestions from database and knowledge base
- Handles user responses and continues processing

**Validation**
- Input validation: Checks data ranges, formats, completeness
- Calculation validation: Verifies formulas, data sources
- Pathway validation: Ensures SBTi compliance

**Fallback Mechanisms**
- LLM timeout → Uses rule-based extraction
- Database unavailable → Uses cached data
- Calculation error → Returns error with helpful message

---

## Conclusion

This documentation provides a comprehensive overview of the FitSol ESG Co-Pilot system from both technical and domain expert perspectives. The system combines:

- **AI Intelligence** for understanding natural language and extracting data
- **Real Calculations** using verified formulas and databases
- **Industry Standards** (GHG Protocol, SBTi) for accuracy and compliance
- **User-Friendly Interface** for easy interaction
- **File Processing** for bulk operations and comparisons

The three AI agents work together to provide a complete ESG intelligence platform that helps organizations measure, benchmark, and plan their carbon emissions reduction strategies.

For technical support or questions, please refer to the codebase or contact the development team.

---

**Document Version**: 1.0  
**Last Updated**: 2024  
**Maintained By**: FitSol Development Team
