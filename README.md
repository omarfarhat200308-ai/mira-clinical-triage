# MIRA — AI Clinical Triage for Rural India

> **Origin:** This project was originally built as **TABIB** at a hackathon in June 2026, and was later renamed and rebuilt into MIRA's 5-node pipeline.

> Multi-agent diagnostic coordination system for ASHA workers 
> and rural health volunteers. Built on Band's agent coordination 
> layer with Claude API.

**Track 3: Regulated & High-Stakes Workflows**
Band of Agents Hackathon 2026

---

## The Problem

India has 900,000+ ASHA (Accredited Social Health Activist) workers 
serving rural communities where the nearest doctor is hours away. 
When a patient presents with fever, rash, or chest pain, ASHA workers 
have no decision support. They guess. Patients die from delayed 
referrals for treatable conditions like dengue hemorrhagic fever.

MIRA (originally TABIB) puts a clinical triage system in their hands — in Hindi and 
English — with zero training required.

---

## What It Does

A user describes a patient's symptoms in plain language. Three 
specialized AI agents coordinate in real time to produce a 
structured clinical decision:

**Decision options:**
- ✅ MONITOR AT HOME — safe, with instructions
- ⚠️ REFER TODAY — needs facility care within hours  
- 🚨 REFER NOW — DO NOT DELAY — emergency escalation

---

---

## Architecture — this repository (TABIB v1, 3 agents via Band)

Each WhatsApp message passes through three Claude-powered agents. Every agent
is a single `anthropic` Messages API call (`claude-sonnet-4-5`) with a prompt
from `shared_prompts/prompts.py`.

| # | Agent | File | Input → Output |
|---|---|---|---|
| 1 | **Intake (TIA)** | `src/intake_agent.py` | Raw WhatsApp text (English / Hindi / Telugu / mixed) → structured patient JSON: age, sex, symptoms, duration, vitals, pregnancy status, known conditions, language. Missing fields stay `null`, nothing is guessed. |
| 2 | **Diagnostic (TDA)** | `src/diagnostic_agent.py` | Intake JSON → top-3 differential, red flags, missing critical info, immediate ASHA actions, notes for the PHC doctor. Framed as pattern-flagging for a doctor, not diagnosis. Guided by NHM and IMNCI. |
| 3 | **Triage (TTA)** | `src/triage_agent.py` | Intake + diagnostic → the final WhatsApp reply: `Decision:` (REFER NOW / REFER TODAY / MONITOR AT HOME), next steps, warning signs, a summary line for the doctor. Replies in the patient's language. |

```
WhatsApp (Twilio) → FastAPI /webhook → BandOrchestrator
        ├─ Band room per phone number: @TDA mention → Band agents
        │    → poll for a message containing "Decision:" (25 s timeout)
        └─ fallback: inline Intake → Diagnostic → Triage via Anthropic
                      (src/band_coordinator.py)
→ reply sent back over WhatsApp
```

- **Band agent loop** (`src/band_orchestrator.py`): one persistent Band room
  per patient phone number, TDA/TTA added as participants, completion
  detected by the `Decision:` keyword. The result is tagged
  `[path=band_agent_loop]`.
- **Inline fallback**: if Band is not configured or the loop times out, the
  same three agents run in-process, tagged `[path=inline_fallback]`. The
  patient always gets an answer.
- **WhatsApp bridge** (`src/band_whatsapp_bridge.py`, `src/webhook.py`):
  FastAPI webhook with Twilio send/receive and a `/` health check that
  reports Band reachability.
- Design notes and the Band SDK API findings are in `TIER3_PLAN.md`.

## MIRA — the 5-node pipeline

After the hackathon the project was renamed MIRA and rebuilt as a fixed,
auditable pipeline. That codebase is a separate repository and is not part of
this one. Its five agent stages are:

| # | Node | Role |
|---|---|---|
| 1 | **Intake** | Extracts structured symptoms. Refuses to continue without age, sex and the primary symptom's duration or severity, and asks a clarifying question instead. |
| 2 | **Language** | Detects the patient's language and maps symptoms to a canonical tag vocabulary at the start, then translates the result back at the end. English, Hindi, Marathi, Tamil, Telugu, Gujarati. |
| 3 | **Diagnostic** | A bounded differential from a fixed taxonomy, used only to estimate urgency. Grounded in a retrieval layer over IMNCI / NHM / WHO CHW protocol text. Never patient-facing. |
| 4 | **Triage** | A hardcoded red-flag rule engine runs first, and any hit forces EMERGENCY regardless of the LLM. Otherwise the LLM's grounded assessment is used, escalated one band up if confidence is low. |
| 5 | **Referral** | Maps the triage level to a facility type by deterministic lookup, with next steps passed through a drug-name/dosage filter. |

MIRA also adds a romanization normalizer for Latin-script Telugu/Hindi, and a
doctor-handoff WhatsApp message on emergency or urgent cases. It is described
as a verifiable workflow automator with LLM reasoning at bounded checkpoints,
not as autonomous agents.

## What we used

| Layer | Tech |
|---|---|
| LLM | Anthropic Claude (`claude-sonnet-4-5` in this repo) via the `anthropic` SDK |
| Agent coordination | Band (`band-sdk[anthropic]`): chat rooms, @mentions, agent REST API |
| Messaging | WhatsApp through Twilio |
| API server | FastAPI |
| Config | `python-dotenv` (keys live in `.env`, which is gitignored) |
| Languages | English, Hindi, Telugu |
| Guidelines | NHM and IMNCI (prompt guidance in this repo) |

## Setup

```bash
pip install fastapi uvicorn anthropic twilio python-dotenv python-multipart "band-sdk[anthropic]"
# create .env in the repo root with the variables below
cd src && uvicorn webhook:app --port 8000
```

Environment variables: `ANTHROPIC_API_KEY`, `TWILIO_ACCOUNT_SID`,
`TWILIO_AUTH_TOKEN`, `TWILIO_WHATSAPP_FROM`, `BAND_AGENT_API_KEY`,
`BAND_AGENT_ID`, and optionally `BAND_TDA_ID` / `BAND_TTA_ID` to enable the
Band agent loop. Without the Band variables the inline fallback is used.

Tests: `src/test_band_orchestrator.py`, `src/test_whatsapp_bridge.py`,
`src/test_tabib.py`.

## Scope and limits

This is decision support for trained ASHA workers and PHC staff, not a
diagnostic tool and not a substitute for medical advice. It has not been
clinically validated.

## History

Built as TABIB at the Band of Agents Hackathon (Track 3: Regulated &
High-Stakes Workflows), June 2026. The git history was rewritten to remove
credentials that were committed early on. Those keys were rotated.
