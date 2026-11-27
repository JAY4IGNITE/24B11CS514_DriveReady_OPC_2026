# 🧩 Day 3: Prompt Templates, Memory, Evaluation & Agent Sneak Peek

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JAY4IGNITE/24B11CS514_DriveReady_OPC_2026/blob/main/DAY3/task.ipynb)

Comprehensive guide, concepts, and implementation details for **Day 3** of the Agentic AI Workshop — adapted for **BrightPage Stationery & Office Supplies** using the **Google Gemini API** (`langchain-google-genai`).

---

## 🎯 Task Overview
Refactor raw API calls into composable **LangChain LCEL chains** with `ChatPromptTemplate`, `MessagesPlaceholder`, and `StrOutputParser`, evaluate prompt quality systematically with a CSV test suite (`tests.csv`), compare low vs high temperature stability, and preview LangChain's `create_agent`.

- **Primary Notebook:** [`task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY3/task.ipynb)
- **Domain:** BrightPage Stationery & Office Supplies
- **Test Suites:**
  - [`tests.csv`](file:///c:/Users/ramuv/AgenticAI/DAY3/tests.csv) — BrightPage Stationery & Office Supplies (10 core test cases)
  - [`brightpage_tests.csv`](file:///c:/Users/ramuv/AgenticAI/DAY3/brightpage_tests.csv) — BrightPage Stationery extended suite (13 test cases including Day 4 customer edge cases)
- **Model Used:** `gemini-3.5-flash-lite` via Google Gemini API (`ChatGoogleGenerativeAI`)

---

## 🧠 Core Engineering Concepts

### 1. Composable LCEL Chain (`prompt | llm | StrOutputParser()`)
Instead of manually assembling message lists and extracting `.content` every time:
- **`ChatPromptTemplate.from_messages(...)`**: Structures `system`, conversation `history`, and `human` inputs cleanly.
- **`MessagesPlaceholder("history")`**: Injects multi-turn conversation history directly into the template.
- **`StrOutputParser()`**: Converts the model's `AIMessage` output into a plain Python `str`.

### 2. Automated Prompt Evaluation (`tests.csv`)
Testing prompts manually with 1–2 chats misses regressions. A CSV test suite pairs realistic customer `question` inputs with deterministic `must_include` assertions so every prompt change yields a measurable pass rate (`passed / total`).

### 3. Temperature & Determinism (`0.3` vs `1.0`)
- **`temperature=0.3`**: Keeps factual prices, policies, and fallback phrasing consistent across runs.
- **`temperature=1.0`**: Increases response variety, which can cause wording drift across identical test runs.

### 4. Agent Sneak Peek (`create_agent`)
`from langchain.agents import create_agent` wraps the model, system prompt, and tools into an agent loop. With `tools=[]` it behaves like a standard chain; adding tools (as done in Day 4 for BrightPage) turns it into a multi-step reasoning agent.

### 5. Choosing the Right Tool: Plain Code vs One LLM Call vs Agent
| Business Job (BrightPage Stationery) | Approach | Why |
| :--- | :--- | :--- |
| Calculate cart subtotal, 10% bulk discount, and ₹40 delivery fee | **Plain code** | Deterministic arithmetic — exact, free, and instant. |
| Classify customer message as `order`, `inquiry`, or `complaint` | **One LLM call** | Natural language classification needs understanding, not tool loops. |
| Check stock, compute bulk totals, and log an order end-to-end | **Agent** | Requires multi-step reasoning and deciding which tools to call. |

---

## 🏗️ Notebook Implementation Breakdown

1. **Cell 1 · Setup (Gemini API):** Installs `langchain`, `langchain-google-genai`, and `python-dotenv`, resolves `GOOGLE_API_KEY` (from `.env`, Colab Secrets, or `getpass`), and initializes `ChatGoogleGenerativeAI(model="gemini-3.5-flash-lite", temperature=0.3)`.
2. **Cell 2 · Template + Memory + Chain:** Builds the BrightPage Stationery LCEL chain (`prompt | llm | StrOutputParser()`) and `ask()` helper with multi-turn memory.
3. **Cell 3 · Customer Verification:** Tests free delivery threshold, out-of-city delivery policy, unlisted item fallback (`scientific calculators`), and 6-notebook pricing math.
4. **Cell 4 · Automated Evaluation (`tests.csv`):** Runs `run_tests("tests.csv")` across 10 BrightPage customer questions (`V1`).
5. **Cell 5 · Agent Sneak Peek:** Runs `create_agent(model=llm, tools=[], system_prompt=BRIGHTPAGE_SYSTEM_V1)`.
6. **Cell 6 · Prompt Iteration (`V1` → `V2`):** Refines formatting/policy instructions (`V2`) and compares pass rates on `tests.csv`.
7. **Cell 7 · Temperature Stability (`1.0`):** Runs `tests.csv` twice at `temperature=1.0` and measures answer drift (`changed / 10`).
8. **Cell 8 · Plain Code vs One LLM Call vs Agent:** Maps 3 BrightPage Stationery business jobs to their ideal architecture.

---

## 🚀 How to Run

### In Google Colab
1. Open [`DAY3/task.ipynb`](file:///c:/Users/ramuv/AgenticAI/DAY3/task.ipynb) in Google Colab.
2. Click **🔑 Secrets** on the left sidebar, add **`GOOGLE_API_KEY`** with your key from [Google AI Studio](https://aistudio.google.com/app/apikey), and toggle **Notebook access** to **ON**.
3. Run all cells (`Ctrl + F9`).

### Locally
1. Add `GOOGLE_API_KEY=your_gemini_api_key_here` to the root [`.env`](file:///c:/Users/ramuv/AgenticAI/.env) file.
2. Install dependencies: `pip install langchain langchain-google-genai python-dotenv`.
3. Open `DAY3/task.ipynb` and run all cells.
