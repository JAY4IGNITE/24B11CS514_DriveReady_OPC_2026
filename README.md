# Agentic AI Workshop

Repository for daily tasks, progressive agent architectures, and hands-on implementations in Agentic AI.

---

## Daily Task Tracker

### [DAY 1] Fundamentals & First API Call
- **Directory:** [`DAY1/`](file:///c:/Users/ramuv/AgenticAI/DAY1)
- **Detailed Guide:** [`DAY1/README.md`](file:///c:/Users/ramuv/AgenticAI/DAY1/README.md)
- **Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY1/task.ipynb)
- **Domain:** BrightPage Stationery & Office Supplies
- **Concepts & Implementation:**
  - Understanding LLM core mechanics: Next-token prediction, token estimation, temperature, context window limits, and statelessness.
  - Making the first API call to Groq using LangChain `ChatGroq` (`openai/gpt-oss-120b`).
  - Integrating safety checks using Meta Llama Prompt Guard (`meta-llama/llama-prompt-guard-2-22m`).
  - Interactive notebook testing interface with preset prompts.

---

### [DAY 2] Prompts, Memory, and Building an Agent
- **Directory:** [`DAY2/`](file:///c:/Users/ramuv/AgenticAI/DAY2)
- **Detailed Guide:** [`DAY2/README.md`](file:///c:/Users/ramuv/AgenticAI/DAY2/README.md)
- **Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY2/task.ipynb)
- **Domain:** BrightPage Stationery & Office Supplies
- **Concepts & Implementation:**
  - **4-Pillar System Prompt:** Calibrating **Role**, **Facts** (catalogue prices, free delivery over ₹500, bulk discounts), **Rules** (unlisted item fallback: *"I'll check with the store manager..."*), and **Format** (concise, under 60 words).
  - **Few-Shot In-Context Prompting:** High-accuracy customer intent classifier returning exact labels (`order`, `inquiry`, `complaint`).
  - **Structured JSON Output:** Extracting unstructured conversational customer messages into database-ready order JSON schemas.
  - **Python Foundations for Agents:** Inventory dictionaries, type hints, docstrings, and `@log_call` execution logging decorators.
  - **Memory by Hand:** Stateful multi-turn conversation loop with sliding-window history trimming (`MAX_TURNS`) to preserve context window limits.
  - **Error Resilience:** Wrapping API calls in `try / except` to catch `RateLimitError` (429) or connection drops gracefully.
  - **Interactive Colab Dashboard:** `ipywidgets` interface featuring live chat bubbles, quick policy test presets, intent classifier tester, and JSON order extraction.
  - **Student's 10% Verification Suite:** Experiments testing temperature variations ($T=0.2$ vs $T=1.2$), hallucination boundaries, and memory window limits.

---

### [DAY 3] Prompt Templates, Memory, Evaluation & Agent Sneak Peek (Gemini API)
- **Directory:** [`DAY3/`](file:///c:/Users/ramuv/AgenticAI/DAY3)
- **Detailed Guide:** [`DAY3/README.md`](file:///c:/Users/ramuv/AgenticAI/DAY3/README.md)
- **Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY3/task.ipynb)
- **Domain:** BrightPage Stationery & Office Supplies
- **Concepts & Implementation:**
  - **Gemini API Integration:** Migrated from Groq to `ChatGoogleGenerativeAI` (`gemini-3.5-flash-lite`) via `langchain-google-genai`.
  - **LCEL Chains & Memory:** `ChatPromptTemplate`, `MessagesPlaceholder("history")`, and `StrOutputParser()`.
  - **Automated Prompt Evaluation:** CSV-driven test harness (`tests.csv`, `brightpage_tests.csv`) measuring pass rates across prompt iterations.
  - **Temperature & Stability:** Comparing `temperature=0.3` vs two runs at `temperature=1.0`.
  - **LangChain Agent Sneak Peek:** `from langchain.agents import create_agent` and classifying BrightPage jobs as `plain code`, `one LLM call`, or `agent`.

---

### [DAY 4] Tools, Structured Records, Sheets & Agent Traces
- **Directory:** [`DAY4/`](file:///c:/Users/ramuv/AgenticAI/DAY4)
- **Detailed Guide:** [`DAY4/README.md`](file:///c:/Users/ramuv/AgenticAI/DAY4/README.md)
- **Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY4/task.ipynb)
- **Domain:** BrightPage Stationery & Office Supplies
- **Concepts & Implementation:**
  - **Tool-Calling Agent:** `@tool` decorators (`get_stationery_price`, `is_store_open`, `check_delivery_fee`, `log_stationery_order`) wired into `create_agent` with `ChatGroq` (`openai/gpt-oss-20b`).
  - **Pydantic Validation & `with_structured_output`:** Enforced `StationeryOrder` schema and tested structured extraction on 5 messy BrightPage customer messages.
  - **Sheet Integration:** Connected to Google Sheets (`BrightPage Stationery Orders`) via `gspread` in Colab with a local CSV-backed `LocalWorksheet` mirror (`brightpage_orders.csv`).
  - **Customer Edge-Case Testing:** Verified `"scientific calculator"` (not in catalogue), missing delivery date, and `0 geometry box sets` prevent invalid sheet writes and added them to [`DAY3/brightpage_tests.csv`](file:///c:/Users/ramuv/AgenticAI/DAY3/brightpage_tests.csv) (13/13 passing).
  - **Trace Inspection & Break-It Test:** Counted model vs tool calls using `pretty_print()`, and verified how renaming `get_stationery_price` to `tool1` (`"does stuff"`) breaks tool selection before restoring it.

---

### [DAY 5] Memory (`InMemorySaver` + `thread_id`) & RAG (`ChromaDB`)
- **Directory:** [`DAY5/`](file:///c:/Users/ramuv/AgenticAI/DAY5)
- **Detailed Guide:** [`DAY5/README.md`](file:///c:/Users/ramuv/AgenticAI/DAY5/README.md)
- **Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY5/task.ipynb)
- **Domain:** BrightPage Stationery & Office Supplies (+ 3-paragraph PDF Story RAG Lab)
- **Concepts & Implementation:**
  - **Thread Memory (`InMemorySaver` + `thread_id`):** Isolated multi-customer conversations (`"ravi"` vs `"sravani"`) combining recall, RAG policy answers, and sheet order logging.
  - **RAG with `ChromaDB` & `all-MiniLM-L6-v2`:** Indexed 16 BrightPage store policies/FAQs and exposed `@tool search_brightpage_faq` alongside Day 4 tools.
  - **Single-Agent Sign-Off (`tests.csv`):** Evaluated 18 test cases (13 BrightPage core/edge + 5 RAG policy tests) with a fresh `thread_id` per row, achieving **17/18 (94%)**.
  - **Multi-Paragraph PDF RAG Lab (`story_rag.pdf`):** Extracted and chunked *The Whispering Umbrella* (3 paragraphs) with `pypdf`, indexed in ChromaDB, and verified grounded Q&A plus honest refusal on unanswerable questions.

---

### [DAY 6] Workflows, Planning & LangGraph
- **Directory:** [`DAY6/`](file:///c:/Users/ramuv/AgenticAI/DAY6)
- **Detailed Guide:** [`DAY6/README.md`](file:///c:/Users/ramuv/AgenticAI/DAY6/README.md)
- **Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY6/task.ipynb)
- **Domain:** BrightPage Stationery & Office Supplies
- **Concepts & Implementation:**
  - **Planning Techniques:** Compared Chain-of-Thought (no tools), ReAct (`create_agent`), and Plan-and-Execute (`ExecutionPlan` structured output + tool execution) on a multi-item school workshop quote (`Rs 2,170`, free delivery).
  - **Plain-Python Router to LangGraph `StateGraph`:** Built `Intent` structured classifier (`classify_with`), `BrightPageState` (`TypedDict`), nodes, `add_conditional_edges`, and Mermaid graph visualization (`brightpage_router_graph.png`).
  - **Focused Agents Inside Workflow Nodes:** Separated `faq_agent` (read-only pricing/FAQ/hours tools, no order logging) from `order_agent` (pricing + `log_stationery_order`) and a zero-LLM `other` branch.
  - **Streamed Routing & Edge Cases:** Achieved **8/8 (100%)** routing accuracy with `graph.stream(..., stream_mode="updates")`, handling ambiguous inquiries, Telugu-English code-mixed requests, and unlisted products cleanly.
  - **Extended 4-Branch Workflow (`ext_graph`):** Added `complaint` routing and a post-order `review_order` state node (`brightpage_extended_graph.png`) flagging bulk orders (`qty >= 10` or `discount > 0` or `total > 1000`) as `owner_review_required`.

---

## Setup & API Keys

- **Day 1, Day 2, Day 4, Day 5 & Day 6** use **Groq API Key** (`GROQ_API_KEY`).
- **Day 3** uses **Google Gemini API Key** (`GOOGLE_API_KEY` from [Google AI Studio](https://aistudio.google.com/app/apikey)).
- **Google Colab:** Add `GROQ_API_KEY` / `GOOGLE_API_KEY` to Colab Secrets (the Key icon on the left sidebar) and toggle **"Notebook access"** to **ON**.
- **Local Runs:** Managed via the single root `.env` (automatically discovered across all days via `python-dotenv`). Copy `.env.example` to `.env` and add your keys.
