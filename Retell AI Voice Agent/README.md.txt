# Retell AI Voice Agent — Appointment Reminder

## Overview

This project automates **outbound appointment reminder calls** using a Retell AI voice agent triggered by an N8N workflow. When a new calendar event is detected, the system automatically extracts appointment details, and places a personalized phone call to the client using a conversational AI voice agent named **Maren**.

Built by **Sri Adidam** as a practical AI automation for managing client consultations.

---

## How It Works
```
Google Calendar → N8N Schedule Trigger → GPT-4.1-mini (extract fields) → Retell AI API (outbound call)
```

1. **N8N** checks Google Calendar on a schedule
2. **GPT-4.1-mini** extracts structured data (name, phone, reason, time) from the event description
3. **Retell AI** places an outbound call using a pre-configured voice agent
4. The voice agent **Maren** confirms the appointment with the client conversationally

---

## Agent Behavior

**Voice:** Maren (Retell AI built-in voice)  
**Model:** GPT-4.1  
**Language:** English (US)  
**Channel:** Voice (outbound phone call)

### Personality & Style
- Confident, professional, and friendly
- Conversational — not robotic or overly formal
- Concise and clear
- Does **not** reschedule or cancel appointments — not authorized

### Call Flow

| Step | Action |
|------|--------|
| 1 | Confirm identity — *"Hi, am I speaking with {{name}}?"* |
| 2 | Introduce as Maren, Sri's Personal Assistant |
| 3 | State appointment reason and time in human-friendly format |
| 4 | Ask for confirmation — *"Does that time still work for you?"* |
| 5 | End call gracefully on confirmation or if contact is unavailable |

### Dynamic Variables Passed at Runtime
| Variable | Description |
|----------|-------------|
| `{{name}}` | Client's full name |
| `{{phone_number}}` | Client's phone number |
| `{{reason}}` | Purpose of the appointment |
| `{{start_time}}` | Start time (converted from ISO to human-friendly format) |
| `{{end_time}}` | End time (converted from ISO to human-friendly format) |

---

## Files in This Folder

| File | Description |
|------|-------------|
| `n8n-workflows/Sri-n8n_Voice_Agent_Flow.json` | N8N workflow — schedule trigger, calendar fetch, GPT extraction, Retell API call |
| `retell-agent/Sri_s_retell_ai_first_voice_agent.json` | Retell AI agent config — voice, prompt, tools, LLM settings |
| `screenshots/` | Screenshots of the N8N workflow and Retell AI agent setup |
| `README.md` | This file |

---

## Setup Instructions

### Prerequisites
- [N8N](https://n8n.io/) instance (self-hosted or cloud)
- [Retell AI](https://retellai.com/) account with an agent configured
- Google Calendar OAuth2 credentials in N8N
- OpenAI API key in N8N
- A Retell AI API key

### Step 1 — Import the N8N Workflow
1. Open your N8N instance
2. Go to **Workflows → Import from File**
3. Upload `Sri-n8n_Voice_Agent_Flow.json`
4. Reconnect credentials:
   - **Google Calendar** — connect your Google OAuth account
   - **OpenAI** — add your OpenAI API key
   - **HTTP Request (Retell)** — add your Retell AI API key as a custom auth header: `Authorization: Bearer YOUR_API_KEY`

### Step 2 — Import the Retell AI Agent
1. Log into [Retell AI](https://retellai.com/)
2. Go to **Agents → Import Agent**
3. Upload `Sri_s_retell_ai_first_voice_agent.json`
4. Verify the agent prompt and dynamic variables are intact
5. Publish the agent

### Step 3 — Update the N8N HTTP Request Node
In the **HTTP Request** node, update these values:
- `from_number` — your Retell AI outbound phone number
- `override_agent_id` — your Retell AI agent ID (found in the agent settings)

### Step 4 — Add Appointment Data to Google Calendar
Each calendar event description should include:
```
Name: John Smith
Email: john@example.com
Phone: +1XXXXXXXXXX
Reason: AI Strategy Consultation
```

### Step 5 — Activate the Workflow
Toggle the workflow to **Active** in N8N. It will now run on the configured schedule and place calls automatically.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| [N8N](https://n8n.io/) | Workflow automation |
| [Google Calendar API](https://developers.google.com/calendar) | Appointment source |
| [OpenAI GPT-4.1-mini](https://openai.com/) | Structured data extraction from event descriptions |
| [Retell AI](https://retellai.com/) | Voice agent + outbound calling |

---

## Notes & Limitations

- The agent **cannot** reschedule or cancel appointments — by design
- Timestamps from Google Calendar are automatically converted to human-readable format by the agent prompt
- The workflow processes **all** upcoming events on each run — consider adding a filter node to limit to events within the next 24–48 hours
- Keep your Retell API key and OpenAI key out of version control — use N8N's credential manager

---

## Author

**Sri Adidam**  
Built as part of an AI automation portfolio project.
