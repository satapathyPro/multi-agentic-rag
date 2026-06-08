# Multi-Agentic RAG System

> Self-correcting multi-agent Retrieval-Augmented Generation system — hybrid retrieval with web search fallback, LangChain orchestration, Groq LLMs, and built-in fact-checking.

![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?style=flat-square&logo=python&logoColor=white)
![LangChain](https://img.shields.io/badge/LangChain-0.3%2B-1C3C3C?style=flat-square&logo=chainlink&logoColor=white)
![Groq](https://img.shields.io/badge/Groq-LLM-F55036?style=flat-square&logo=groq&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-1.x-FF4B4B?style=flat-square&logo=streamlit&logoColor=white)
![ChromaDB](https://img.shields.io/badge/ChromaDB-Vector_Store-00B4D8?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

---

## Overview

This project implements a production-grade, **self-correcting multi-agent RAG pipeline** that intelligently routes queries across a network of specialized agents. Instead of a single monolithic retrieval step, the system chains dedicated agents for routing, retrieval, query reformulation, web search, synthesis, answer generation, fact-checking, and safety validation — producing high-quality, grounded answers even when the internal knowledge base falls short.

---

## Agent Architecture

```
User Query
    │
    ▼
┌─────────┐
│  Router │  ──── Classifies query intent and selects the optimal path
└────┬────┘
     │
     ├──────────────────────┬──────────────────────┐
     ▼                      ▼                      ▼
┌──────────┐        ┌──────────────┐        ┌───────────┐
│Retriever │        │ WebSearcher  │        │Clarifier  │
│(Internal │        │  (Tavily)    │        │(Ask User) │
│  KB)     │        └──────┬───────┘        └─────┬─────┘
└────┬─────┘               │                      │
     │                     │                      │
     ▼                     │                      ▼
┌─────────┐                │                   (End)
│  Grader │  ──── Relevance score
└────┬────┘
     │
     ├── relevant ──────────────────────────────┐
     │                                          │
     ├── reformulate ──► Reformulator ──► Retriever
     │
     └── fallback ──────► WebSearcher
                               │
                               ▼
                        ┌────────────┐
                        │Synthesizer │  ──── Merges retrieved + searched context
                        └─────┬──────┘
                              │
                              ▼
                        ┌───────────┐
                        │ Generator │  ──── Drafts the final answer
                        └─────┬─────┘
                              │
                              ▼
                        ┌─────────────┐
                        │ FactChecker │  ──── Verifies factual claims via web
                        └─────┬───────┘
                              │
                              ▼
                        ┌───────────────┐
                        │ SafetyChecker │  ──── Detects / revises harmful content
                        └─────┬─────────┘
                              │
                              ▼
                          Answer ✓
```

### Agent Roles

| Agent | Responsibility |
|---|---|
| **Router** | Classifies intent and selects the best workflow path |
| **Retriever** | Fetches relevant chunks from the internal vector store |
| **Grader** | Scores retrieved documents for relevance; triggers fallback if needed |
| **Reformulator** | Rewrites the query for improved retrieval before retrying |
| **WebSearcher** | Pulls real-time external information via Tavily API |
| **Synthesizer** | Merges and de-duplicates context from multiple sources |
| **Generator** | Produces the final natural-language answer |
| **FactChecker** | Extracts factual claims and verifies them against live web results |
| **SafetyChecker** | Screens output for harmful or inappropriate content; revises or blocks |
| **Clarifier** | Asks the user a follow-up question when the query is ambiguous |

---

## Key Features

- **Multi-agent orchestration** — LangGraph-powered directed agent graph with conditional edges and retry loops
- **Hybrid knowledge sources** — Ingest PDFs, DOCX, TXT files, and URLs into a ChromaDB vector store; falls back to live web search automatically
- **Self-correcting pipeline** — Query reformulation and relevance grading before escalating to web search
- **Integrated fact-checking** — Extracted claims are verified in real time via Tavily search
- **Safety layer** — Every generated response passes through a safety filter before reaching the user
- **Interactive Streamlit UI** — Chat interface with live workflow logs, execution trace, and configurable sidebar
- **Configurable parameters** — Chunk size, retriever top-K, and LLM temperature adjustable at runtime

---

## Tech Stack

| Layer | Technology |
|---|---|
| LLM Provider | Groq (`llama3-70b-8192`, `mixtral-8x7b`) |
| Orchestration | LangChain + LangGraph |
| Vector Store | ChromaDB |
| Web Search | Tavily API |
| Embeddings | Google Generative AI Embeddings |
| Frontend | Streamlit |
| Language | Python 3.9+ |

---

## Installation & Setup

### 1. Clone the Repository

```bash
git clone https://github.com/satapathyPro/multi-agentic-rag.git
cd multi-agentic-rag
```

### 2. Create a Virtual Environment

```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

> Requires Python 3.9+

### 4. Configure API Keys

Create `.streamlit/secrets.toml` in the project root:

```toml
GROQ_API_KEY        = "your_groq_api_key"
TAVILY_API_KEY      = "your_tavily_api_key"
GOOGLE_API_KEY      = "your_google_api_key"
LANGCHAIN_API_KEY   = "your_langsmith_api_key"   # optional, for tracing
```

| Key | Where to get it |
|---|---|
| `GROQ_API_KEY` | [console.groq.com](https://console.groq.com/) |
| `TAVILY_API_KEY` | [app.tavily.com](https://app.tavily.com/) |
| `GOOGLE_API_KEY` | [ai.google.dev](https://ai.google.dev/) |
| `LANGCHAIN_API_KEY` | [smith.langchain.com](https://smith.langchain.com/) |

### 5. Run the App

```bash
streamlit run app.py
```

Open [http://localhost:8501](http://localhost:8501) in your browser.

---

## Usage

1. **Load knowledge sources** — In the sidebar, paste URLs (one per line) and/or upload files (PDF, DOCX, TXT). Click **"Apply Parameters and Update Knowledge"**.

2. **Ask a question** — Type your query in the chat input. The system routes it through the agent graph and streams back a grounded, fact-checked answer.

3. **Inspect the trace** — Expand **"Execution Details"** beneath each response to see which agents ran, relevance grades, reformulation attempts, and safety decisions.

4. **Tune parameters** — Adjust chunk size, retriever K, and LLM temperature from the sidebar at any time; click Apply to rebuild the index.

5. **Reset** — Use **"Clear Chat History"** to start a fresh session.

---

## Project Structure

```
multi-agentic-rag/
├── app.py                  # Streamlit entry point
├── agents/
│   ├── router.py           # Query intent classifier
│   ├── retriever.py        # Vector-store retrieval agent
│   ├── grader.py           # Relevance grader
│   ├── reformulator.py     # Query rewriter
│   ├── web_searcher.py     # Tavily-backed web search agent
│   ├── synthesizer.py      # Context merger
│   ├── generator.py        # Answer generator
│   ├── fact_checker.py     # Claim verifier
│   ├── safety_checker.py   # Safety filter
│   └── clarifier.py        # Disambiguation agent
├── graph/
│   └── workflow.py         # LangGraph agent graph definition
├── knowledge/
│   └── loader.py           # Document ingestion and chunking
├── requirements.txt
└── .streamlit/
    └── secrets.toml        # API keys (not committed)
```

---

## Customization

- **Swap LLMs** — Change the model name in any agent file to use a different Groq model (e.g., `llama-3.1-8b-instant` for speed).
- **Replace the vector store** — Swap ChromaDB for Pinecone, FAISS, or Weaviate by updating `knowledge/loader.py`.
- **Add agents** — Insert new nodes into `graph/workflow.py` and wire conditional edges as needed.
- **Change the search provider** — Replace Tavily with SerpAPI, Bing, or DuckDuckGo by modifying `agents/web_searcher.py`.

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

## About the Developer

**Subham Satapathy** — Software Engineer with 6+ years building cloud-scale distributed systems and production-grade AI automation.

- GitHub: [satapathyPro](https://github.com/satapathyPro)
- LinkedIn: [subhamumd](https://www.linkedin.com/in/subhamumd/)
- Email: satapathypro@gmail.com

> This project reflects an interest in LLM-orchestrated workflows, system observability, and robust self-correcting agentic pipelines.
