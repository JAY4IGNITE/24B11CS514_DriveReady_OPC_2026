# 🔀 Day 6: Workflows, Planning & LangGraph

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JAY4IGNITE/24B11CS514_DriveReady_OPC_2026/blob/main/DAY6/task.ipynb)

Comprehensive guide, concepts, and verified implementation details for **Day 6** of the Agentic AI Workshop — refactoring the **BrightPage Stationery & Office Supplies** assistant into a structured **LangGraph** workflow with focused specialist agents, following the exact Day 6 notebook structure.

---

## 🎯 Task Overview
Build a plain-Python intent router (`handle`), then rebuild it in **LangGraph** (`StateGraph`, `TypedDict` state, `classify`, `ask_agent`, `answer_faq`, `take_order`, `other`, `route`, Mermaid diagram rendering, and `stream_mode="updates"`). Complete the **Your 50%** extension with a 4-intent router (`order`, `faq`, `complaint`, `other`) and a post-order `review_order` node.

- **Primary Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY6/task.ipynb) (11 code cells, all executed cleanly)
- **Domain:** BrightPage Stationery & Office Supplies
- **Rendered Graph Diagrams:** [`brightpage_router_graph.png`](file:///c:/Users/ramuv/AgenticAI/DAY6/brightpage_router_graph.png) & [`brightpage_extended_graph.png`](file:///c:/Users/ramuv/AgenticAI/DAY6/brightpage_extended_graph.png)
- **Local Sheet Output:** [`brightpage_orders.csv`](file:///c:/Users/ramuv/AgenticAI/DAY6/brightpage_orders.csv)
- **Models Used:** `openai/gpt-oss-120b` via Groq (`ChatGroq`, `temperature=0.2`) & `all-MiniLM-L6-v2` embeddings (`SentenceTransformerEmbeddingFunction`)

---

## 🧠 Core Engineering Concepts

### 1. Plain-Python Router (`handle`)
- Uses `Intent(BaseModel)` with `Literal["order", "faq", "other"]` and `classifier = llm.with_structured_output(Intent)`.
- `handle(message)` classifies once with a system instruction (`"Classify the customer message into Intent."`) and routes via `if / elif / else`.

### 2. LangGraph State, Focused Agents & Conditional Edges
- **Shared State (`BrightPageState`):** `TypedDict` holding `message`, `intent`, and `reply`.
- **Focused Agents Inside Nodes:**
  - `faq_agent`: Has `[search_brightpage_faq, get_stationery_price, is_store_open, check_delivery_fee]` — **cannot** call `log_stationery_order`.
  - `order_agent`: Has `[get_stationery_price, check_delivery_fee, log_stationery_order]`.
  - `other`: Returns a fixed polite greeting with zero extra LLM calls.
- **Graph Wiring (`builder` -> `graph`):** `START -> classify -> route -> {order, faq, other} -> END`, visualized via `draw_mermaid_png()`.

### 3. Streaming & Customer Edge-Case Tests (`8/8`)
Streamed `graph.stream({"message": msg}, stream_mode="updates")` across 8 messages including hard-to-sort inquiries and Telugu-English mix:
- `"I want to know if you can supply 20 spiral notebooks by tomorrow"` -> `faq` (`Rs 1,530` total, free delivery, no order logged).
- `"I'm Ravi, book 2 a4 copier paper reams for home delivery on 24 Dec"` -> `order` (saved, `Rs 640`).
- `"Naku 2 geometry box sets kavali, store pickup on 28 Dec, my name is Priya"` -> `order` (saved, `Rs 320`).
- `"Good morning!"` and `"hi"` -> `other`.
- `"Hi, delivery charges entha?"` -> `faq`.

### 4. Student's 50%: Extended 4-Branch Workflow (`my_graph`)
Built `CustomShopState` (`message`, `intent`, `order_record`, `approval_status`, `reply`) with `CustomIntent` (`order`, `faq`, `complaint`, `other`) and a post-order `review_order` node:
- `my_order` records the newly saved row into `state["order_record"]`.
- `review_order` flags bulk/high-value orders (`qty >= 10`, `discount > 0`, or `total > 1000`) as `owner_review_required`.
- Verified all 4 branches cleanly, with Meena's 10-notebook order (`Rs 765`, `Rs 85` bulk discount) saved and marked `owner_review_required`.

---

## 🏗️ Notebook Implementation Breakdown

1. **Cells 1–5 · Day 5 Carry-Over Setup:** Dependency check, `ChatGroq` (`llm` + `invoke_retry`), `LocalWorksheet` (`brightpage_orders.csv`), 16 BrightPage FAQs in ChromaDB, and the 5 BrightPage `@tool`s.
2. **Cell 6 · Plain-Python Router (`handle`):** `Intent`, `classifier`, and `if / elif / else` routing test.
3. **Cell 7 · Build Cell 1 (State & Classifier):** `BrightPageState(TypedDict)`, `Intent`, and `classifier`.
4. **Cell 8 · Build Cell 2 (Two Focused Agents):** `FAQ_PROMPT`, `ORDER_PROMPT`, `faq_agent`, and `order_agent`.
5. **Cell 9 · Build Cell 3 (Nodes, Edges, Compile & Draw):** `classify`, `ask_agent`, `answer_faq`, `take_order`, `other`, `route`, `StateGraph`, and `brightpage_router_graph.png`.
6. **Cell 10 · Build Cell 4 (Run, Stream & Customer Tests):** Streams 8 customer messages and prints saved sheet rows.
7. **Cell 11 · Your 50% (Custom 4-Intent Workflow):** `CustomShopState`, `CustomIntent`, `my_graph`, `brightpage_extended_graph.png`, and 4-branch verification.

---

## 🚀 How to Run

### In Google Colab
1. Open [`DAY6/task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY6/task.ipynb) via the **Open In Colab** badge above.
2. Add **`GROQ_API_KEY`** to Colab **🔑 Secrets** and toggle **Notebook access** to **ON**.
3. Run all cells (`Ctrl + F9`).

### Locally
1. Ensure `GROQ_API_KEY` is set in the root [`.env`](file:///c:/Users/ramuv/AgenticAI/.env) file.
2. Install dependencies: `pip install langchain langchain-groq langgraph chromadb sentence-transformers pydantic python-dotenv ipython`.
3. Open `DAY6/task.ipynb` and run all cells.
