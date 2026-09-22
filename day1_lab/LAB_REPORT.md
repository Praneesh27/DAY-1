# Day 1 Lab Report & Observation Manual
**Course**: Agentic AI: Foundations and Open-Source Practice  
**Lab Title**: Python in VS Code: Chatbot vs Rule-Based Workflow vs AI Agent  
**Unit**: Unit 1: Foundations of AI Agents (sub-topics 1.1 and 1.2)  

---

## 10. Observations

### Comparative Evaluation Table

| Criterion | Chatbot (System 1) | Rule-Based Workflow (System 2) | AI Agent (System 3) |
| :--- | :--- | :--- | :--- |
| **Q1 correct? (Y/N)** | **N** (Hallucinates an arbitrary fee like Rs. 25,000) | **Y** (Exact: Rs. 18,000) | **Y** (Exact: Rs. 18,000 via `get_course_fee`) |
| **Q2 correct? (Y/N)** | **N** (Guesses incorrect base fees and calculations) | **Y** (Exact: Rs. 27,000 calculated via regex & formula) | **Y** (Exact: Rs. 27,000 via fee lookup + calculator tool) |
| **Q3 correct? (Y/N)** | **N** (Guesses price difference without data) | **N** ("Sorry, I do not have a rule for this type of question.") | **Y** (Exact: Yes, by Rs. 3,000 via fee lookups + subtraction tool) |
| **Q4 handled well? (Y/N)** | **Y** (Generates creative, relevant 2-line welcome message) | **N** ("Sorry, I can only answer questions about course fees.") | **Y** (Answers directly without calling unnecessary tools) |
| **Challenge question handled? (Y/N)** | **N** (Guesses combination based on hallucinated prices) | **N** ("Sorry, I do not have a rule for this type of question.") | **Y** (Inspects fees, tests combinations, finds CS101 + DS303 = Rs. 27,000) |
| **Same output on a repeat run? (Y/N)** | **No** (Varies with phrasing and generation sampling) | **Yes** (100% deterministic and reproducible) | **Largely Yes** (Deterministic results with `temperature=0`, though step order/wording can slight vary) |
| **Approximate response time** | ~1 - 2 seconds (single LLM inference call) | < 1 millisecond (instant in-memory regex match) | ~2 - 5 seconds (multi-step ReAct loop across multiple tool calls) |
| **Number of LLM calls per question** | Exactly 1 | 0 | 1 to 4 calls (1 call per reasoning iteration + final answer) |
| **One strength** | Highly fluent and creative with natural language; requires zero coding for new questions. | 100% reliable, zero hallucination, zero latency, free to execute. | Flexible problem solver: handles unseen queries by combining tools dynamically. |
| **One weakness** | Confidently hallucinates private or factual data; cannot perform reliable math. | Extremely rigid; breaks immediately on minor rephrasing or unseen queries. | Higher latency, non-deterministic reasoning paths, potential tool failure on small models. |
| **Best suited for** | Creative copy generation, summarization, general conversational FAQ without private facts. | Strict regulatory/financial calculations, invoice generation, fixed lookup APIs. | Autonomous assistants requiring dynamic research, calculation, multi-step problem solving, and API orchestration. |

---

### Agent Trace: Copy the Printed Steps for Question 2
**Question**: *"What is the total fee for CS101 and AI202 after a 10% scholarship?"*

| Step | Tool called and arguments | Result (observation) |
| :--- | :--- | :--- |
| **1** | `get_course_fee({'course_code': 'CS101'})` | `12000` |
| **1** | `get_course_fee({'course_code': 'AI202'})` | `18000` |
| **2** | `calculator({'expression': '(12000 + 18000) * 0.9'})` | `27000.0` |
| **3** | *No tool call (Final answer generated)* | `A: The total fee after a 10% scholarship is Rs. 27,000.` |

---

## 11. Discussion Questions

### 1. The chatbot gave a confident but wrong fee. Why is that more dangerous than replying "I don't know"?
A confident incorrect response (hallucination) misleads users into believing the information is verified and authoritative. In academic or commercial settings (e.g., student fee payments, contracts, medical diagnostics), relying on a fabricated number leads to financial discrepancies, loss of trust, and potential legal liability. If the system admits "I don't know," the user is alerted to verify with official sources.

### 2. The workflow was always correct for questions 1 and 2. Why might a finance office still prefer it over the agent?
A finance office values **strict determinism, auditability, zero risk of unexpected failure, zero LLM API cost, and instant speed**. The rule-based workflow guarantees that identical logic runs every single time, without the risk of an LLM loop getting stuck, misinterpreting discount formulas, or incurring cloud token costs.

### 3. The agent's steps can change between runs. What problems would that cause in a real product?
1. **Flaky testing and QA**: Regressions become difficult to isolate when execution trajectories vary probabilistically.
2. **Unpredictable costs & latency**: A query that took 1 tool step today might take 4 steps tomorrow, multiplying latency and token usage.
3. **Auditing & compliance hurdles**: In regulated domains, explaining why the agent chose a particular decision path becomes non-trivial.

