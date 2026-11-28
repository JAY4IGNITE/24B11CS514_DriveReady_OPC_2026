# 🔀 Day 6: Workflows, Planning & LangGraph

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JAY4IGNITE/24B11CS514_DriveReady_OPC_2026/blob/main/DAY6/task.ipynb)

Comprehensive guide, concepts, and verified implementation details for **Day 6** of the Agentic AI Workshop — refactoring the **BrightPage Stationery & Office Supplies** assistant into a structured **LangGraph** workflow with focused specialist agents.

---

## 🎯 Task Overview
Compare **Chain-of-Thought**, **ReAct**, and **Plan-and-Execute**, build a plain-Python intent router, and rebuild it in **LangGraph** (`StateGraph`, `TypedDict` state, nodes, conditional edges, Mermaid diagram rendering, and `stream_mode="updates"`). Then extend the workflow (Student's 50%) with a 4-intent router (`order`, `faq`, `complaint`, `other`) and a post-order `review_order` state node.

- **Primary Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY6/task.ipynb) (11 code cells, all executed cleanly)
- **Domain:** BrightPage Stationery & Office Supplies
- **Rendered Graph Diagrams:** [`brightpage_router_graph.png`](file:///c:/Users/ramuv/AgenticAI/DAY6/brightpage_router_graph.png) & [`brightpage_extended_graph.png`](file:///c:/Users/ramuv/AgenticAI/DAY6/brightpage_extended_graph.png)
- **Local Sheet Output:** [`brightpage_orders.csv`](file:///c:/Users/ramuv/AgenticAI/DAY6/brightpage_orders.csv)
- **Models Used:** `openai/gpt-oss-120b` via Groq (`ChatGroq`, `temperature=0.2`) & `all-MiniLM-L6-v2` embeddings (`SentenceTransformerEmbeddingFunction`)

---

## 🧠 Core Engineering Concepts

### 1. Workflows vs Agents & Three Planning Techniques
Evaluated on a multi-part request (*"School workshop order for Saturday: 20 spiral notebooks and 2 A4 copier paper reams. What is the total and will local home delivery be free?"*):
- **Chain-of-Thought (no tools):** Reasons step by step and correctly refuses to invent prices without catalogue access.
- **ReAct Agent (with tools):** Dynamically calls `get_stationery_price` and `check_delivery_fee` -> computes `20 × Rs 85` (10% bulk discount = `Rs 1530`) + `2 × Rs 320` (`Rs 640`) = **`Rs 2,170`** with **free delivery** (`> Rs 500`).
- **Plan-and-Execute:** Uses `llm.with_structured_output(ExecutionPlan)` to generate a 5-step plan first, then executes it with the tool agent to produce the same verified `Rs 2,170` total and free-delivery answer.

### 2. Plain-Python Router vs LangGraph `StateGraph`
- **Intent Classification (`classify_with`):** Wraps `llm.with_structured_output(Intent)` with an explicit `CLASSIFY_SYSTEM` prompt so conversational greetings (e.g., `"Good morning!"`, `"Thanks, have a great day!"`) always return structured intent labels instead of triggering conversational replies.
- **Shared State (`BrightPageState`):** `TypedDict` holding `message`, `intent`, and `reply`; each node returns only the keys it updates.
- **Focused Agents Inside Nodes:**
  - `faq_agent`: Has `[search_brightpage_faq, get_stationery_price, is_store_open, check_delivery_fee]` — **cannot** call `log_stationery_order`.
  - `order_agent`: Has `[get_stationery_price, check_delivery_fee, log_stationery_order]`.
  - `other_node`: Returns a fixed polite greeting with zero extra LLM calls.

### 3. Streaming & Routing Verification (`8/8 = 100%`)
Streamed `graph.stream({"message": msg}, stream_mode="updates")` across 8 test cases including ambiguous inquiries, Telugu-English code-mixed messages, and unlisted items:
- **8/8 (100%)** routed to the expected node (`faq`, `order`, `other`).
- `"I want to know if you can supply 20 spiral notebooks by tomorrow"` -> correctly classified as `faq` (answered without logging a false order).
- `"Naku 2 geometry box sets kavali, store pickup on 28 Dec, my name is Priya"` -> routed to `order` and saved (`Rs 320`).
- `"Do you sell laptops?"` -> routed to `faq` -> `"I'll check with the store manager about availability."`

### 4. Student's 50%: Extended 4-Branch Workflow (`ext_graph`)
Built `ExtendedBrightPageState` (`message`, `intent`, `order_record`, `approval_status`, `reply`) with `ExtendedIntent` (`order`, `faq`, `complaint`, `other`) and a post-order `review_order` node:
- `order` node captures the newly logged sheet record into `state["order_record"]`.
- `review_order` reads `state["order_record"]` and flags bulk/high-value orders (`qty >= 10`, `discount > 0`, or `total > 1000`) as `owner_review_required` (previewing Day 7's human-in-the-loop approval).
- Verified all 4 branches cleanly (`['faq', 'order', 'complaint', 'other']`) with Meena's 10-notebook order (`Rs 765`, `Rs 85` bulk discount) flagged as `owner_review_required`.

---

## 🏗️ Notebook Implementation Breakdown

1. **Cells 1–5 · Setup, Sheet Mirror, ChromaDB & Tools:** Installs dependencies, initializes `ChatGroq`, connects `LocalWorksheet` (`brightpage_orders.csv`), indexes 16 BrightPage FAQs in ChromaDB, and defines the 5 BrightPage `@tool`s.
2. **Cell 6 · Planning Techniques:** Compares Chain-of-Thought, ReAct, and Plan-and-Execute on a multi-item workshop quote.
3. **Cell 7 · Plain-Python Router:** Builds `Intent`, `classify_with`, and `handle_plain_python` with `if / elif / else`.
4. **Cell 8 · Focused Specialist Agents:** Creates `faq_agent` and `order_agent` with separated tool permissions.
5. **Cell 9 · LangGraph Router & Visualization:** Compiles `graph` (`START -> classify -> {order, faq, other} -> END`) and renders `brightpage_router_graph.png`.
6. **Cell 10 · Streamed Tests & Edge Cases:** Streams all 8 customer messages (`8/8` routing accuracy) and verifies sheet rows.
7. **Cell 11 · Extended 4-Intent Workflow (Student 50%):** Compiles `ext_graph`, renders `brightpage_extended_graph.png`, verifies all 4 intents and `review_order`, and confirms all 3 valid orders in [`brightpage_orders.csv`](file:///c:/Users/ramuv/AgenticAI/DAY6/brightpage_orders.csv).

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
