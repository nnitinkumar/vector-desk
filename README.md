# VectorDesk

VectorDesk is a FastAPI service for document analysis, comparison, and retrieval-augmented chat. Upload PDFs (or DOCX/TXT) to get an LLM-generated structural analysis, diff two documents against each other, or build a FAISS-backed vector index and chat with your documents.



## Features

- **Document analysis** (`POST /analyze`) — upload a PDF and get back a structured LLM analysis of its contents.
- **Document comparison** (`POST /compare`) — diff a reference document against an actual document and get a row-by-row comparison table.
- **Conversational RAG chat** (`POST /chat/index`, `POST /chat/query`) — index one or more documents into a per-session FAISS store, then ask questions against them via an LCEL retrieval chain.
- **Pluggable LLM/embedding providers** — configured in [`config/config.yaml`](config/config.yaml), currently Groq and Google Gemini for the LLM, Google `text-embedding-004` for embeddings.
- **Structured logging** via `structlog` ([`logger/`](logger/)).

## Architecture

```
api/main.py                    FastAPI app and route handlers
src/document_analyzer/         Single-document LLM analysis
src/document_compare/          Two-document comparison (LLM-driven diff)
src/document_ingestion/        PDF/DOCX loading, chunking, FAISS index build/load
src/document_chat/             Conversational retrieval chain (LCEL)
utils/model_loader.py          LLM + embedding provider loading, API key management
config/config.yaml             Provider, model, and retriever settings
prompt/prompt_library.py       Prompt templates used by the LLM chains
templates/, static/            Minimal HTML/CSS UI served at `/`
```

## Setup

### Prerequisites

- Python 3.10+
- API keys for at least one LLM provider (Groq and/or Google) and Google for embeddings

### Install

```bash
# Clone the repository
git clone https://github.com/<your-username>/vector-desk.git
cd vector-desk

# Create and activate a virtual environment
conda create -p .venv python=3.10 -y
conda activate ./.venv

# Install dependencies (also installs this package in editable mode)
pip install -r requirements.txt
```

### Configure environment variables

Create a `.env` file in the project root:

```bash
GROQ_API_KEY=your_groq_key
GOOGLE_API_KEY=your_google_key
LLM_PROVIDER=google        # or "groq" — must match a key under `llm:` in config/config.yaml
ENV=local                  # "production" skips loading .env and expects real env vars / secrets
```

`API_KEYS` may alternatively be set to a single JSON object (e.g. from an ECS secret) containing `GROQ_API_KEY` and `GOOGLE_API_KEY` — see [`utils/model_loader.py`](utils/model_loader.py).

### Run

```bash
uvicorn api.main:app --host 0.0.0.0 --port 8080 --reload
```

Then open `http://localhost:8080` for the UI, or call the API directly (see below).

## API

| Endpoint | Method | Description |
|---|---|---|
| `/health` | GET | Health check |
| `/analyze` | POST | Upload a single PDF for structural analysis |
| `/compare` | POST | Upload `reference` and `actual` PDFs to compare |
| `/chat/index` | POST | Upload one or more files to build a session's FAISS index |
| `/chat/query` | POST | Ask a question against a previously indexed session |

## Minimum Requirements

### LLM Providers
- **Groq** (Free tier)
- **Google Gemini** (Free trial)

### Embedding Models
- **Google** (`text-embedding-004`, used by default)

### Vector Store
- **FAISS** (local, on-disk index under `faiss_index/`)

## API Keys

### GROQ API Key
- [Get your API Key](https://console.groq.com/keys)
- [Groq Documentation](https://console.groq.com/docs/overview)

### Google (Gemini) API Key
- [Get your API Key](https://aistudio.google.com/apikey)
- [Gemini Documentation](https://ai.google.dev/gemini-api/docs/models)

## Testing

```bash
pytest
```

## Roadmap

- Persistent chat history across requests
- Source citations returned alongside chat answers
- Hosted vector store (Pinecone/Weaviate/pgvector) as a FAISS alternative
- N-way document comparison
- OCR fallback for scanned/image-only PDFs
- Retrieval/faithfulness evaluation harness
- Auth (API key or JWT) and multi-tenancy
- Streaming (SSE) chat responses
- OpenAI as an additional LLM provider
- LangSmith tracing / cost & latency observability
