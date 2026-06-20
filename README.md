# RAG-Based Customer Support Assistant

A Retrieval-Augmented Generation (RAG) assistant that answers customer
support questions using a local knowledge base (PDF), grounds every answer
in retrieved context, and escalates to a human reviewer when it isn't
confident enough to answer on its own. The workflow is orchestrated with
**LangGraph** and runs locally via **Ollama**.

## How it works

The pipeline is a LangGraph state machine (`graph.py`) with these nodes:

```
START
  │
  ▼
process_query        # trims input, routes empty queries to "error"
  │
  ▼
classify_intent       # keyword check: complaint/refund/cancel/legal → "complex"
  │                    # short query (≤12 words) → "simple"
  ▼
retrieve_docs          # similarity search against ChromaDB (see retrieval.py)
  │
  ▼
decision_node          # routing logic (see below)
  ├──▶ generate_answer  # LLM answers from retrieved context only
  │        └─▶ hitl      (if the LLM call fails)
  ├──▶ hitl              # escalate to a human reviewer
  └──▶ error             # empty-query short-circuit
  │
  ▼
END
```

**Routing logic** in `decision_node`:
| Condition | Route |
|---|---|
| Input was empty | `error` |
| No documents retrieved | `hitl` |
| Best Chroma distance > `CONFIDENCE_THRESHOLD` (default `0.85`) | `hitl` |
| Intent is `complex` or `out_of_scope` | `hitl` |
| Otherwise | `generate_answer` |

> Note: Chroma returns a **distance** (lower = more similar), so the
> threshold check is "escalate if the best match is *too far*," not a
> similarity score in the usual 0–1-higher-is-better sense.

### Retrieval (`retrieval.py`)
- Loads the persisted Chroma collection (`support_docs`) from `chroma_db/`.
- Expands the query with a small synonym map (e.g. `delivery` →
  `shipping`, `delivery time`) to improve recall, runs
  `similarity_search_with_score` for each expansion, then deduplicates and
  keeps the top `TOP_K` results by distance.

### Generation (`graph.py: generate_answer`)
- Uses `ChatOllama` (model from `OLLAMA_MODEL`, default `llama3.1`) with a
  strict prompt: answer **only** from retrieved context, and reply with
  *"I don't have enough information."* if the context doesn't support an
  answer. If the LLM call throws, the state is routed to `hitl` instead of
  failing.

### Human-in-the-loop (`hitl.py`)
- `request_human_support()` prints the query, escalation reason, sources,
  and any draft answer, then blocks on `input()` for a human-approved
  response from the terminal. This is a CLI simulation of HITL, not a
  ticketing/queue integration.

### Ingestion (`ingestion.py`)
- Finds all PDFs in `data/` (currently `support_advanced.pdf`), loads them
  with `PyPDFLoader`, splits with `RecursiveCharacterTextSplitter`
  (`CHUNK_SIZE`/`CHUNK_OVERLAP` from config), embeds with
  `OllamaEmbeddings` (`OLLAMA_EMBEDDING_MODEL`, default `nomic-embed-text`),
  and writes/persists everything into the local Chroma store at
  `CHROMA_DIR` (default `chroma_db/`).
- `main.py` calls this automatically on first run if `chroma_db/` doesn't
  exist yet, so dropping a new PDF into `data/` and deleting `chroma_db/`
  is the easiest way to rebuild the knowledge base.

### CLI entrypoint (`main.py`)
- Ensures the knowledge base exists, builds the graph, and runs a simple
  `Customer Query: ` chat loop until you type `exit`/`quit`.

## Project structure

```
.
├── main.py                              # CLI entrypoint / chat loop
├── graph.py                             # LangGraph workflow definition
├── retrieval.py                         # Chroma loading + similarity search
├── ingestion.py                         # PDF → chunks → embeddings → Chroma
├── hitl.py                              # CLI-based human escalation
├── config.py                            # Env-driven AppConfig (dataclass)
├── requirements.txt
├── .env / .env.example                  # Configuration (see below)
├── data/
│   └── support_advanced.pdf             # Source knowledge base document
├── chroma_db/                           # Persisted Chroma vector store
│   ├── chroma.sqlite3
│   └── <collection-uuid>/               # HNSW index files
├── HLD_RAG_Customer_Support_Assistant.docx
├── RAG_CustomerSupport_LLD.docx
└── RAG_Technical_Documentation.docx
```