### 4. Design a system that uses a workflow for common questions and an agent for the rest. Where would you draw the line?
Use a **hybrid router pattern**:
- **Layer 1 (Fast Deterministic Workflow)**: High-frequency, standardized intent patterns (e.g., exact course fee lookups, standard tuition fee schedules, invoice downloads) are routed directly to deterministic code or SQL queries.
- **Layer 2 (Agent Fallback / Escalation)**: If intent classification has low confidence or the user query involves complex composition (e.g., multi-course discount eligibility, budgeting constraints, unstructured comparisons), route the query to the tool-calling AI agent.

### 5. Which parts of agent.py are the LLM, the tools, and the loop?
- **LLM**: The OpenAI API call `client.chat.completions.create(model=MODEL, messages=messages, tools=TOOLS, temperature=0)`.
- **Tools**: The functions defined in `tools.py` (`get_course_fee`, `calculator`) mapped via `TOOL_FUNCTIONS` and described in the JSON Schema `TOOLS`.
- **Loop**: The `for step in range(1, max_steps + 1):` construct in `agent.py`, which implements the iterative Reason -> Act -> Observe cycle until the LLM stops requesting tool calls.

---

## 13. Viva Questions & Answers

### 1. What is the difference between a chatbot, a rule-based workflow, and an AI agent?
- **Chatbot**: Plain LLM mapping natural language input directly to text output without tool execution or access to external private data.
- **Rule-based workflow**: Hard-coded procedural logic (if/else, regex) written by a software engineer. Deterministic and fast, but completely rigid.
- **AI agent**: A system where an LLM is embedded within an iterative control loop (**Agent = LLM + Tools + Loop**). The LLM reasons about what tool to invoke, observes the tool output, and iterates until the goal is satisfied.

### 2. In agent.py, which lines are the LLM, which are the tools, and which is the loop?
- **LLM**: Step 1 (`response = client.chat.completions.create(...)`).
- **Loop**: `for step in range(1, max_steps + 1):` managing iterative turns.
- **Tools**: Step 3 (`function = TOOL_FUNCTIONS.get(name)` followed by `result = function(**arguments)`).

### 3. Who actually executes a tool: the LLM or your Python program?
**The Python program executes the tool.** The LLM only generates a structured text request (JSON specifying the tool name and arguments). The local Python runtime inspects the request, calls the actual Python function, obtains the return value, and feeds it back to the LLM.

### 4. Why does the agent need a max_steps limit?
To prevent **infinite loops** and runaway token consumption. If the LLM produces repetitive tool calls, encounters unresolvable errors, or fails to terminate, `max_steps` guarantees that execution halts safely.

### 5. Why does the calculator tool avoid Python's eval() function?
`eval()` executes arbitrary Python code and represents a severe **Arbitrary Code Execution (ACE) security vulnerability**. An LLM could generate malicious input such as `__import__('os').system('rmdir /s /q C:\\')`. Using Python's `ast` (Abstract Syntax Tree) ensures only safe, whitelisted arithmetic operations (`+`, `-`, `*`, `/`) can ever execute.

### 6. What is the purpose of the JSON Schema tool descriptions in tools.py?
The JSON Schema informs the LLM of the tool's name, purpose, argument types, and required parameters. The LLM's attention mechanism uses this metadata to decide whether a tool matches the user's intent and how to format arguments correctly.

### 7. Why do we use a virtual environment for each project?
To isolate project-specific dependencies and library versions (e.g., `openai`, `python-dotenv`) from the system-wide Python installation, preventing package version collisions between different projects.

### 8. Why is the API key kept in a .env file instead of inside config.py?
To prevent sensitive credentials and private API keys from being leaked or committed to public version control (e.g., GitHub). The `.env` file is kept local and ignored via `.gitignore`.

### 9. What does it mean that Ollama, Groq, and Hugging Face all provide an OpenAI-compatible API?
They all adhere to the standard REST endpoint specification defined by OpenAI (e.g., `POST /v1/chat/completions` with the standard `messages`, `tools`, and `temperature` payload format). This allows developers to swap the underlying LLM provider simply by changing `BASE_URL` and `API_KEY` without modifying application code.

---

## 14. Result

Thus, a Python development environment was set up in VS Code and connected to an open large language model, and a chatbot, a rule-based workflow, and an AI agent were implemented and compared on the same task. The observations show that **the plain LLM chatbot is fluent but hallucinates factual data; the rule-based workflow is completely reliable and deterministic for pre-programmed inputs but breaks entirely on new or rephrased queries; and the AI agent achieves the optimal balance by leveraging tool execution (retrieval and calculation) within an iterative Reason-Act-Observe loop, providing flexibility and high factual accuracy across both anticipated and open-ended queries.**
