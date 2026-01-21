# n8n Workflows Documentation

## Overview
This repository contains n8n automation workflows designed for intelligent job-candidate matching using AI-powered analysis and vector embeddings.

---

## Workflow: Job Offers Relevance

### Purpose
The **Job Offers Relevance** workflow automates the intelligent matching between job postings and candidate profiles. It leverages AI agents and vector embeddings to calculate precise match scores based on skills, role alignment, seniority level, and technical tooling compatibility.

### Workflow Summary
This workflow implements a **three-stage agent-based matching engine** that:
1. Analyzes job requirements dynamically
2. Retrieves relevant candidate profiles from a vector database
3. Calculates weighted match scores
4. Persists results to a database

### Execution Schedule
- **Trigger:** Daily at 11:00 AM (configurable)
- **Frequency:** Automated daily execution

---

## Technical Architecture

### Data Flow

```
11am Trigger
    ↓
Job Offers (PostgreSQL Query)
    ↓
Loop Over Items (Batch Processing)
    ├→ If (Description Validation)
    │   ├→ (TRUE) The Scout Agent → The Analyst → The Matchmaker
    │   └→ (FALSE) Edit Fields (Fallback Score: 1)
    └→ Aggregate & Split Out
        ↓
    Job Offers - store (Sub-workflow)
```

### Key Components

#### 1. **Data Source: PostgreSQL**
- **Node:** Job Offers
- **Operation:** SELECT query filtering job postings
- **Filter Condition:** `match_score_percentage = 0` (unprocessed jobs)
- **Database:** `job_offers` table in public schema
- **Purpose:** Retrieves job postings that need relevance scoring

#### 2. **Batch Processing**
- **Node:** Loop Over Items
- **Type:** Split In Batches (n8n-nodes-base.splitInBatches)
- **Purpose:** Iterates through each job offer for individual processing
- **Branching Logic:**
  - Continues to agent pipeline if job description is valid
  - Routes to fallback if description is missing (null)

#### 3. **AI Agent Pipeline**

##### Stage 1: The Scout (Job Analysis & Candidate Retrieval)
- **Type:** LangChain Agent with Mistral 14B
- **LLM Model:** ministral-14b-latest
- **Temperature:** 0.1 (deterministic output)
- **Purpose:**
  - Dynamically extracts actual job requirements from title and description
  - Queries Qdrant vector store for matching candidate profiles
  - Generates evidence report comparing job requirements against candidate capabilities
  
- **Process:**
  1. Analyzes job title and description for actual requirements
  2. Searches vector store using relevant skills and experience queries
  3. Extracts structured evidence on:
     - **Skills & Technologies:** Programming languages, frameworks, tools
     - **Role & Experience:** Position type and candidate background alignment
     - **Seniority Level:** Years of experience required vs. candidate tenure
     - **Domain Expertise:** Industry or specialized knowledge
  
- **Output Schema:** Structured evidence report with status flags

##### Stage 2: The Analyst (Weighted Scoring)
- **Type:** LangChain Chain LLM with Mistral Large
- **LLM Model:** mistral-large-latest
- **Temperature:** 0 (maximum determinism)
- **Purpose:** Calculates precise weighted match scores using mathematical formulation
  
- **Scoring Formula (45/25/20/10 Weighting):**
  - **Skills Alignment [45%]:** Match percentage for required technical skills
  - **Role Alignment [25%]:** Fit between job role and candidate background
  - **Seniority Level [20%]:** Experience requirements fulfillment
  - **Technical Tooling [10%]:** Platform/tool stack compatibility
  
- **Fallback Logic:**
  - If no candidate profile data found: `score = 1` (lowest possible)
  - Ensures all jobs receive a numerical score
  
- **Output:** Calculation steps + final integer score (1-100)

##### Stage 3: The Matchmaker (JSON Formatting)
- **Type:** LangChain Chain LLM with Mistral Small
- **LLM Model:** magistral-small-latest
- **Temperature:** 0 (strict formatting)
- **Purpose:** Transforms analytical output into standardized JSON format
  
- **Process:**
  1. Extracts final score from Analyst reasoning
  2. Maps job metadata (ID, title) to output structure
  3. Produces clean, validated JSON
  
- **Output Schema:**
  ```json
  {
    "id": "string (job identifier)",
    "title": "string (job title)",
    "match_score_percentage": "integer (1-100)"
  }
  ```

#### 4. **Vector Database: Qdrant**
- **Node:** Qdrant Vector Store
- **Collection:** skills
- **Embedding Model:** Google Gemini (PaLM API)
- **Purpose:** Stores high-dimensional vector embeddings of candidate profiles
  
