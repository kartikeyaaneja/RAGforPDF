# Project Summary

## 📁 Folder Structure

- `./datasets`:  
  Contains original PDF and its converted markdown file.

- `./data`:  
  Contains data created using code. This is saved to avoid rerunning expensive operations.

- `./images`:  
  Contains extracted images from the PDF.

## 📄 Text File

- `requirements.txt`:  
  Required Python libraries.

## 📓 Notebooks

- `Indexing.ipynb`:  
  Creates and stores chunks from a PDF file.

- `RAG.ipynb`:  
  Contains modules for retrieval, reranking, query rewriting and expansion, LLM generation, and a final function pipelining all the others.

---

## 🔍 Detailed Explanation of Code

### `Indexing.ipynb`

- Uses **Docling** to:
  - Convert the PDF to markdown.
  - Chunk the markdown content.
  - Extract unique images (by hash) from the PDF.
- Uses **GPT-4o** to summarize each unique image (if `OPENAI_API_KEY` is set in the environment).
- Embeddings are generated using **HuggingFace's** `BAAI/bge-large-en-v1.5`.
- Writes chunk data to:
  - `largeEmbeddings.csv` in `./data`
  - MongoDB (optional, for reuse and quick access)
- CSV fields: `chunk text`, `page number`, `imagePath` (NaN for text/table chunks), and `embeddings`.

> **Note**:  
> Code execution is conditional. If `largeEmbeddings.csv` exists, it simply reads and displays the file as a DataFrame.

---

### `RAG.ipynb`

- **Query Rewriting & Expansion Module**:
  - Input: A single query.
  - Output: `[rewritten query, broader context query, narrower context query]`.

- **Chunk Retrieval Module**:
  - Input: List of queries.
  - Output: Top N matched chunks (merged, deduplicated).

- **Re-Ranking Module**:
  - Model: `BAAI/bge-reranker-large`.
  - Filters: Only retains chunks with a **positive re-rank score**.

- **LLM Generation Module**:
  - Supports both MCQs and long answers.
  - Recommendations:
    - `llama3.2` for MCQs.
    - Avoid `DeepSeek` for MCQs.

- **Full Pipeline Function Parameters**:
  - `query`: The question being asked.
  - `mcq=False`: Whether it's a multiple-choice question.
  - `n=10`: Number of chunks to retrieve.
  - `coll="doc"`: MongoDB collection to use.
  - `llmModelName="llama3.2"`: LLM for generation.
  - `htmlOut=[]`: Populated with HTML links to relevant images.

- **Evaluation Module**:
  - Runs only if corresponding `.csv` files don’t exist:
    - `mcqs_predicted.csv`
    - `longQA_predicted.csv`
  - Metrics:
    - Accuracy for MCQs across 3 models.
    - Levenshtein Similarity, ROUGE, BLEU for long answers.
  - Includes test queries with image/table references.
  - Interactive **DEMO** at the bottom for step-by-step and full pipeline views.

---

## ⚙️ Running the Notebooks

### ➤ Running `Indexing.ipynb`

- To regenerate everything:
  - Delete/move/rename `largeEmbeddings.csv` in `./data`.
- Docling conversion may take **10–25 minutes**, depending on system specs.
- Set the `OPENAI_API_KEY` in your environment to enable **image chunk summaries**.
- If `largeEmbeddings.csv` exists, it will include previously computed image chunks.

### ➤ Running `RAG.ipynb`

- To rerun evaluations:
  - Delete/move/rename `mcqs_predicted.csv` or `longQA_predicted.csv`.
- Evaluation duration:  
  **2–30 seconds per query**, depending on LLM used (overall time: **1–25 minutes**).
- To test different models:
  - Modify the test model list in the evaluation section.
  - Ensure the models are available via **Ollama**.

> **Note**:  
> Jupyter may throw some JavaScript-related display errors while downloading models. These do **not** affect code execution.

---