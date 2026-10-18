# Enterprise RAG Pipeline: Corporate Policy Engine

A local Retrieval-Augmented Generation (RAG) reference pipeline for ingesting, chunking and querying corporate documents using the modern LangChain Expression Language (LCEL) architecture.

---

## 📈 Evolution: From MVP to a testable reference pipeline

This repository documents the **progressive enhancement** of a basic RAG proof-of-concept into a reference pipeline. Each commit addresses a specific architectural gap in the original implementation, demonstrating senior-level engineering practices across **security**, **cost optimization**, **resilience**, **testability**, and **responsible AI**.

| # | Commit | Senior Engineering Signal |
|---|--------|--------------------------|
| 1 | `security: audit secrets, harden gitignore, and add defense-in-depth guardrails` | **DevSecOps** — 4-layer defense (gitignore → `.env.example` → `SECURITY.md` → runtime validation). API key format verification before any billable call. OWASP fail-securely pattern. |
| 2 | `refactor: extract configuration into centralized dataclass with cost-aware defaults` | **12-Factor App** — every parameter documented with cost/performance/compliance tradeoffs. Frozen dataclasses prevent runtime mutation. Single source of truth for all tuning. |
| 3 | `feat: support multi-format ingestion with extensible loader registry` | **SOLID Design** — Open/Closed Principle via loader registry. The loader registry is an extension point; additional formats require implementation and testing. File validation before any API call (cost awareness). |
| 4 | `feat: idempotent incremental ingestion with content-addressed deduplication` | **Data Engineering** — SHA-256 content addressing (same pattern as Git/Docker/IPFS). Unchanged-content detection aims to avoid redundant embedding calls. `--force-reindex` flag for intentional rebuilds. |
| 5 | `feat: source-grounded responses with relevance scoring and citation metadata` | **Source metadata** — retrieval scores and source metadata are exposed; prompt instructions request citations. This does not establish citation support for every generated claim. |
| 6 | `feat: interactive REPL with streaming output and MMR diversity retrieval` | **UX Maturity** — token-by-token streaming; latency requires measurement. MMR retrieval prevents redundant context. Timing instrumentation for cost monitoring. |
| 7 | `feat: resilient API calls with exponential backoff, rate-limit handling, and structured logging` | **Resilience Engineering** — exponential backoff with jitter thwarts thundering herd. Discriminates between retryable (429/5xx) and fatal (401/400/404) errors. |
| 8 | Mocked external API test suite | **Testing Strategy** — tests target guardrails, retry logic, chunk ID determinism and format validation. This wording sweep does not reproduce a test count or timing. |

### Original MVP Limitations

The initial implementation (`5709b2c` — `1a664b8`) was a functional proof-of-concept with significant technical debt:

- ❌ Hardcoded API keys and file paths
- ❌ Single `.txt` file support only
- ❌ Destructive re-ingestion (delete + rebuild every run)
- ❌ No error handling for API failures
- ❌ Black-box answers with no source attribution
- ❌ Batch-only output (no streaming)
- ❌ Zero test coverage

### Implemented reference features

After 8 incremental commits:

- ✅ Defense-in-depth security with 4 control layers
- ✅ 12-factor configuration with documented cost tradeoffs
- ✅ Multi-format ingestion (.txt, .md, .pdf) via extensible registry
- ✅ Idempotent re-ingestion with content-addressed deduplication
- ✅ Source-cited answers with relevance scores and verbose debug mode
- ✅ Interactive REPL with streaming, MMR diversity, and timing instrumentation
- ✅ Exponential backoff retry with error classification
- ✅ External API mocking in the test suite; see its commands for current results

---



## 🧠 Architecture Overview

This system allows users to ask complex, natural language questions about internal company policies. The prompt asks the model to use retrieved context and decline unsupported questions. This instruction does not guarantee grounded answers or correct abstention; those behaviors need evaluation.

### Core Stack
* **Orchestration:** LangChain (v0.2+ LCEL Framework)
* **Vector Database:** ChromaDB (Persistent Local Storage)
* **Embeddings:** OpenAI `text-embedding-3-small` (1,536 dimensions)
* **LLM:** OpenAI `gpt-4o-mini` (configured default; cost and suitability require dated evaluation)
* **Resilience:** `tenacity` (exponential backoff with jitter)
* **Testing:** `pytest` with external API mocks; no Actions run returned for `main`
* **Environment:** Python 3.11+

## ✨ Key Features

* **Automated Ingestion (`ingest.py`):** Intelligently chunks text documents using `RecursiveCharacterTextSplitter` to preserve paragraph and sentence context boundaries.
* **Semantic Retrieval (`search.py`):** Bypasses keyword-matching in favor of geometric cosine-similarity search to find the underlying meaning of user queries.
* **LCEL composition:** Uses LangChain pipe routing (`|`) to compose retrieval and generation stages.
* **Input and error handling:** Includes checks for missing databases, empty collections, and missing environment variables to prevent runtime crashes.

## 🚀 Getting Started

### 1. Clone & Environment Setup

```powershell
git clone https://github.com/koatedevopskpai/gem-langchain-rag.git
cd gem-langchain-rag

# Create and activate a virtual environment

python -m venv venv
.\venv\Scripts\activate

# Install dependencies
pip install langchain langchain-openai langchain-chroma chromadb python-dotenv
```

### 2. Environment Variables
Create a .env file in the root directory and add your OpenAI API key:
```
OPENAI_API_KEY="sk-your-api-key-here"
```
### 3. Usage
#### Step A: Populate the Vector Database
Place your source documents in the data/ folder (a sample policy.txt is provided). Run the ingestion script to chunk the text and generate embeddings:

```powerShell
python ingest.py
```
#### Step B: Query the Engine
Once the database is populated, execute the search script to run the LCEL pipeline:

```powerShell
python search.py
```
### 📂 Project Structure
```
📦 gem-langchain-rag
 ┣ 📂 data/                          # Source documents (.txt, .md, .pdf)
 ┃ ┗ 📜 policy.txt                   # Sample corporate policy
 ┣ 📂 chroma_langchain_db/           # Local vector database (Git-ignored)
 ┣ 📂 tests/                         # External API mocks and local unit checks
 ┃ ┣ 📜 conftest.py                  # Shared fixtures & OpenAI mocks
 ┃ ┣ 📜 test_config.py               # Configuration validation
 ┃ ┣ 📜 test_ingest.py               # Ingestion pipeline
 ┃ ┣ 📜 test_loaders.py              # Loader registry
 ┃ ┣ 📜 test_retry.py                # Resilience patterns
 ┃ ┗ 📜 test_search.py               # Retrieval pipeline
 ┣ 📜 config.py                      # 12-factor config dataclass
 ┣ 📜 .env.example                   # Environment template with security notes
 ┣ 📜 SECURITY.md                    # Security policy & incident response
 ┣ 📜 ingest.py                      # Document loading & embedding pipeline
 ┣ 📜 loaders.py                     # Extensible loader registry (.txt, .md, .pdf)
 ┣ 📜 search.py                      # LCEL retrieval & generation pipeline
 ┣ 📜 retry.py                       # Exponential backoff & error classification
 ┣ 📜 .env                           # API keys (Git-ignored)
 ┣ 📜 .gitignore                     # 28-entry hardened ignore patterns
 ┗ 📜 README.md                      # Project documentation
```


## Claim review — 18 October 2026

See [verification and limits](evidence/W5-05.md) for the source revision, claim
classification and observed CI status. Wording corrections do not establish a
new deployment, passing test suite or production operation.
