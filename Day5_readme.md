# Day 5 – PDF Question Answering with Embeddings, FAISS, and Groq (RAG)

## Overview
This notebook builds a question-answering system for a PDF document using Retrieval-Augmented Generation (RAG). It extracts text from an uploaded PDF, splits it into chunks, converts the chunks into embeddings, stores them in a FAISS index, and retrieves the most relevant chunks to help a Groq-hosted model answer questions from the document.

## Learning Objectives
- Understand what text embeddings are and compare two embedding models.
- Extract text from a PDF using PyMuPDF.
- Split text into overlapping chunks.
- Create embeddings with `sentence-transformers`.
- Store and search vectors with a FAISS index.
- Retrieve relevant context for a question.
- Prompt an LLM to answer only from the retrieved context.

## Tools and Libraries
| Tool / Library | Purpose |
|---|---|
| Python | Main programming language |
| `pymupdf` (`fitz`) | Extracts text from PDF files |
| `sentence-transformers` | Creates text embeddings |
| `faiss-cpu` | Stores embeddings and performs similarity search |
| `numpy` | Handles embedding arrays |
| `groq` | Groq Python SDK for calling the chat model |
| `easyocr`, `pdf2image`, `poppler-utils` | Installed for OCR on scanned PDFs (not used in the notebook) |
| `all-MiniLM-L6-v2` | Embedding model used for the PDF chunks and questions |
| `all-mpnet-base-v2` | Second embedding model, used only in the comparison |
| `openai/gpt-oss-20b` | Chat model that generates the answers (temperature 0.2) |

## Notebook Walkthrough

### 1. Install dependencies
```python
!pip -q install pymupdf sentence-transformers faiss-cpu openai numpy groq
```

### 2. Compare embedding models
The sentence "The Little Rooftop Garden" is encoded with `all-MiniLM-L6-v2` and `all-mpnet-base-v2`. Printing the vector shapes shows that different models produce vectors of different sizes.

### 3. Install OCR tools
`poppler-utils`, `easyocr`, and `pdf2image` are installed, but the notebook does not use them. Text is read directly from the PDF.

### 4. Upload the PDF
```python
from google.colab import files
uploaded = files.upload()
pdf_path = next(iter(uploaded))
```

### 5. Extract text
```python
doc = fitz.open(pdf_path)
pdf_text = ""
for page in doc:
    pdf_text += page.get_text()
```
The notebook prints the total character count and the first 2,000 characters.

### 6. Split into chunks
| Setting | Value |
|---|---|
| Chunk size | 500 characters |
| Overlap | 100 characters |

Overlap keeps sentences that fall on a chunk boundary available in both neighboring chunks. Empty chunks are skipped.

### 7. Create embeddings and the FAISS index
- Each chunk is encoded with `all-MiniLM-L6-v2`.
- Embeddings are normalized with `faiss.normalize_L2`.
- `IndexFlatIP` stores them; with normalized vectors, inner product equals cosine similarity.

### 8. Connect to Groq
The API key is read from the Colab secret `GROQ_KEY2` and passed to the `Groq` client.

### 9. Answer questions with `ask_pdf()`
1. Encode and normalize the question.
2. Search the FAISS index for the 3 most similar chunks.
3. Join those chunks into a context string.
4. Prompt the model to answer using only that context, or reply "The answer is not available in the document." if the context lacks the answer.
5. Return the model's reply.

### 10. Interactive loop
A `while True` loop asks for a question, prints the answer, and stops when the user types `exit`.

## How to Run
1. Open the notebook in Google Colab.
2. Add your Groq API key as a Colab secret named `GROQ_KEY2` and enable notebook access.
3. Run the install cell and the embedding comparison.
4. Run the PDF upload cell and choose a text-based PDF.
5. Run the cells for text extraction, chunking, embeddings, and the FAISS index.
6. Run the Groq connection cell and define `ask_pdf()`.
7. Run the final cell and ask questions about the PDF. Type `exit` to stop.

## Key Takeaways
- RAG gives an LLM relevant document content so it answers from the source.
- Chunking with overlap preserves context across chunk boundaries.
- Embeddings turn text into vectors that can be searched by meaning.
- Normalized vectors with `IndexFlatIP` give cosine-similarity search.
- Instructing the model to answer only from the context reduces unsupported answers.

## Notes
- The notebook calls `files.upload()` twice. The second call defines `pdf_path`, so upload the PDF again when prompted.
- `fitz` reads text-based PDFs. Scanned PDFs need OCR, and the OCR libraries are installed but not used.
- The notebook has no saved outputs, so the PDF content and sample answers are not shown.
