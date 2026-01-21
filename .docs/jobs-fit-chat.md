# Jobs Fit Chat Workflow

## Table of Contents

- [Overview](#overview)
- [Technical Architecture](#technical-architecture)
  - [Data Flow](#data-flow)
  - [Key Components](#key-components)
- [Process Flow: Step-by-Step](#process-flow-step-by-step)
- [AI Agent Details](#ai-agent-details)
- [Supported Input Types](#supported-input-types)

## Overview

The **Jobs Fit Chat** workflow provides an interactive chat-based system for real-time conversations about job-candidate matching, skills assessment, and technical questions. It leverages AI agents, vector embeddings, and conversation memory to deliver expert recruitment analysis and technical guidance.

This workflow implements a **conversational AI recruitment assistant** that:
1. Maintains conversation context using PostgreSQL memory
2. Analyzes candidate profiles from Qdrant vector store
3. Provides intelligent matching for job offers
4. Answers technical questions with expert-level responses
5. Supports both German and English languages

## Technical Architecture

### Data Flow

```
User Input (Chat Trigger)
    ↓
Jobs Fit Agent (Mistral LLM)
    ├→ Qdrant Vector Store (Candidate Profiles)
    ├→ Think Tool (Internal Reasoning)
    └→ Postgres Chat Memory (Conversation History)
        ↓
Intelligent Response Generation
    ↓
Chat Response (Recruiter-style Output)
```

### Key Components

#### 1. **Chat Interface: Webhook Trigger**
- **Node:** Chat
- **Type:** n8n-nodes-langchain.chatTrigger
- **Mode:** Public webhook
- **Purpose:** Receives user messages via webhook interface
- **Webhook ID:** 2faf8edb-f103-4860-8ef5-6b95814421eb

#### 2. **AI Agent: Jobs Fit**
- **Type:** LangChain Agent with Mistral 14B
- **LLM Model:** ministral-14b-latest
- **Temperature:** 0.1 (balanced creativity/determinism)
- **Role:** Senior IT Recruiter & Technical Analyst (IQ 120)
- **Purpose:** Processes user inputs and generates expert responses

#### 3. **Vector Store: Qdrant**
- **Node:** Qdrant Vector Store
- **Collection:** skills
- **Purpose:** Stores candidate profile embeddings for skills, projects, and technical expertise
- **Tool Integration:** Provides candidate data as AI tool for matching analysis

#### 4. **Internal Reasoning: Think Tool**
- **Node:** Think
- **Type:** n8n-nodes-langchain.toolThink
- **Purpose:** Internal reasoning tool for score calculations and technical analysis
- **Output:** Factual, deterministic responses based on vector store data

#### 5. **Conversation Memory: PostgreSQL**
- **Node:** Postgres Chat Memory
- **Table:** Job_Fit_History
- **Context Window:** 500 tokens
- **Purpose:** Maintains conversation history for contextual responses

#### 6. **Embeddings: Google Gemini**
- **Node:** Embeddings Google Gemini
- **Purpose:** Generates vector embeddings for candidate profile storage and retrieval
- **Integration:** Powers Qdrant vector store operations

## Process Flow: Step-by-Step

### Step 1: User Input Reception
- Chat trigger receives user message via webhook
- Supports both German and English input

### Step 2: Input Classification
- Agent analyzes input to determine type:
  - **Job Offer:** Contains keywords like PHP, Magento, REST API, Developer, etc.
  - **General Question:** Technical questions, best practices, architecture discussions

### Step 3: Data Retrieval
- Queries Qdrant vector store for relevant candidate profile data
- Retrieves skills, projects, tools, and technical stack information
- If no profile exists: Returns `STATUS: NO_PROFILE_DATA_FOUND`

### Step 4: Intelligent Processing
- **Job Offers:** Calculates match scores using weighted algorithm
- **Questions:** Provides expert technical answers
- Maintains conversation context using PostgreSQL memory

### Step 5: Response Generation
- Generates recruiter-style responses in user's language
- Includes match scores, skill mapping, and technical insights
- Stores conversation history for continuity

## AI Agent Details

### Jobs Fit Agent Architecture

The agent operates as a **Senior IT Recruiter & Technical Analyst** with specialized capabilities:

#### Core Prompt Features:
- **Language Detection:** Responds in the same language as user input (German/English)
- **Data-Driven Analysis:** Always searches Qdrant first for candidate profiles
- **Deterministic Output:** Uses fixed algorithms for scoring and analysis
- **Professional Tone:** Direct, recruiter-style communication

#### Scoring Algorithm (Job Offers):
- **Skills Alignment [45%]:** PHP, Magento, Linux compatibility
- **Role Fit [25%]:** ERP, Enterprise, Dynamics 365 alignment
- **Seniority Level [20%]:** Years of experience assessment
- **Technical Tooling [10%]:** C#, Azure, Windows Server compatibility

**Final Score:** Integer 1-100 (1 if no profile data found)

### Think Tool Integration
The internal reasoning tool provides:
- Match percentage calculations
- Concise explanations based on vector store data
- Factual, deterministic outputs
- Skills and experience analysis

## Supported Input Types

### 1. Job Offers (German/English)
**Keywords Detected:** PHP, Magento, Symfony, Laravel, REST, API, Backend, Frontend, Fullstack, Developer, Engineer, Software Engineer, Entwickler, Softwareentwickler, Webentwickler, Programmierer, Schnittstelle

**Output Structure:**
1. **Match Score: XX %** (always first)
2. Job-Mapping (requirements ↔ skills)
3. Concrete project examples
4. Strengths & Gaps (honest assessment)
5. Skill-transfer assessment if technologies missing

### 2. General Questions (Technical)
**Examples:**
- Architecture decisions
- Best practices
- Technology trade-offs
- Implementation strategies
- Technical problem-solving

**Response Style:**
- Direct expert answers
- Senior-level technical guidance
- Hands-on practical advice
- No HR framing (pure technical focus)

### Language Support
- **German (Deutsch):** Native support for German job postings and questions
- **English:** Full English language processing
- **Mixed Input:** Responds in detected input language

### Memory & Context
- **Conversation History:** Maintains context across messages
- **Session Continuity:** Remembers previous interactions
- **Context Window:** 500 tokens for relevant history retention
