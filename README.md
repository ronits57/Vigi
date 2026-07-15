# Vigi — LLM Prompt Shield 

Vigi is a production‑ready prompt shielding web app and API that makes LLM interactions safer. It detects and blocks prompt injection, screens for harmful content, and automatically redacts PII on both input and output—complete with live traces, structured logs, and a clean chat UI. 

This project is designed for fast demos and real deployments (Cloud Run). It favors simple, auditable code and policy‑driven behavior over black boxes.

Highlights
- End‑to‑end shielding pipeline with an LLM‑aware strategy system (ml | heuristic | llm | hybrid)
- Chat UI that shows a step‑by‑step backend “Flow” trace for transparency
- Structured event logging (EVENT_JSON) and lightweight APIs for policy and logs
- Cloud‑native ready: Procfile + gunicorn + Buildpacks (no Dockerfile needed)
- Sensible defaults: LLM‑only mode, model loads skipped when unnecessary, safe fallbacks when no key is present

---

## Table of contents
- Overview and architecture
- Quick start (local)
- API reference
- Policy configuration
- Observability and logs
- Frontend UX tips
- Deploy to Google Cloud Run (Buildpacks)
- Security & privacy notes
- Extending Vigi
- How it meeds the criteria for an LLM Gaurd

---

## Overview and architecture

Core technologies
- Backend: Flask (Python)
- LLM: Google Gemini via `google-generativeai`
- NLP: spaCy (optional NER PII redaction)
- UI: Single‑page HTML/CSS/JS (templates/test.html)

High‑level flow

```
User Prompt
	 ↓
Harmful content check  ──┐   (ml / heuristic / llm / hybrid)
	 ↓ allow/block           ├─> Structured event logs (EVENT_JSON)
Prompt injection check  ──┘
	 ↓ allow/block
PII redaction (input)
	 ↓
LLM generation (Gemini)
	 ↓
Optional response screening (harmful + PII)
	 ↓
Response + trace to UI
```

Key files
- `app.py` — Flask app, `/shield_prompt`, `/api/policy`, `/api/logs`, UI route
- `detectors.py` — Orchestrates detection/redaction with configurable strategies
- `llm_detectors.py` — Gemini‑backed helpers (soft dependency, safe no‑op if no key)
- `policy.json` — Toggle modules, pick strategies, thresholds, LLM model
- `templates/test.html` — Chat UI + Live “Flow” panel, responsive and animated
- `smoke_test.py` — Minimal safety checks to validate behavior

---

## Quick start (local)

Prerequisites
- Python 3.10+
- A Gemini API key (create a `.env` with `GEMINI_API_KEY=...`)

Setup

```powershell
# From the project root
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt

# Optional but recommended if you want spaCy NER redaction locally
python -m spacy download en_core_web_sm

# Create .env with your key
"GEMINI_API_KEY=YOUR_KEY_HERE" | Out-File -Encoding ascii -NoNewline .env

# Run (dev server)
python app.py
# Opens on http://127.0.0.1:8080 (binds to 0.0.0.0 and respects PORT)
```

Sanity test

```powershell
python smoke_test.py
```

Behavioral optimizations
- When harmful content strategy is `llm`, the app skips loading the local model for faster cold starts.
- Redaction always includes a light regex pass for emails/phones even if spaCy isn’t installed.

---

## Observability and logs

Event shape (examples)
- `BLOCK` (harmful or injection)
- `REDACT` (PII on input or output)
- `SUCCESS` (allowed flow)

Every event includes a UTC timestamp, a preview of the LLM response (safe truncated), and metadata for quick debugging.

---

## Frontend UX tips

- The landing page is a single responsive HTML template with:
	- Chat pane (left) and a Flow/Raw JSON pane (right) with a draggable splitter
	- Scroll‑aware navbar that subtly shrinks and returns to base at top
	- Optional custom cursor (white dot) for a premium feel
	- Smooth GSAP reveals where appropriate, respecting reduced‑motion settings

Keyboard shortcuts
- Ctrl/Cmd + Enter to send

## Security & privacy notes

- Never commit `.env`; use Secret Manager in production.
- LLM responses and prompts are logged only with small, truncated previews (not full payloads by default).
- PII redaction is conservative and layered: regex + spaCy (if installed) + optional LLM pass in `llm`/`hybrid`.
- Public assets never contain secrets. A placeholder file documents the risk.


## How this meets the needs of a model armor

- Impact
	- Tackles core safety issues (injection, toxicity, PII) with an explainable pipeline suitable for real apps.
- Technical quality
	- Policy‑driven architecture; LLM‑aware strategies with short‑circuiting; production‑grade serving via gunicorn.
- UX & transparency
	- Clear chat demo, live flow traces, JSON panel, copy‑to‑clipboard, subtle motion.
- Reliability
	- Smoke tests; safe fallbacks if spaCy or keys are missing; model loads skipped in LLM‑only mode to speed startup.
- Deployability
	- Cloud Run ready with Buildpacks + Procfile; secrets via Secret Manager; CI trigger recommended.

---

## License

Copyright © Ronit Sherawat

---

## Maintainers

Team Vigi — Anshika Gupta and Ronit Sherawat
