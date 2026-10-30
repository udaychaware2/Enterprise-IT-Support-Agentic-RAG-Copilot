# Enterprise IT Support Agentic RAG Copilot

An end-to-end **Forward Deployed Engineer (FDE) project** that transforms an Agentic RAG prototype into a deployable enterprise IT support application.

Built using **LangGraph, FastAPI, Pinecone, Groq, Tavily, and JavaScript**, the system intelligently retrieves information from private company documents, evaluates evidence quality, and falls back to web search when internal knowledge is insufficient.

The application includes a web interface, document ingestion pipeline, REST APIs, and decision-path auditing.

---

## 1. Project Overview

### The Business Problem

Consider **NovaRetail**, a fictional retail company with 3,000 employees.

Its internal IT department maintains documentation covering:

- VPN configuration and troubleshooting
- Password and MFA policies
- Software installation procedures
- Laptop troubleshooting guides
- IT service-desk runbooks

Despite having these resources, employees continue creating repetitive support tickets because finding the correct information is difficult.

Traditional keyword search often returns irrelevant documents, while generic AI chatbots can generate answers that are not supported by company policies.

Additionally, some questions require information that may not exist in internal documentation, such as current software outages or vendor announcements.

### The Solution

This project builds an AI-powered IT Support Copilot that:

1. Searches the company's private knowledge base first.
2. Evaluates the relevance and quality of retrieved information.
3. Uses external web search when internal evidence is insufficient.
4. Rewrites unsuccessful queries and retries retrieval.
5. Generates answers grounded in retrieved evidence.
6. Provides a decision trace explaining how each answer was produced.
7. Allows authorized administrators to upload new company documents.

### Example Use Cases

**Internal knowledge question**

> How do I connect to the company VPN from home?

The system retrieves the relevant VPN instructions from the private knowledge base without unnecessarily searching the internet.

**External information question**

> What is the latest Microsoft Teams outage guidance?

If the internal knowledge base cannot answer the question, the system searches the web, evaluates the retrieved evidence, and generates an answer using external sources.

---

## 2. Why This Is an FDE Project

A Forward Deployed Engineer does more than build an AI model or demonstrate an LLM in a notebook.

The responsibility is to understand a business problem and translate it into a working software solution.

This project demonstrates that complete engineering workflow:

```text
Customer Problem
       |
       v
Discovery & Requirements
       |
       v
Solution Architecture
       |
       v
Knowledge Integration
       |
       v
Agentic RAG Development
       |
       v
API Development
       |
       v
Frontend Development
       |
       v
Testing & Audit
       |
       v
Deployment
       |
       v
Continuous Improvement
```

The repository brings together AI engineering, backend development, knowledge integration, and product development into one application.

---

## 3. System Architecture

```text
                 Employee / User
                        |
                        v
              HTML / CSS / JavaScript
                    Web UI
                        |
                        | POST /api/chat
                        v
                     FastAPI
                        |
                        v
                    LangGraph
                 Agentic RAG Workflow
                        |
               +--------+--------+
               |                 |
               v                 v
          Private KB         External Search
           Pinecone             Tavily
               |                 |
               +--------+--------+
                        |
                        v
                    Groq LLM
                        |
                        v
                 Grounded Answer
                        |
                        v
                  Web Interface
```

### Component Responsibilities

| Component | Responsibility |
|---|---|
| LangGraph | Orchestrates the agent workflow and conditional routing |
| Groq | Handles routing, evidence grading, query rewriting, and answer generation |
| Pinecone | Stores and retrieves private company knowledge |
| HuggingFace Embeddings | Converts document chunks into vector representations |
| Tavily | Provides external web search |
| FastAPI | Exposes application functionality through REST APIs |
| HTML/CSS/JavaScript | Provides the employee-facing interface |
| SQLite | Stores basic decision-path audit information |
| Docker | Supports reproducible application packaging |

---

## 4. Agentic RAG Workflow

The central component of this project is a stateful workflow implemented using LangGraph.

Unlike traditional RAG, which follows a relatively straightforward retrieval-and-generation process, this system evaluates its current evidence and decides what action to take next.

### Workflow

