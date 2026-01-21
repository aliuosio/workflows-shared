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
