# PDF Q&A with RAG

Ask questions about a PDF and get answers with page citations.
Built with LangChain, Gemini (free tier) and FAISS.

## How it works

```
PDF -> chunks (1000 chars, 150 overlap) -> Gemini embeddings -> FAISS index
    -> top-5 retrieval -> Gemini answers using only the retrieved text
    -> answer + page numbers
```

If the answer is not in the document, the model says so instead of guessing.

## Tech stack

- Python 3.12
- LangChain
- Gemini: `gemini-embedding-001` (embeddings), `gemini-3.1-flash-lite` (answers)
- FAISS (vector search)
- PyPDF (PDF parsing)

## Project structure

```
pdf-qa-rag/
├── notebooks/
│   └── pdf_qa_rag.ipynb   # full pipeline, one notebook
├── data/                     # put your PDF here (not committed)
├── artifacts/                # saved FAISS index (not committed)
├── .env.example              # template for your API key
├── requirements.txt
└── README.md
```

## Setup (Windows, Command Prompt)

1. Clone the repo and enter the folder:
   ```cmd
   git clone https://github.com/YOUR_USERNAME/pdf-qa-rag.git
   cd pdf-qa-rag
   ```
2. Create and activate a virtual environment:
   ```cmd
   python -m venv .venv
   .venv\Scripts\activate
   ```
3. Install the libraries:
   ```cmd
   pip install -r requirements.txt
   ```
4. Copy `.env.example` to `.env` and add your free Gemini API key from Google AI Studio.
5. Download the DeepSeek-R1 paper from https://arxiv.org/abs/2501.12948 and save it as `data/deepseek-r1.pdf`.
6. Open `notebooks/pdf_qa_rag.ipynb`, select the `.venv` kernel and run all cells.

The first run embeds the PDF and takes about 5 minutes on the free tier. After that, the saved index loads in seconds.

To use a different PDF or change the chunk settings, delete the `artifacts/` folder first so the index is rebuilt.

## Usage

Change `QUESTION` in the last cell and rerun it:

```python
QUESTION = "What is GRPO?"
answer, pages_used = ask(QUESTION)
```

The answer is printed together with the pages it came from.

## Results on the sample paper

- 86 pages -> 297 chunks
- 3072-dimension embeddings
- Every answer lists the source pages
- Questions outside the paper (for example "What is the capital of France?") are refused instead of answered from the model's memory

## Design decisions

- **15% chunk overlap (150 of 1000)**, not 40%, to avoid near-duplicate chunks and wasted embedding cost.
- **Index saved to disk**, so embeddings are paid for only once.
- **Batches of 80 with a 65 s pause**, to stay under the free-tier limit of 100 embedding requests per minute.
- **Answer only from context** plus a fallback message, to reduce hallucination.
- **Page citations from chunk metadata**, so every answer can be verified.
- **`StrOutputParser`** instead of indexing into the response object, so the code does not break when the SDK response format changes.
- **Same-language answers**, so a question in one language is not answered in the language of the retrieved text.

## Known limitations

- Broad questions such as "what is this document about?" retrieve scattered chunks (often from the appendix), so the summary is weak. Specific questions work well.
- Retrieval quality is not yet measured.
- `langchain-community` is being phased out, so imports will move to standalone packages.

## Roadmap

- [ ] Evaluation set with hit-rate@k
- [ ] Chunk size experiments (500 / 1000 / 1500)
- [ ] Reranker, with before/after numbers
- [ ] Migrate off `langchain-community`
- [ ] Move logic into `src/`, add a Streamlit or FastAPI front end and Docker