```text
                  User Question
                       |
                       v
                 [1] Route Question
                       |
             +---------+---------+
             |                   |
             v                   v
          DIRECT                 KB
             |                   |
             v                   v
       Direct Answer      [2] Retrieve from
                           Private Pinecone KB
                                  |
                                  v
                        [3] Grade KB Evidence
                                  |
                          +-------+-------+
                          |               |
                         GOOD            WEAK
                          |               |
                          v               v
                    Generate KB     [4] Tavily Search
                       Answer              |
                                           v
                                  [5] Grade Web Evidence
                                           |
                                    +------+------+
                                    |             |
                                   GOOD          WEAK
                                    |             |
                                    v             v
                              Generate Web   [6] Rewrite
                                 Answer          Query
                                                   |
                                                   v
                                            Retry Retrieval
                                                   |
                                                   v
                                            Retry Limit?
                                                   |
                                                   v
                                          Insufficient Evidence
```

### Workflow Components

**1. Question Routing**

Determines whether an incoming message is a simple conversational interaction or an IT support question requiring retrieval.

**2. Private Knowledge Retrieval**

Searches the company's Pinecone knowledge base for relevant document chunks.

**3. Evidence Grading**

Evaluates whether the retrieved private documents contain sufficient information to answer the question.

**4. External Web Search**

If internal evidence is insufficient, Tavily searches for relevant external information.

**5. Web Evidence Grading**

Evaluates whether the external search results provide useful evidence.

**6. Query Rewriting**

If retrieval remains unsuccessful, the system rewrites the original question to improve search relevance and retries the private knowledge retrieval process.

A retry guard prevents the workflow from entering an endless loop.

### Why Agentic RAG?

Traditional RAG:

```text
Question
   |
   v
Retrieve Documents
   |
   v
Generate Answer
```

This project:

```text
Question
   |
   v
Route
   |
   v
Retrieve
   |
   v
Evaluate Evidence
   |
   v
Choose Information Source
   |
   v
Rewrite if Necessary
   |
   v
Retry
   |
   v
Generate Grounded Answer
```

The key difference is that the application makes decisions based on the current workflow state instead of blindly generating an answer from retrieved documents.

---

## 5. Technology Stack

| Category | Technology | Purpose |
|---|---|---|
| Agent Orchestration | LangGraph | Stateful agent workflow |
| LLM | Groq | Reasoning, routing, grading, and generation |
| Embeddings | HuggingFace | Document vectorization |
| Vector Database | Pinecone | Private knowledge storage and retrieval |
| Web Search | Tavily | External information retrieval |
| Backend | FastAPI | REST API development |
| Frontend | HTML, CSS, JavaScript | User interface |
| Audit Logging | SQLite | Query and decision-path logging |
| Packaging | Docker | Application packaging |

---

## 6. Project Structure

```text
FDE_Agentic_RAG_IT_Copilot/
│
├── app/
│   ├── api/
│   │   └── routes.py
│   │
│   ├── core/
│   │   ├── config.py
│   │   └── logging.py
│   │
│   ├── rag/
│   │   ├── state.py
│   │   ├── vectorstore.py
│   │   └── workflow.py
│   │
│   ├── services/
│   │   ├── audit.py
│   │   └── ingestion.py
│   │
│   └── main.py
│
├── data/
│   └── sample_kb/
│       ├── company_it_handbook.md
│       └── service_desk_runbook.md
│
├── static/
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── app.js
│
├── templates/
│   └── index.html
│
├── tests/
│   └── test_ingestion.py
│
├── uploads/
│
├── .env.example
├── Dockerfile
├── ingest_sample_kb.py
├── requirements.txt
├── run.py
└── README.md
```

### Important Modules

| Module | Description |
|---|---|
| `state.py` | Defines LangGraph state and structured decisions |
| `vectorstore.py` | Handles Pinecone and embedding integration |
| `workflow.py` | Implements the complete Agentic RAG workflow |
| `ingestion.py` | Loads, chunks, and processes uploaded documents |
| `audit.py` | Records basic query audit information |
| `routes.py` | Exposes chat, health, and ingestion endpoints |
| `config.py` | Manages environment configuration |
| `main.py` | Initializes the FastAPI application |

