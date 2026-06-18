# 🤖 RAG Customer Support System

An enterprise-grade Retrieval-Augmented Generation (RAG) Customer Support Assistant that combines Hybrid Retrieval, Semantic Search, BM25 Keyword Matching, Reciprocal Rank Fusion (RRF), and Cross-Encoder Reranking to deliver accurate, context-aware responses from organizational knowledge bases.

The system leverages Large Language Models (LLMs) and advanced retrieval techniques to reduce hallucinations, improve answer relevance, and provide reliable customer support experiences. It also incorporates Human-in-the-Loop (HITL) escalation for handling low-confidence and sensitive customer queries.

---

# 🚀 Features

## 📄 Document Processing

* PDF document ingestion
* Automated text extraction
* Recursive text chunking
* Context-preserving chunk overlap
* Embedding generation

## 🔍 Hybrid Retrieval Pipeline

* Semantic Search using vector embeddings
* BM25 keyword retrieval
* Reciprocal Rank Fusion (RRF)
* Cross-Encoder document reranking
* Query expansion for improved recall

## 🧠 Intelligent Response Generation

* Context-aware answer generation
* Retrieval-grounded responses
* Hallucination mitigation
* Confidence-based decision making

## ⚙️ Workflow Orchestration

* LangGraph-based workflow management
* Query classification
* Retrieval pipeline routing
* Human-in-the-Loop escalation

## 👨‍💻 Human-in-the-Loop Support

* Low-confidence query detection
* Sensitive issue escalation
* Manual response intervention
* Enterprise support workflow simulation

---

# 🏗️ System Architecture

## Knowledge Base Creation

```text
Enterprise Documents (PDFs)
           │
           ▼
     Document Loader
           │
           ▼
      Text Chunking
   (1000 chars, overlap)
           │
           ▼
 Embedding Generation
 (nomic-embed-text)
           │
           ▼
    Vector Database
      (ChromaDB)
```

---

## Query Processing Pipeline

```text
Customer Query
       │
       ▼
Intent Classification
       │
       ▼
Query Expansion
       │
       ▼
 ┌──────────────┬──────────────┐
 │              │
 ▼              ▼
BM25      Semantic Search
Retrieval   Retrieval
 │              │
 └──────┬───────┘
        ▼
 Reciprocal Rank Fusion
        ▼
Cross Encoder Reranking
        ▼
 Top Ranked Documents
        ▼
 Confidence Scoring
        ▼
 ┌──────────────┐
 │ Decision Node│
 └──────┬───────┘
        │
   ┌────┴────┐
   ▼         ▼
LLM      Human Agent
Answer   Escalation
   │         │
   └────┬────┘
        ▼
  Final Response
```

---

# 🧩 Core Components

## Semantic Search

Converts user queries and document chunks into dense vector embeddings and retrieves semantically similar content using vector similarity search.

### Benefits

* Understands contextual meaning
* Handles paraphrased queries
* Improves retrieval quality

Example:

Query:

```text
Can I get my money back?
```

Document:

```text
Refunds are allowed within 30 days.
```

Semantic retrieval successfully identifies the relationship between "money back" and "refund."

---

## BM25 Retrieval

Implements sparse keyword-based retrieval using BM25 ranking.

### Benefits

* Excellent for exact keyword matching
* Handles product IDs, error codes, and technical terms
* Complements semantic retrieval

Example:

```text
Error Code E1023
Invoice ID INV-204
SKU ABC123
```

---

## Hybrid Retrieval

Runs BM25 and Semantic Search simultaneously.

```text
Query
 │
 ├── BM25 Retrieval
 │
 └── Semantic Retrieval
```

This improves recall and retrieval robustness.

---

## Reciprocal Rank Fusion (RRF)

Combines rankings from multiple retrieval systems into a single unified ranking.

### Advantages

* Balances keyword and semantic retrieval
* Improves retrieval stability
* Reduces ranking bias

Formula:

```text
RRF Score = Σ 1 / (k + rank)
```

---

## Cross Encoder Reranking

Uses a transformer-based Cross Encoder model to evaluate query-document relevance.

Model:

```text
cross-encoder/ms-marco-MiniLM-L-6-v2
```

### Purpose

After retrieval, candidate documents are reranked based on actual relevance.

Benefits:

* Higher precision
* Better context selection
* Improved final answers

---

## Confidence Scoring

Calculates confidence using:

* Retrieval relevance
* Cross Encoder score
* Ranking consistency

Routing Strategy:

| Confidence Score | Action                 |
| ---------------- | ---------------------- |
| > 0.80           | Generate AI Response   |
| 0.50 – 0.80      | Low Confidence Warning |
| < 0.50           | Human Escalation       |

---

## Human-in-the-Loop (HITL)

Queries are escalated when:

* Confidence is low
* Legal concerns arise
* Refund disputes occur
* Sensitive customer issues are detected

This improves reliability and mirrors real-world enterprise support systems.

---

# 🛠️ Tech Stack

### Programming Language

* Python

### AI & LLM

* Llama 3.1
* Ollama
* Generative AI
* Prompt Engineering

### Retrieval & Search

* ChromaDB
* FAISS
* BM25
* Semantic Search
* Reciprocal Rank Fusion (RRF)
* Cross Encoder Reranking

### Frameworks

* LangChain
* LangGraph
* Sentence Transformers

### NLP

* Embeddings
* Query Expansion
* Text Chunking
* Information Retrieval

---

# 📊 Project Workflow

### Phase 1: Knowledge Base Creation

1. Load PDF documents
2. Extract text
3. Split into chunks
4. Generate embeddings
5. Store vectors in ChromaDB

### Phase 2: Query Processing

1. Receive customer query
2. Classify intent
3. Expand query
4. Execute hybrid retrieval
5. Fuse rankings using RRF
6. Rerank using Cross Encoder
7. Calculate confidence score
8. Generate answer or escalate

---

# 🎯 Applications

* Customer Support Automation
* Enterprise Knowledge Assistants
* Internal Helpdesk Systems
* FAQ Automation
* Technical Documentation Search
* AI-Powered Service Desks

---

# 📚 Skills Demonstrated

* Retrieval-Augmented Generation (RAG)
* Large Language Models (LLMs)
* Hybrid Information Retrieval
* Semantic Search
* Vector Databases
* Prompt Engineering
* LangGraph Workflow Design
* Human-in-the-Loop Systems
* NLP Pipelines
* AI Application Development

---

# 👩‍💻 Author

**Roshini Mutyala**

B.Tech – Computer Science and Engineering (Artificial Intelligence)

Passionate about Generative AI, Retrieval-Augmented Generation, Intelligent Systems, and AI-Powered Software Development.
