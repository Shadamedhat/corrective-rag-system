# Corrective RAG System

A Corrective Retrieval-Augmented Generation (RAG) system that answers questions from course documents.
Unlike a standard RAG, it **checks the quality of the retrieved chunks before answering**. If the context is weak, it rewrites the question and searches again. If it still finds nothing, it says so instead of making up an answer.

---

## Project Objective

- Build an end-to-end Corrective RAG pipeline.
- Improve the reliability of document-based question answering.
- Evaluate retrieved chunks before generating the final answer.
- Detect irrelevant or low-quality retrieved context.
- Rewrite the user query and retrieve again when needed.
- Reduce hallucination by generating answers only from verified context.

---

## How It Works

```
Question -> Retrieve chunks -> Evaluate each chunk
                                   |
             good chunks found ----+---- no good chunks
                   |                          |
           Generate answer           Rewrite the question
                   |                          |
          Answer + Sources             Retrieve again -> Evaluate
                                                  |
                                   good chunks -> Generate answer
                                   still nothing -> "I could not find the answer
                                                    in the documents."
```

---

## Pipeline Steps

1. **Upload**: upload course files (PDF, DOCX, CSV, TXT).
2. **Extract**: read the text from each file type (PDF by page, CSV by row).
3. **Clean**: remove extra spaces and new lines, and drop very short texts.
4. **Split**: cut the text into chunks of 500 characters with an overlap of 100.
5. **Embed**: convert each chunk into a vector using `all-MiniLM-L6-v2`.
6. **Store**: save the vectors and metadata (source, location) in ChromaDB.
7. **Retrieve**: get the top 4 closest chunks to the question (cosine similarity).
8. **Evaluate**: an LLM checks each chunk and keeps only the ones that help answer the question.
9. **Rewrite**: if no good chunks are found, the LLM rewrites the question and retrieval runs again.
10. **Generate**: the LLM answers using the verified chunks only.
11. **Sources**: every answer is shown with its source file and location (page / row).
12. **Fallback**: if there is still no good context, the system replies that it could not find the answer.

---

## Documents Used

| Course | Files |
|---|---|
| Machine Learning | CSV file of ML questions and answers |
| Python | DOCX course guide |
| Linear Algebra | PDF (Strang, A = CR) and a TXT course guide |

Total: 76 extracted documents, split into 205 chunks.

---

## Tech Stack

| Part | Tool |
|---|---|
| Text extraction | `pypdf`, `python-docx`, `pandas` |
| Embeddings | `sentence-transformers` (`all-MiniLM-L6-v2`) |
| Vector database | ChromaDB |
| LLM | Groq API (`openai/gpt-oss-120b`) |
| Deployment | Gradio |

---

## Testing

The system was tested on 9 questions of 3 types:

| Type | Example | Expected behavior |
|---|---|---|
| Normal | What is the LEGB scope rule in Python? | Correct answer with sources |
| Tricky (different wording) | In what order does Python look for a variable name? | Correct answer with sources |
| Out of scope | What is the capital of France? | Query is rewritten, then "I could not find the answer in the documents." |

Result: all normal and tricky questions were answered correctly with sources, and all out-of-scope questions returned the fallback message without making up an answer.

---

## How to Run

1. Open the notebook in Google Colab.
2. Run the cells in order.
3. Upload your course files when asked.
4. Enter your Groq API key when asked (get one from https://console.groq.com/keys).
5. Run the last cell to launch the Gradio app and ask your questions.

Note: the API key is entered with `getpass`, so it is never saved in the notebook.

---

## Project Structure

```
corrective-rag-system/
|-- corrective_rag.ipynb   # full notebook (pipeline + Gradio app)
|-- README.md
```

---

## Author

**Shada Medhat**