---

## 7. Getting Started

### Prerequisites

- Python 3.10+
- Pinecone account and API key
- Groq API key
- Tavily API key
- OpenAI API key, if using the configured OpenAI embedding or fallback components

### Step 1: Clone the Repository

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd FDE_Agentic_RAG_IT_Copilot
```

### Step 2: Create a Virtual Environment

```bash
python -m venv venv
```

Activate it:

**Windows**

```bash
venv\Scripts\activate
```

**macOS / Linux**

```bash
source venv/bin/activate
```

### Step 3: Install Dependencies

```bash
pip install -r requirements.txt
```

### Step 4: Configure Environment Variables

Create a `.env` file in the project root.

```env
# LLM API Keys
GROQ_API_KEY=your_groq_api_key
OPENAI_API_KEY=your_openai_api_key

# External Search
TAVILY_API_KEY=your_tavily_api_key

# Pinecone
PINECONE_API_KEY=your_pinecone_api_key
PINECONE_INDEX_NAME=fde-it-support-rag
PINECONE_NAMESPACE=company-it-kb

# LLM Models
GROQ_MODEL=openai/gpt-oss-20b
OPENAI_MODEL=gpt-4o-mini
EMBEDDING_MODEL=text-embedding-3-small

# Security
ADMIN_API_KEY=your_secure_admin_key

# Application
APP_ENV=development
```

### Environment Variable Reference

| Variable | Description |
|---|---|
| `GROQ_API_KEY` | Groq API authentication |
| `OPENAI_API_KEY` | OpenAI API authentication |
| `TAVILY_API_KEY` | Tavily search authentication |
| `PINECONE_API_KEY` | Pinecone authentication |
| `PINECONE_INDEX_NAME` | Pinecone index name |
| `PINECONE_NAMESPACE` | Knowledge base namespace |
| `GROQ_MODEL` | Groq model identifier |
| `OPENAI_MODEL` | OpenAI model identifier |
| `EMBEDDING_MODEL` | Embedding model identifier |
| `ADMIN_API_KEY` | Protects administrative ingestion endpoints |
| `APP_ENV` | Application environment |

**Security considerations**

- Never commit `.env` files or API keys to GitHub.
- Add `.env` to `.gitignore`.
- Use strong, randomly generated administrative API keys.
- Restrict API key access to the services that require them.
- Additional authentication and authorization controls are necessary before using the application with real enterprise data.

### Step 5: Ingest Sample Knowledge

```bash
python ingest_sample_kb.py
```

This loads the sample company documents and processes them through the ingestion pipeline.

```text
Company Documents
       |
       v
Document Loading
       |
       v
Text Chunking
       |
       v
Embedding Generation
       |
       v
Pinecone Indexing
       |
       v
Private Knowledge Base
```

### Step 6: Start the Application

```bash
python run.py
```

Open the application:

```text
http://127.0.0.1:8000
```

FastAPI interactive documentation:

```text
http://127.0.0.1:8000/docs
```

---

## 8. Demonstration Scenarios

The following scenarios demonstrate how the agent makes different retrieval and routing decisions.

### Scenario 1: Private Knowledge Base Retrieval

**Question**

> How do I connect to the company VPN from home?

**Expected Workflow**

```text
Router → KB
       |
       v
Private KB Retrieval
       |
       v
Evidence Grade → GOOD
       |
       v
Generate Answer from Private KB
```

**What this demonstrates**

The system recognizes that the answer exists in internal documentation and does not need an unnecessary external search.

### Scenario 2: Company Policy Question

**Question**

> Can IT support ask me to share my MFA code?

The sample company handbook specifies that employees must never share their passwords or MFA codes with support personnel.

**Expected Workflow**

```text
Router → KB
       |
       v
Private KB Retrieval
       |
       v
Evidence Grade → GOOD
       |
       v
Generate Private KB Answer
```

**What this demonstrates**

The application can retrieve and use company-specific information that is not part of the LLM's original training knowledge.

### Scenario 3: External Information Retrieval

**Question**

> What is the latest Microsoft Teams outage guidance?

**Expected Workflow**

```text
Router → KB
       |
       v
