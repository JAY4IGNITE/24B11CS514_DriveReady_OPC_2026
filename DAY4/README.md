# 🛠️ Day 4: Tools, Structured Records, Sheets & Agent Traces

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JAY4IGNITE/24B11CS514_DriveReady_OPC_2026/blob/main/DAY4/task.ipynb)

Comprehensive guide, concepts, and verified implementation details for **Day 4** of the Agentic AI Workshop — built end-to-end for **BrightPage Stationery & Office Supplies**.

---

## 🎯 Task Overview
Build a complete tool-calling stationery shop assistant for **BrightPage Stationery & Office Supplies** using LangChain `create_agent`, `@tool` decorators, a Pydantic `StationeryOrder` schema, `with_structured_output`, and persistent order logging via Google Sheets (`gspread` in Colab) or a local CSV-backed `LocalWorksheet` mirror.

- **Primary Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY4/task.ipynb)
- **Domain:** BrightPage Stationery & Office Supplies
- **Regression Test Suite:** [`../DAY3/brightpage_tests.csv`](file:///c:/Users/ramuv/AgenticAI/DAY3/brightpage_tests.csv) (expanded from 10 to 13 rows)
- **Local Sheet Output:** [`brightpage_orders.csv`](file:///c:/Users/ramuv/AgenticAI/DAY4/brightpage_orders.csv)
- **Model Used:** `openai/gpt-oss-20b` via Groq (`ChatGroq`, `temperature=0.2`)

---

## 🧠 Core Engineering Concepts

### 1. `@tool` Functions & Why Docstrings Are the Model's API Docs
BrightPage Stationery exposes four `@tool`s with typed arguments and clear docstrings:
- **`get_stationery_price(item, qty=1)`:** Exact + partial/token-overlap lookup across the 10-item catalogue, enforces `qty >= 1`, and applies the 10% bulk discount for `qty >= 10` or subtotal `> Rs 1000`.
- **`is_store_open(day)`:** Returns store opening hours (`Mon–Sat 9:00 AM to 8:30 PM`, `Sun 10:00 AM to 4:00 PM`).
- **`check_delivery_fee(order_total, delivery_type)`:** Returns `Rs 0` for `store_pickup` or orders above `Rs 500`, and `Rs 40` otherwise.
- **`log_stationery_order(...)`:** Validates via `StationeryOrder`, computes discount and delivery fee, and appends a row to the **BrightPage Stationery Orders** sheet.

**Break-It Experiment (`tool1` / `"does stuff"`):**
Renaming `get_stationery_price` to `tool1` with docstring `"does stuff"` caused the model to make **0 tool calls (`[]`)** on `"How much are 3 spiral notebooks?"` and ask for the price/SKU instead of calling the tool. Restoring the descriptive name `get_stationery_price` and its docstring immediately restored tool selection (`['get_stationery_price']` -> `3 x spiral notebook (200 pgs) @ Rs 85 = Rs 255`).

### 2. Pydantic Schema (`StationeryOrder`) & `with_structured_output`
`StationeryOrder` validates `customer_name`, `item` (10-item `Literal`), `qty >= 1`, `delivery_type` (`"home_delivery"` | `"store_pickup"`), `delivery_date`, and optional `notes`.
- Tested `llm.with_structured_output(StationeryOrder)` on **5 messy customer messages** (slang, typos, abbreviations, pickup vs delivery, and packing/call notes) — all 5 parsed cleanly into valid `StationeryOrder` records.

### 3. Reading Agent Traces with `pretty_print()`
For `"Are you open on Sunday? How much are 2 A4 copier paper reams?"`:
- **Model calls (`AIMessage`):** `3`
- **Tool calls requested:** `2` (`['is_store_open', 'get_stationery_price']`)
- **Tool results (`ToolMessage`):** `2`

Execution sequence:
1. `HumanMessage` -> `AIMessage` calls `is_store_open(day="sunday")` -> `ToolMessage` returns `"Open 10:00 AM to 4:00 PM on sunday."`
2. `AIMessage` calls `get_stationery_price(item="a4 copier paper ream (500 sheets)", qty=2)` -> `ToolMessage` returns `"2 x a4 copier paper ream (500 sheets) @ Rs 320 = Rs 640"`
3. Final `AIMessage` synthesizes both tool results into a concise customer reply.

### 4. Customer Edge-Case Testing & Sheet Integrity
Tested three adversarial/incomplete customer queries on fresh BrightPage agent sessions:
1. **Unlisted item (`"scientific calculator"`):** `"Can I order 2 scientific calculators for tomorrow?"` -> Replies `"I'll check with the store manager about availability."` without quoting a price or logging an order.
2. **Missing delivery date:** `"I'm Sita. Please book 2 spiral notebooks for home delivery."` -> Quotes the total (`Rs 170` + `Rs 40` delivery = `Rs 210`) and asks for the delivery date before logging.
3. **Invalid quantity (`0 geometry box sets`):** `"I'm Kiran. Please order 0 geometry box sets for 25 Dec."` -> Explains minimum quantity is at least 1 and asks how many are needed.

- **Sheet integrity verified:** Before and after all 3 edge tests, the sheet retained only Ravi's 1 valid order (`Ravi`, `a4 copier paper ream (500 sheets)`, `2`, `home_delivery`, `24 Dec`, `320`, `0.0`, `0`, `640.0`).
- **Day 3 `brightpage_tests.csv` evaluation:** Added these 3 cases as rows 11–13 in [`DAY3/brightpage_tests.csv`](file:///c:/Users/ramuv/AgenticAI/DAY3/brightpage_tests.csv); the Day 4 BrightPage agent passed **13/13 (100%)**.

---

## 🏗️ Notebook Implementation Breakdown

1. **Section 1 · Setup & Sheet Connection:** Loads `GROQ_API_KEY` from `.env` (local), Colab Secrets, or `getpass`, defines `invoke_retry` for transient errors, and initializes **BrightPage Stationery Orders** via `gspread` (Colab) or `LocalWorksheet` (`brightpage_orders.csv` locally).
2. **Section 2 · BrightPage Stationery `@tool`s, Record & Agent:** Defines `StationeryOrder`, 4 `@tool`s, and `brightpage_agent`, verifying multi-turn conversation and Ravi's booked order (`Rs 640`, free delivery) in the sheet.
3. **Section 3 · Read the Trace:** Prints the full `pretty_print()` trace and counts `AIMessage` model calls (`3`) vs `ToolMessage`/tool calls (`2`).
4. **Section 4 · Structured Extraction on 5 Messy Messages:** Runs `llm.with_structured_output(StationeryOrder)` on 5 noisy WhatsApp-style orders.
5. **Section 5 · Test It Like a Customer:** Runs the `"scientific calculator"`, missing-date, and `0 geometry box sets` checks, asserts no bad rows entered the sheet, and evaluates all 13 rows of [`DAY3/brightpage_tests.csv`](file:///c:/Users/ramuv/AgenticAI/DAY3/brightpage_tests.csv) (**13/13 passed**).
6. **Section 6 · Break It & Restore It:** Demonstrates `tool1` (`"does stuff"`) failing to trigger tool calls (`[]`), then restores `get_stationery_price` and verifies tool invocation (`['get_stationery_price']`).

---

## 🚀 How to Run

### In Google Colab
1. Open [`DAY4/task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY4/task.ipynb) via the **Open In Colab** badge above.
2. Add **`GROQ_API_KEY`** in Colab **🔑 Secrets** and toggle **Notebook access** to **ON**.
3. Run all cells (`Ctrl + F9`) and complete the Google OAuth prompt when `auth.authenticate_user()` runs to create/open the Google Sheet.

### Locally
1. Ensure `GROQ_API_KEY` is set in the root [`.env`](file:///c:/Users/ramuv/AgenticAI/.env) file.
2. Install dependencies: `pip install langchain langchain-groq gspread pydantic python-dotenv`.
3. Open `DAY4/task.ipynb` and run all cells — orders will be persisted locally to [`brightpage_orders.csv`](file:///c:/Users/ramuv/AgenticAI/DAY4/brightpage_orders.csv).