## Setup

```bash
python -m venv .venv
.venv\Scripts\activate        # Windows
# source .venv/bin/activate   # macOS/Linux

pip install -r requirements.txt
```

You'll also need [Ollama](https://ollama.com) running locally with the
models pulled:

```bash
ollama pull llama3.1
ollama pull nomic-embed-text
```

### Configuration (`.env`)

`config.py` reads these variables (defaults shown):

| Variable | Default | Purpose |
|---|---|---|
| `DATA_DIR` | `data` | Folder scanned for PDFs to ingest |
| `CHROMA_DIR` | `chroma_db` | Persisted vector store location |
| `CHROMA_COLLECTION` | `support_docs` | Chroma collection name |
| `CHUNK_SIZE` | `1000` | Characters per chunk |
| `CHUNK_OVERLAP` | `300` | Overlap between chunks |
| `TOP_K` | `4` | Chunks retrieved per query |
| `CONFIDENCE_THRESHOLD` | `0.85` | Max acceptable best-match distance before escalating |
| `OLLAMA_MODEL` | `llama3.1` | Chat model for answer generation |
| `OLLAMA_EMBEDDING_MODEL` | `nomic-embed-text` | Embedding model for ingestion + retrieval |

> ⚠️ The current `.env` has `LLM_PROVIDER=openai`, `OPENAI_API_KEY`, and
> `OPENAI_MODEL` set, but **the code never reads `LLM_PROVIDER` or any
> OpenAI variable** — `ingestion.py` and `graph.py` always use
> `OllamaEmbeddings`/`ChatOllama` regardless of what `.env` says. Either
> those OpenAI lines are vestigial and can be removed, or OpenAI support
> needs to actually be wired into `config.py`/`ingestion.py`/`graph.py` if
> that's the intent.

### Run

```bash
python main.py
```

On first run, if no `chroma_db/` exists, it ingests every PDF in `data/`
automatically. Then type questions at the `Customer Query:` prompt; type
`exit` to quit.

## Documentation

This repo includes three supplementary design documents:
- `HLD_RAG_Customer_Support_Assistant.docx` — high-level design
- `RAG_CustomerSupport_LLD.docx` — low-level design
- `RAG_Technical_Documentation.docx` — technical documentation

## Known gaps / things to reconcile

- **The previous README described a different system** than what's
  implemented: it mentions OpenAI `text-embedding-3-small` embeddings,
  MMR + cross-encoder reranking, GPT-4o-mini generation, and a
  confidence-≥-0.72 auto-answer threshold. None of that is in the current
  code — the actual implementation uses Ollama for both embedding and
  generation, simple query expansion (no MMR), no reranker, and a
  distance threshold of `0.85`. This rewritten README reflects the code as
  it stands; the architecture diagram/feature list above should replace
  the old one rather than coexist with it.
- **`.env` provider mismatch** — see the config table note above.
- **HITL is CLI-only** — `hitl.py` blocks on a terminal `input()` call,
  so it only works in an interactive session, not a server/API context.
- **`.gitignore` excludes `*.bin` and `*.sqlite3`** (i.e. the whole Chroma
  store), which is reasonable for git hygiene, but means a fresh clone
  must re-run ingestion before `main.py` can answer anything.
- **Intent classification is simple keyword matching** (`complaint`,
  `refund`, `chargeback`, `cancel`, `legal`, `escalate`, plus a word-count
  heuristic) rather than an LLM- or ML-based classifier — fine for a
  prototype, but worth flagging as a known simplification.

## Tech stack

- **Orchestration:** LangGraph
- **LLM + embeddings:** Ollama (`llama3.1`, `nomic-embed-text`)
- **Vector store:** ChromaDB (persistent, local)
- **Document loading/splitting:** LangChain (`PyPDFLoader`,
  `RecursiveCharacterTextSplitter`)
- **Config:** `python-dotenv` + a dataclass-based `AppConfig`