Private KB Retrieval
       |
       v
Evidence Grade → WEAK
       |
       v
Tavily Web Search
       |
       v
Web Evidence Grade → GOOD
       |
       v
Generate Web-Based Answer
```

**What this demonstrates**

The agent recognizes when internal knowledge is insufficient and switches to an external information source.

### Scenario 4: Query Rewriting

**Question**

> My work communication app is acting strange after the new update. What should I do?

If the initial retrieval attempts do not provide sufficient evidence, the workflow can rewrite the question and retry retrieval.

**What this demonstrates**

Instead of immediately failing when retrieval is unsuccessful, the agent attempts to improve the search query while respecting a retry limit.

### Scenario 5: Direct Conversation

**Question**

> Hello!

**Expected Workflow**

```text
Router → DIRECT
       |
       v
Direct Answer
```

**What this demonstrates**

Simple conversational messages bypass unnecessary vector retrieval and external search, helping avoid unnecessary API calls.

---

## 9. Document Ingestion

The application includes an **Add Company Document** feature in the sidebar.

Authorized administrators can upload new company documents using the administrative API key configured in `.env`.

### Supported Formats

- PDF (`.pdf`)
- Text (`.txt`)
- Markdown (`.md`)
- Word Documents (`.docx`)

### Ingestion Workflow

```text
Document Upload
       |
       v
File Type Validation
       |
       v
Document Loading
       |
       v
Recursive Text Chunking
       |
       v
Embedding Generation
       |
       v
Pinecone Indexing
       |
       v
Available for Retrieval
```

Newly ingested documents can be retrieved without manually modifying the Python source code.

This makes the application more practical for an enterprise environment where internal documentation changes over time.

---

## 10. API Endpoints

The application exposes its core functionality through FastAPI.

### `GET /api/health`

Checks the application health.

### `POST /api/chat`

Accepts an employee question and executes the Agentic RAG workflow.

**Request**

```json
{
    "question": "How do I reset my company password?"
}
```

**Example Response**

```json
{
    "answer": "Follow the company password reset procedure...",
    "source_used": "private_kb",
    "trace": [
        "Router → KB",
        "Private KB retrieval → 4 chunks",
        "KB evidence grade → GOOD",
        "Answer generation → PRIVATE KB"
    ],
    "citations": []
}
```

The response includes:

- `answer`: Generated response.
- `source_used`: Information source used by the workflow.
- `trace`: Decision path followed by the agent.
- `citations`: Citation information returned by the application.

### `POST /api/ingest`

Uploads a document and adds its processed chunks to Pinecone.

**Authentication**

Requires the `X-Admin-Key` request header.

---

## 11. Key Engineering Concepts Demonstrated

This project brings together several important AI engineering concepts:

| Concept | Implementation |
|---|---|
| Agentic Workflows | LangGraph stateful graph |
| Conditional Routing | Direct conversation vs. retrieval |
| Retrieval-Augmented Generation | Private document retrieval |
| Evidence Evaluation | LLM-based document grading |
| Hybrid Information Sources | Private KB + Tavily |
| Query Optimization | Query rewriting and retry |
| Vector Search | Pinecone |
| Document Processing | Loading and recursive chunking |
| API Development | FastAPI |
| Auditability | SQLite decision logging |
| Application Development | HTML, CSS, JavaScript |
| Environment Configuration | `.env` |

---

## 12. Future Improvements

Potential extensions for taking this application further:

- Enterprise authentication and role-based access control.
- Document-level permissions and tenant isolation.
- Conversation history and persistent user sessions.
- Retrieval caching to reduce repeated embedding and search costs.
- Reranking for improved retrieval relevance.
- More comprehensive evaluation of retrieval and answer quality.
- Observability, latency tracking, and token-cost monitoring.
- Production deployment with Docker and cloud infrastructure.

---

## 13. Project Goal

The objective of this project is to demonstrate how an AI engineer or Forward Deployed Engineer can move beyond isolated LLM experiments and build an application that addresses a practical business problem.

It combines agent orchestration, retrieval systems, external information access, backend APIs, document ingestion, and a user-facing interface into one cohesive engineering project.

**From an AI notebook to an application that solves a business problem.**