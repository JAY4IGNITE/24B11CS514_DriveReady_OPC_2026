# 🧠 Day 5: Memory (`InMemorySaver` + `thread_id`) & RAG (`ChromaDB`)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JAY4IGNITE/24B11CS514_DriveReady_OPC_2026/blob/main/DAY5/task.ipynb)

Comprehensive guide, concepts, and verified implementation details for **Day 5** of the Agentic AI Workshop — completing the single agent for **BrightPage Stationery & Office Supplies**, plus a **multi-paragraph PDF RAG Lab**.

---

## 🎯 Task Overview
Add **short-term thread memory** (`InMemorySaver` + `thread_id`) and **Retrieval-Augmented Generation (RAG)** (`ChromaDB` + `all-MiniLM-L6-v2` embeddings) to the **BrightPage Stationery & Office Supplies** agent, then apply the same chunking + vector search workflow to a standalone 3-paragraph story PDF ([`story_rag.pdf`](file:///c:/Users/ramuv/AgenticAI/DAY5/story_rag.pdf)).

- **Primary Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY5/task.ipynb)
- **Domain:** BrightPage Stationery & Office Supplies
- **PDF RAG Document:** [`story_rag.pdf`](file:///c:/Users/ramuv/AgenticAI/DAY5/story_rag.pdf) (*The Whispering Umbrella* — 3-paragraph story, 1,421 extracted characters)
- **Sign-Off Test Suite:** [`tests.csv`](file:///c:/Users/ramuv/AgenticAI/DAY5/tests.csv) (18 rows: 13 from Day 3/4 + 5 new Day 5 RAG policy tests)
- **Local Sheet Output:** [`brightpage_orders.csv`](file:///c:/Users/ramuv/AgenticAI/DAY5/brightpage_orders.csv)
- **Models Used:** `openai/gpt-oss-120b` via Groq (`ChatGroq`, `temperature=0.2`) & `all-MiniLM-L6-v2` embeddings (`SentenceTransformerEmbeddingFunction`)

---

## 🧠 Core Engineering Concepts

### 1. Three Kinds of Memory
- **Conversation memory (short-term):** Messages inside one chat (`InMemorySaver` + `thread_id`).
- **Customer memory (long-term):** Facts saved across chats (e.g., customer preferences in a sheet/database).
- **Knowledge:** Business policies and FAQs stored in documents and retrieved via **RAG** (`ChromaDB`).

### 2. Conversation Memory: `InMemorySaver` & `thread_id`
Passing `checkpointer=InMemorySaver()` to `create_agent` and `config={"configurable": {"thread_id": thread_id}}` per request isolates concurrent chats automatically:
- Thread `"ravi"` remembers Ravi's name, Rs 700 order context, and logs Ravi's confirmed order (`2 x a4 copier paper ream (500 sheets)`, `29 Nov`, `Rs 640`, free delivery) without re-asking his name.
- Thread `"sravani"` keeps Sravani's gift-wrapping inquiry completely separate from Ravi's order.

### 3. Embeddings, Vector Search (`ChromaDB`) & Chunking
- Stored **16 BrightPage FAQs & policies** in `brightpage_faq` using `faq_collection.upsert(...)` and `SentenceTransformerEmbeddingFunction("all-MiniLM-L6-v2")`.
- Demonstrated `chunk_text(text, chunk_size=220, overlap=40)` splitting 573 characters of policy text into 4 overlapping chunks.
- Wrapped retrieval in `@tool search_brightpage_faq(question)` returning the top 3 matching policy chunks.

### 4. Customer Checks, Break-It Test & Sign-Off (`17/18 = 94%`)
- **Paraphrased policy check:** `"If a notebook arrives torn, how many days do I have to swap it?"` -> correctly retrieves the 3-day replacement policy.
- **Break-it out-of-scope check:** `"Do you sell laptops or wireless keyboards?"` -> replies `"I'll check with the store manager about availability."` (no hallucination).
- **18-row Sign-Off Suite (`tests.csv`):** Run with a fresh `thread_id` (`eval_row_{idx}`) per question -> **17/18 passed (94%)**, well above the 80% sign-off bar.

### 5. Multi-Paragraph PDF RAG Lab (`story_rag.pdf`)
Following the PDF RAG workflow from `RAG-2.ipynb`:
- Created and parsed [`story_rag.pdf`](file:///c:/Users/ramuv/AgenticAI/DAY5/story_rag.pdf) (*The Whispering Umbrella*, 3 paragraphs) with `pypdf.PdfReader`.
- Split into 3 clean paragraph chunks (`473`, `473`, and `449` characters) and indexed them in `story_pdf_rag` (`ChromaDB`).
- Verified `ask_pdf(question, top_k=2)`:
  - Grounded answers for `"Who did Meera run the stationery and bookshop with?"` (`"her grandfather"`), `"Why did Meera keep a bright yellow umbrella outside every morning?"`, and `"How did Meera and her grandfather help the postman during the storm?"`.
  - Honest refusal on `"What was the postman's pet dog named?"` -> `"The answer is not available in the PDF."`

---

## 🏗️ Notebook Implementation Breakdown

1. **Section 1 · Setup & Sheet Connection:** Resolves `GROQ_API_KEY`, configures `invoke_retry`, and initializes **BrightPage Stationery Orders** (`gspread` in Colab or `LocalWorksheet` locally).
2. **Section 2 · BrightPage ChromaDB Knowledge Base:** Upserts 16 policies/FAQs into `brightpage_faq` and verifies semantic search.
3. **Section 3 · Chunking & Tools:** Demonstrates overlapping text chunking and defines `get_stationery_price`, `is_store_open`, `check_delivery_fee`, `search_brightpage_faq`, and `log_stationery_order`.
4. **Section 4 · Thread Memory Agent (`InMemorySaver`):** Tests multi-turn memory + RAG + order logging across `"ravi"` and `"sravani"` threads.
5. **Section 5 · Customer Tests, Break-It & Sign-Off (`tests.csv`):** Runs paraphrased/out-of-scope checks and evaluates all 18 rows of [`tests.csv`](file:///c:/Users/ramuv/AgenticAI/DAY5/tests.csv) (**17/18, 94%**).
6. **Section 6 · Multi-Paragraph PDF RAG Lab:** Extracts, chunks, indexes, and queries [`story_rag.pdf`](file:///c:/Users/ramuv/AgenticAI/DAY5/story_rag.pdf).

---

## 🚀 How to Run

### In Google Colab
1. Open [`DAY5/task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY5/task.ipynb) via the **Open In Colab** badge above.
2. Upload [`story_rag.pdf`](file:///c:/Users/ramuv/AgenticAI/DAY5/story_rag.pdf) and [`tests.csv`](file:///c:/Users/ramuv/AgenticAI/DAY5/tests.csv) (or clone the repository) and add **`GROQ_API_KEY`** to Colab **🔑 Secrets**.
3. Run all cells (`Ctrl + F9`).

### Locally
1. Ensure `GROQ_API_KEY` is set in the root [`.env`](file:///c:/Users/ramuv/AgenticAI/.env) file.
2. Install dependencies: `pip install langchain langchain-groq langgraph chromadb sentence-transformers pypdf pydantic python-dotenv`.
3. Open `DAY5/task.ipynb` and run all cells.
