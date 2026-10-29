# 📚 RAG Book Assistant

Chat with any PDF, fully on your own machine. Upload a book, paper or report, ask questions in plain English, and get answers grounded **only** in that document.

## ✨ Features

- **Upload any PDF** from the browser and build a searchable knowledge base in one click
- **Grounded answers**: the model is told to answer only from the retrieved context, and to say so when the answer isn't in the document
- **100% local and free**: embeddings and the chat model both run through [Ollama](https://ollama.com), with no API keys, no usage limits and no data leaving your computer
- **Smarter retrieval**: uses MMR (Maximal Marginal Relevance) to return relevant *and* diverse chunks instead of near-duplicates
- **Two ways to use it**: a Streamlit web app, or a simple terminal chat

## 🧠 How It Works

mermaid
flowchart LR
    A[PDF upload] --> B[PyPDFLoader]
    B --> C["Text splitter<br/>1000 chars, 200 overlap"]
    C --> D["nomic-embed-text<br/>Ollama embeddings"]
    D --> E[(ChromaDB)]
    F[User question] --> G["MMR retriever<br/>top 4 chunks"]
    E --> G
    G --> H[Prompt + context]
    H --> I["gemma4:e2b<br/>via Ollama"]
    I --> J[Answer]

1. **Load**: the PDF is read page by page with `PyPDFLoader`.
2. **Split**: pages are cut into overlapping 1000-character chunks so context isn't lost at the boundaries.
3. **Embed**: each chunk is converted into a vector with `nomic-embed-text`.
4. **Store**: vectors are saved in a persistent ChromaDB collection (`chroma_db/`).
5. **Retrieve**: your question is embedded and the 4 most relevant, diverse chunks are fetched with MMR.
6. **Generate**: the chunks and question go to `gemma4:e2b`, which answers using only that context.

## 🛠️ Tech Stack

| Layer | Tool |
|---|---|
| UI | Streamlit |
| Orchestration | LangChain |
| Vector database | ChromaDB (`langchain-chroma`) |
| Embeddings | Ollama, `nomic-embed-text` |
| LLM | Ollama, `gemma4:e2b` |
| PDF parsing | PyPDF |

## 📁 Project Structure

RAG-book-assistant/
├── app.py               # Streamlit web app (upload PDF + ask questions)
├── main.py              # Terminal chat over an existing chroma_db
├── create_database.py   # One-off script: PDF -> chunks -> embeddings -> chroma_db
├── document loader/     # Loader experiments and test scripts
├── requirements.txt
└── .gitignore

## 🚀 Getting Started

### Prerequisites

- Python 3.10 or newer
- [Ollama](https://ollama.com/download) installed and running

### 1. Clone the repository

git clone https://github.com/sakshithyadav/RAG-book-assistant.git
cd RAG-book-assistant

### 2. Create a virtual environment and install dependencies

python -m venv venv

# Windows
venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

pip install -r requirements.txt

### 3. Pull the Ollama models

```bash
ollama pull nomic-embed-text
ollama pull gemma4:e2b
```

### 4. Run the app
bash
streamlit run app.py

Open http://localhost:8501, upload a PDF, click **Create Vector Database**, then ask your questions. Embedding a large book can take a few minutes, and a progress bar shows how far along it is.

## ⚙️ Configuration

| Setting | Where | Default |
|---|---|---|
| Chunk size / overlap | `RecursiveCharacterTextSplitter` | `1000` / `200` |
| Chunks sent to the LLM (`k`) | `as_retriever(search_kwargs)` | `4` |
| Candidates considered by MMR (`fetch_k`) | `as_retriever(search_kwargs)` | `10` |
| Relevance vs. diversity (`lambda_mult`) | `as_retriever(search_kwargs)` | `0.5` |
| Embedding model | `OllamaEmbeddings(model=...)` | `nomic-embed-text` |
| Chat model | `ChatOllama(model=...)` | `gemma4:e2b` |
| Embedding batch size | `BATCH_SIZE` in `app.py` | `32` |

Any model that Ollama can run will work. Use the **same embedding model** for building the database and for answering questions.

## 🧯 Troubleshooting

*Collection expecting embedding with dimension of X, got Y`**
The chroma_db folder was built with a different embedding model. Stop the app, delete chroma_db/, and rebuild it.

**connectex: No connection could be made(Ollama error)**
Ollama isn't running, or its embedding process crashed. Start or restart Ollama, run `ollama list` to confirm the models are installed, and lower `BATCH_SIZE` in `app.py` for very large PDFs.

**Answers are empty or "I could not find the answer in the document"**
Make sure the vector database was created from the PDF you're asking about. Scanned PDFs that are only images contain no extractable text and need OCR first.

**Answers mix up two different books**
Each new upload is added on top of the existing `chroma_db`. Delete the folder before uploading a different document.

## 🗺️ Roadmap

- [ ] Replace the old database automatically on every new upload
- [ ] Show source page numbers alongside each answer
- [ ] Chat history and follow-up questions
- [ ] Support for multiple PDFs and other file types
- [ ] OCR support for scanned documents

## 👤 Author

Built by **Sakshith**. Find me on GitHub: [@sakshithyadav](https://github.com/sakshithyadav).

If you found this useful, consider giving the repo a ⭐
