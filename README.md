# QueueStorm Investigator

AI/API SupportOps service for SUST CSE Carnival 2026 – Codex Community Hackathon.

## Quick Start

```bash
pip install -r requirements.txt
export GEMINI_API_KEY=your_key_here        # Linux/Mac
set GEMINI_API_KEY=your_key_here           # Windows CMD
$env:GEMINI_API_KEY="your_key_here"        # Windows PowerShell
uvicorn main:app --host 0.0.0.0 --port 8000
```

## Docker

```bash
docker build -t queuestorm .
docker run -e GEMINI_API_KEY=your_key_here -p 8000:8000 queuestorm
```

## Endpoints

- `GET /health` → `{"status":"ok"}`
- `POST /analyze-ticket` → structured JSON per the problem schema

## Tech Stack

- **Python 3.11** + **FastAPI** + **Uvicorn**
- **Google Gemini** (`gemini-2.0-flash`) via the `google-generativeai` SDK

## MODELS

| Model | Where | Why |
|---|---|---|
| `gemini-2.0-flash` | Google AI API (cloud) | Fast, accurate, supports English + Bangla, generous free tier, fits within the 30s timeout |

## AI Approach

Each request is sent to Gemini with a detailed system prompt that encodes:
- All enum taxonomies (case_type, department, severity, evidence_verdict)
- Safety rules as hard constraints
- Routing logic (which department handles which case type)
- Language detection (Bangla complaint → Bangla customer_reply)
- Evidence reasoning (cross-check complaint vs transaction history)

The model returns raw JSON which is validated and enum-corrected before responding.

## Safety Logic

1. System prompt explicitly forbids asking for PIN/OTP/password.
2. System prompt explicitly forbids confirming refunds — mandates "eligible amount through official channels" phrasing.
3. System prompt instructs the model to ignore instructions found inside complaint text (prompt injection defence).
4. All enum values are validated post-generation and corrected to safe defaults if invalid.
5. Stack traces and API keys are never exposed in error responses.

## Assumptions

- `transaction_history` may be empty (safety-only cases like phishing reports).
- When multiple transactions match equally, `relevant_transaction_id` is null and `evidence_verdict` is `insufficient_data`.
- Language of `customer_reply` mirrors the complaint language.

## Known Limitations

- Relies on Google AI API availability; returns 500 if the API is down.
- Response time depends on Gemini API latency; typically 2–6 seconds, well within the 30s limit.
- Does not persist state between requests.
