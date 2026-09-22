# Agentic AI: Foundations and Open-Source Practice
## Day 1 Lab: Python in VS Code: Chatbot vs Rule-Based Workflow vs AI Agent

### 📁 Project Structure (`day1_lab/`)
```
day1_lab/
├── .env                <- API keys & model configuration (Ollama, Groq, or Hugging Face)
├── .gitignore          <- Ignores .env and .venv/
├── requirements.txt    <- Dependencies (openai, python-dotenv)
├── config.py           <- Shared configuration, LLM client, and private college data
├── check_setup.py      <- Step 1: Tests provider connection and model response
├── chatbot.py          <- System 1: Plain LLM chatbot (demonstrates hallucination)
├── workflow.py         <- System 2: Rule-based workflow (demonstrates reliability & rigidity)
├── tools.py            <- Fee lookup and safe AST calculator tools
├── agent.py            <- System 3: Tool-using AI agent with Reason-Act-Observe loop
├── challenge.py        <- Edge-case test comparing workflow vs agent
├── LAB_REPORT.md       <- Complete Observations table, Discussion & Viva answers, and Result
└── .venv/              <- Python virtual environment
```

---

### 🚀 How to Run

#### 1. Activate the Virtual Environment
Open PowerShell inside `day1_lab`:
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

#### 2. Choose LLM Provider in `day1_lab\.env`
Choose **one** option in `.env`:
- **Option A (Ollama - Local)**:
  Install Ollama and run `ollama run qwen2.5:1.5b`.
  ```env
  PROVIDER=ollama
  MODEL=qwen2.5:1.5b
  ```
- **Option B (Groq - Free Cloud Key)**:
  Sign up at [console.groq.com](https://console.groq.com) and create a free key:
  ```env
  PROVIDER=groq
  GROQ_API_KEY=gsk_your_actual_key_here
  MODEL=openai/gpt-oss-20b
  ```
- **Option C (Hugging Face - Free Token)**:
  ```env
  PROVIDER=huggingface
  HF_TOKEN=hf_your_actual_token_here
  MODEL=openai/gpt-oss-20b
  ```

#### 3. Execute the Programs
```powershell
# Step 1: Verify model connection
python check_setup.py

# System 1: Plain Chatbot
python chatbot.py

# System 2: Rule-Based Workflow (verified deterministic)
python workflow.py

# Safe tools verification
python tools.py

# System 3: AI Agent
python agent.py

# Extra challenge query
python challenge.py
```

All observations, discussion question answers, and viva solutions are documented in [`day1_lab/LAB_REPORT.md`](file:///c:/Users/praneeshr/OneDrive/Desktop/ai%20day%201/day1_lab/LAB_REPORT.md).
