# Canada Travel Advisory Multi-Agent Chatbot

A Flask-based multi-agent assistant for Canadian travel information with three specialist workflows:
- Travel advisory Q&A (RAG over Government of Canada travel pages)
- Checking vaccine requirement
- Visa requirement guidance
- Simulated appointment booking

## Repository Structure

- `app.py`: Flask app and API routes
- `static/index.html`: Frontend chat UI
- `prompts/`: System prompts used by supervisor and specialists
- `LLM_project.ipynb`: Data crawl + parsing + vector DB build workflow
- `agents.ipynb`: Agent orchestration and evaluation notebook
- `parsed_pages.json`: Parsed advisory page data
- `bookings.json`: Simulated booking state
- `chroma_db_canada_travel/`: Persisted Chroma vector DB

## Prerequisites

- Python 3.12+
- pip

## Setup

1. Create and activate a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Create environment file:

```bash
cp .env.example .env
```

4. Edit `.env` and set at least:
- `OPENAI_API_KEY`
- `QWEN_API_BASE_URL` (default in template already points to the project endpoint)

## Run the App

```bash
python app.py
```

Open http://localhost:5001

Frontend demo gate value is `demo` (for UX only, not real security).

## API Endpoints

- `POST /api/chat`: Main chat endpoint
- `GET /api/available-slots?date=YYYY-MM-DD`: Available booking slots
- `POST /api/save-booking`: Save simulated booking
- `GET /api/bookings`: List simulated bookings

## External APIs and Data Sources

- Government of Canada Travel Advice and Advisories (`travel.gc.ca`) is the primary knowledge source used to build the retrieval corpus.
- REST Countries API (`https://restcountries.com/v3.1/name`) is used in the notebook agent workflow for read-only country-name validation during typo handling.
- LLM inference is called through the configured OpenAI-compatible endpoint in `QWEN_API_BASE_URL`.

## Agent Robustness

- Typo-resilient destination handling: API-first country validation, then conservative fuzzy recovery and confirmation prompts when confidence is high.
- Guardrail-aware supervision: explicit handling for prompt-injection attempts, ambiguity clarification, temporal warnings, and multi-turn continuity.
- Conservative reliability behavior: partial-context responses, explicit no-fabrication fallbacks, and redirection to official sources when coverage is incomplete.
- Evaluation discipline: structured 18-case test set covering routing correctness, task correctness, guardrail compliance, and multi-turn completion.
- Realistic simulation UX for booking: business-hour slot logic, duplicate prevention, and deterministic simulated reference IDs.

## Notebook Workflow

### 1) Build or refresh travel data and vectors

Use `LLM_project.ipynb` when you need to:
- Crawl/update advisory pages
- Regenerate `parsed_pages.json`
- Rebuild `chroma_db_canada_travel/`

### 2) Agent orchestration and evaluation

Use `agents.ipynb` to:
- Run supervisor + specialist routing logic
- Execute evaluation experiments and interactive tests

## Public Repository Safety Checklist

Before submitting a public repo:

1. Keep secrets out of git:
- Never commit `.env`
- Commit only `.env.example` with placeholders

2. Remove hardcoded credentials/passwords:
- No API keys, tokens, or private passwords in source, notebooks, or frontend scripts

3. Clean notebook outputs before publish:
- Clear outputs to avoid leaking local paths or environment details

4. Verify tracked files before push:

```bash
git status
git diff --staged
```

5. Run a quick secret-pattern scan:

```bash
git grep -nE '(api[_-]?key|token|secret|password|sk-[A-Za-z0-9]{20,})'
```

If any real key was ever committed earlier, rotate it before making the repo public.
