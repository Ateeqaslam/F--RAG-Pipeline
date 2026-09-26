# F--RAG-Pipeline
A lightweight RAG pipeline: PDF ingestion, PII redaction, chunking, ChromaDB vector search, and Gemini-powered Q&amp;A with retry handling. This is was a learning/practice project.
F--RAG-Pipeline (First Retrieval-Augmented Generation Pipeline)


# RAG Pipeline

A lightweight, end-to-end **Retrieval-Augmented Generation (RAG)** pipeline that lets you ask questions about a PDF document and get answers grounded strictly in that document's content — powered by **ChromaDB** for vector search and **Google Gemini** for embeddings and generation.

---

## How It Works

```
PDF File
   │
   ▼
Text Extraction (pypdf)
   │
   ▼
PII Redaction (regex: emails, phone numbers, card numbers)
   │
   ▼
Chunking (LangChain RecursiveCharacterTextSplitter)
   │
   ▼
Embedding + Storage (ChromaDB + Gemini embedding model)
   │
   ▼
User Query ──► Input Validation ──► PII Redaction ──► Similarity Search
   │
   ▼
Context-Grounded Answer (Gemini, with retry on failure)
```

The model is explicitly instructed to answer **only** from retrieved context, and to say so when an answer isn't present in the document — this reduces hallucinated responses.

---

## Tech Stack & Why

| Component | Tool | Why this choice |
|---|---|---|
| PDF parsing | [`pypdf`](https://pypi.org/project/pypdf/) | Pure-Python, no external binary dependencies, sufficient for text-based PDFs. |
| Chunking | [`langchain-text-splitters`](https://pypi.org/project/langchain-text-splitters/) | `RecursiveCharacterTextSplitter` splits on natural boundaries (paragraphs → sentences → words) instead of blindly cutting at a fixed character count, which keeps chunks more semantically coherent. |
| Vector store | [`ChromaDB`](https://www.trychroma.com/) | Runs in-memory/embedded with zero infrastructure setup — ideal for a single-document pipeline without standing up a separate vector database service. |
| Embeddings + generation | **Google Gemini API** | One provider for both embeddings and generation keeps the pipeline simple and avoids managing multiple API keys/SDKs. |
| Retry handling | [`tenacity`](https://tenacity.readthedocs.io/) | Wraps the model call with exponential backoff so transient API errors (rate limits, timeouts) don't crash the whole pipeline. |
| PII redaction | `re` (standard library) | A lightweight first line of defense — sensitive data (emails, phone numbers, card numbers) is stripped **before** it's embedded, stored, or sent to the model. |

---

## Getting Started

### Prerequisites

- Python 3.9+
- A [Google Gemini API key](https://ai.google.dev/)

### Installation

```bash
pip install chromadb pypdf langchain-text-splitters tenacity google-genai
```

### Configuration

Set your Gemini API key as an environment variable:

```bash
export GEMINI_API_KEY="your-api-key-here"
```

> **Note:** The original notebook was built for Google Colab and reads the key via `google.colab.userdata`. For local or non-Colab use, replace that line with `os.environ["GEMINI_API_KEY"]` set directly, or load it from a `.env` file.

### Usage

```python
User_PDF = "your_document.pdf"
User_prompt = "What does this document say about X?"

text = extract_text_from_pdf(User_PDF)
redacted_text = redact_sen_data(text)
chunks = text_splitter(redacted_text)

# ... embed, store, query, and generate (see notebook for full flow)
```

Run the notebook cells in order: install dependencies → configure API key → run the pipeline.

---

## Security & Privacy Notes

- **PII redaction** runs on both the source document and the user's query before anything reaches the vector store or the model API — reducing (not eliminating) the risk of sensitive data leaving your machine.
- **Input validation** uses a keyword blocklist as a basic guard against obviously malicious prompts. This is a lightweight first step, not a complete defense — keyword matching can be bypassed and isn't a substitute for proper prompt-injection or content-moderation tooling in a production setting.
- Regex-based redaction (email/phone/card patterns) is best-effort and may produce false positives/negatives depending on document formatting.

---

## Known Limitations

- Re-running ingestion on the same PDF will attempt to re-insert chunks with the same IDs, which can conflict with existing collection entries.
- Chunk size (200 characters) is small and may fragment context on longer, denser documents — tune `chunk_size`/`chunk_overlap` for your use case.
- Currently handles a single PDF and a single query per run rather than a persistent, multi-document, multi-turn session.

---

## Forking & Contributing
 
Contributions, suggestions, and forks are welcome. Here's how to get set up:
 
### Forking the repo
 
1. Click **Fork** in the top-right corner of this repository on GitHub.
2. Clone your fork locally:
```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   cd <repo-name>
```
3. Add the original repo as an upstream remote (so you can pull in future updates):
```bash
   git remote add upstream https://github.com/<original-owner>/<repo-name>.git
```
 
### Setting up your dev environment
 
```bash
pip install chromadb pypdf langchain-text-splitters tenacity google-genai
export GEMINI_API_KEY="your-api-key-here"
```
 
### Making changes
 
1. Create a new branch for your change:
```bash
   git checkout -b feature/your-feature-name
```
2. Make your changes and test them against a sample PDF.
3. Commit with a clear, descriptive message:
```bash
   git commit -m "Add: brief description of the change"
```
4. Push to your fork and open a pull request against `main`:
```bash
   git push origin feature/your-feature-name
```
 
### Contribution guidelines
 
- Keep pull requests focused on a single change — smaller PRs are easier to review.
- If you're fixing a bug, describe how to reproduce it in the PR description.
- If you're adding a feature, briefly explain the use case it solves.
- Match the existing code style (clear function names, comments on non-obvious logic).
- Update this README if your change affects setup, usage, or the pipeline's behavior.
- Be extra careful with anything touching the PII redaction or input validation logic — flag any changes to these in your PR description so they get a closer look.
### Reporting issues
 
If you find a bug or have a feature request, please open an issue with:
- A short description of the problem or request
- Steps to reproduce (for bugs)
- Expected vs. actual behavior

---

## License

[MIT](LICENSE)

---

*This README was edited and proofread with the help of [Claude](https://claude.ai).*