- **Content:**
  - Candidate technical skills and expertise
  - Programming languages (PHP, Python, JavaScript, C#)
  - Frameworks (Magento, Symfony, Laravel, .NET)
  - APIs and integration patterns (REST, GraphQL)
  - Frontend/Backend/Fullstack specialization markers
  - Domain experience (E-Commerce, CI/CD, Cloud platforms)
  - Project-based technologies (ERP, CRM, etc.)
  
- **Function:** Enables semantic similarity search during candidate retrieval

#### 5. **Embedding Generation**
- **Node:** Embeddings Google Gemini
- **Provider:** Google PaLM API
- **Purpose:** Converts job descriptions and candidate skills to vector embeddings
- **Used By:** Qdrant Vector Store for semantic search operations

#### 6. **Data Persistence: Sub-workflow**
- **Node:** Job Offers - store
- **Type:** Execute Workflow (async, non-blocking)
- **Target Workflow:** `Job Offers - store` (ID: RZooeNSj1IKG6U-NvDA_y)
- **Purpose:**
  - Stores calculated match scores back to database
  - Updates job posting records with relevance percentage
  - Runs asynchronously to improve main workflow performance
  - Converts all fields to string format for compatibility

#### 7. **Validation & Fallback**
- **Node:** If (Conditional Router)
- **Condition:** `json.description != null`
- **True Path:** Routes to Scout agent for full analysis
- **False Path:** Routes to Edit Fields for fallback scoring
- **Fallback Values:**
  - Score: 1 (minimum relevance)
  - Preserves job ID and title
  - Handles data quality issues gracefully

#### 8. **Output Structuring**
- **Nodes:** Aggregate → Split Out
- **Purpose:** 
  - Aggregates results from agent pipeline
  - Splits individual job records for database storage
  - Ensures data consistency before persistence

#### 9. **Structured Output Parsers**
Three LangChain output parsers enforce schema validation:

1. **Scout Parser:** Validates evidence structure
   - Status: "PROFILE_DATA_FOUND" | "NO_PROFILE_DATA_FOUND"
   - Evidence: skills, role, seniority, tooling descriptions

2. **Analyst Parser:** Validates scoring output
   - Calculation steps (array)
   - Final score (integer, 1-100)

3. **Matchmaker Parser:** Validates final JSON
   - id, title, match_score_percentage
   - Prevents additional properties (strict mode)

---

## External Integrations

### API Credentials Required
1. **PostgreSQL:** `Postgres account` - Database connection for job offers
2. **Google PaLM API:** `Google Gemini(PaLM) Api account` - Embedding generation
3. **Qdrant API:** `QdrantApi account` - Vector store access
4. **Mistral Cloud API:** `Mistral Cloud account` - LLM models (14B, Large, Small)

### LLM Models Used
- **Mistral 14B** (ministral-14b-latest): Agent reasoning
- **Mistral Large** (mistral-large-latest): Weighted calculations
- **Mistral Small** (magistral-small-latest): Output formatting

---

## Process Flow: Step-by-Step

### Step 1: Trigger
Daily scheduler activates workflow at 11:00 AM

### Step 2: Data Retrieval
PostgreSQL node fetches all job offers with `match_score_percentage = 0`

### Step 3: Batch Iteration
Loop processes jobs one at a time (configurable batch size)

### Step 4: Validation
If node checks if job description exists
- **Valid:** Proceeds to Scout agent
- **Invalid:** Assigns fallback score of 1

### Step 5: Agent Analysis (If Valid)
1. **Scout:** Analyzes job requirements and retrieves candidate profiles
2. **Analyst:** Calculates weighted match score
3. **Matchmaker:** Formats output as JSON

### Step 6: Output Processing
- Aggregates all job results
- Splits records for individual processing
- Converts to database-compatible format

### Step 7: Persistence
Sub-workflow stores results back to database asynchronously

---

## Configuration & Customization

### Adjustment Points

#### Scheduling
- **Location:** "11am" node → rule.interval[0].triggerAtHour
- **Current:** 11 (11:00 AM)
- **Format:** 24-hour integer (0-23)

#### LLM Temperature Settings
- **Scout:** 0.1 (allow some variation in analysis)
- **Analyst:** 0 (strict mathematical output)
- **Matchmaker:** 0 (precise JSON formatting)
- **Use Case:** Lower temperatures = more deterministic; higher = more creative

#### Weighted Scoring Formula
Located in "The Analyst" node:
- Skills: 45%
- Role: 25%
- Seniority: 20%
- Tooling: 10%
- **Modification:** Edit scoring weights in agent prompt

#### Vector Store Collection
- **Collection Name:** skills
- **Embedding Model:** Google Gemini
- **Use Case:** Customize candidates to match in Qdrant configuration

#### Fallback Score
- **Location:** "Edit Fields" node
- **Current Value:** 1 (minimum relevance)
- **Modifiable:** Change `match_score_percentage` assignment

---

## Error Handling

### Graceful Degradation
1. **Missing Description:** Fallback score assigned (1)
2. **No Candidate Profiles:** Score = 1 per Analyst fallback logic
3. **API Failures:** n8n automatically retries on timeout

### Data Quality Checks
- Description validation prevents NULL processing
- Structured output parsers reject malformed agent responses
- Database uniqueness constraints prevent duplicates

---

## Performance Considerations

- **Batch Processing:** Loop Over Items can be configured for parallel execution
- **Async Persistence:** Sub-workflow runs non-blocking (waitForSubWorkflow: false)
- **Model Selection:** Mistral Small for final formatting reduces computational cost
- **Temperature Settings:** Low temperatures minimize token consumption

---

## Monitoring & Debugging

### Key Metrics
- **Execution Time:** Monitor each agent stage duration
- **Score Distribution:** Check if scores cluster inappropriately
- **Error Rate:** Track fallback assignments (score = 1)
- **Database Updates:** Verify sub-workflow successfully persists results

### Logs to Review
1. PostgreSQL query results (job count)
2. Agent reasoning in "1. The Scout" output
3. Calculation steps in "2. The Analyst" output
4. JSON structure in "3. The Matchmaker" output

---

## Future Enhancements

- Dynamic weighting adjustments based on job category
- Multi-language support for international job postings
- Candidate notification pipeline on high-relevance matches
- A/B testing of different LLM models
- Real-time scoring instead of batch processing
