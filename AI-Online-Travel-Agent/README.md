# 🎙️ AI Online Travel Agent — Proactive Voice Assistant

A voice-first travel assistant that proactively calls travelers when their trip is disrupted — and answers inbound calls with real trip data, live weather, and connecting flight details.

Built using **Retell AI** (voice), **n8n** (orchestration), and **Google Sheets** (data).

---

## 🎯 What It Does

### Inbound Flow (traveler calls in)
- Traveler calls a Retell AI phone number
- n8n looks up their trip details from Google Sheets by caller ID
- The voice agent greets them with their real airline, confirmation number, and connecting flight info
- Agent can answer weather questions using live weather API data

### Outbound Flow (system calls the traveler)
- A disruption signal (webhook POST) triggers n8n
- n8n looks up the affected traveler in Google Sheets
- Retell AI initiates an outbound call to the traveler
- Agent proactively explains what changed and guides next steps

---

## 🏗️ Architecture

```
Inbound:  Phone Call → Retell AI → n8n Webhook → Google Sheets → Dynamic Variables → Voice Agent

Outbound: Disruption Webhook → n8n → Google Sheets → Retell API → Outbound Call → Voice Agent

Weather:  Caller asks about weather → Retell Custom Function → n8n → Nominatim + Open-Meteo → Voice Agent
```

---

## 🛠️ Tech Stack

| Component | Tool |
|-----------|------|
| Voice (STT/TTS) | Retell AI |
| Orchestration | n8n Cloud |
| Data | Google Sheets |
| Geocoding | Nominatim (OpenStreetMap) |
| Weather | Open-Meteo API |
| Phone Number | Retell AI |

---

## 📁 Files in This Repo

| File | Description |
|------|-------------|
| `Retell Inbound.json` | n8n workflow — handles inbound calls, looks up traveler by phone |
| `Retell Outbound.json` | n8n workflow — triggered by disruption, initiates outbound call |
| `Weather Tool.json` | n8n workflow — fetches live weather for any city |
| `build_log.md` | Full build log with all steps, bugs, and lessons learned |
| `README.md` | This file |

---

## 🗂️ Google Sheet Structure

Sheet tab: **Passenger list**

| Column | Description |
|--------|-------------|
| phone | Caller phone number (no + prefix, e.g. 17037312723) |
| airline | Main airline name |
| confirmation | Booking confirmation code |
| last_name_expected | Last name for security verification |
| has_connection | yes or no |
| connecting_airline | Connecting flight airline |
| connecting_confirmation | Connecting flight confirmation |
| connection_city | Connection city name |
| connection_update | Plain text disruption update for the connection |

---

## 🔄 n8n Workflows

### 1. Retell Inbound
**Webhook path:** `POST /retell-inbound`

Triggered by Retell when a call comes in. Looks up the caller by phone number in Google Sheets and returns dynamic variables to personalize the conversation.

**Flow:** Webhook → Get row(s) in sheet → Respond to Webhook

### 2. Retell Outbound
**Webhook path:** `POST /retell-outbound`

Triggered by a disruption signal. Looks up the traveler by confirmation number and initiates an outbound call via Retell API.

**Flow:** Webhook → Get row(s) in sheet → HTTP Request (Retell create-phone-call) → Respond to Webhook

### 3. Weather Tool
**Webhook path:** `POST /weather-tool`

Called by Retell as a Custom Function when the caller asks about weather. Geocodes the city then fetches live weather data.

**Flow:** Webhook → Get Coordinates (Nominatim) → Get Weather (Open-Meteo) → Respond to Webhook

---

## ⚙️ Setup Instructions

### Prerequisites
- n8n Cloud account
- Retell AI account + purchased phone number
- Google account with Google Sheets
- Free APIs: Nominatim (no key needed), Open-Meteo (no key needed)
- Retell API Key (for outbound calls)

### Steps

**1. Google Sheets**
- Create a sheet with the columns listed above
- Add at least one test row with a real phone number
- Format the phone column as Plain Text (no + prefix)
- Share with the Google account used in n8n

**2. n8n**
- Import all three workflow JSON files
- Set up Google Sheets OAuth2 credential
- Add Retell API key as Authorization header in the Outbound workflow
- Activate all three workflows
- Copy the Production webhook URLs for each workflow

**3. Retell AI**
- Create a Conversation Flow voice agent
- Set global prompt (see Retell Agent Prompt section below)
- Add `get_weather` as a Custom Function pointing to the Weather Tool Production URL
- Assign agent to your purchased phone number
- Paste the Retell Inbound Production URL as the Inbound webhook on the phone number

---

## 🤖 Retell Agent Prompt

```
You are an AI phone agent for an online travel agency. You are calling the traveler 
proactively because their trip may be affected (for example: a flight delay, 
cancellation, or a hotel issue).

You have access to the following trip details:
- Main airline: {{airline}}
- Confirmation number: {{confirmation_number}}
- Has connecting flight: {{has_connection}}
- Connecting airline: {{connecting_airline}}
- Connecting confirmation: {{connecting_confirmation}}
- Connection city: {{connection_city}}
- Connection update: {{connection_update}}

If has_connection is "yes", proactively mention the connecting flight status 
using connection_update after explaining the main flight situation.

Your goals:
- Clearly explain what changed in plain language, without causing panic
- Say what it means for their trip in practical terms
- Ask one short question at a time
- Keep replies brief — usually 1 to 3 short sentences
- Never invent confirmation codes, flight times, or refund amounts
- If they are upset, acknowledge once, then move to helpful next steps

Style: Calm, confident, and human. Repeat important facts clearly.

When the caller asks about weather at their destination, use the get_weather 
tool with the destination city name. Never answer weather questions from your 
own knowledge — always call the tool.
```

---

## 🌤️ Weather Custom Function

**Function name:** `get_weather`

**Parameter schema:**
```json
{
  "type": "object",
  "properties": {
    "city": {
      "type": "string",
      "description": "The destination city to get weather for"
    }
  },
  "required": ["city"]
}
```

**URL:** Your Weather Tool n8n Production webhook URL

---

## ⚠️ Known Issues & Notes

- Retell KYC verification required before outbound calls work — contact support@retellai.com
- Phone numbers in Google Sheets must NOT have a + prefix (sheets treats + as a formula)
- n8n Cloud IP is blocked by Open-Meteo geocoding API — use Nominatim instead
- Always use Production webhook URLs in Retell — not the n8n test URLs
- After any Retell agent changes, click Publish for changes to take effect on live calls

---

## 📖 Build Log

See `build_log.md` for a full step-by-step record of how this was built, including all bugs encountered and how they were fixed. Great resource for anyone learning n8n and Retell AI.

---

## 🔮 Roadmap

- [ ] RAG for airline, hotel, and car rental policies
- [ ] Mock flight status API integration
- [ ] Human escalation / handoff flow
- [ ] Session memory (save transcript summary after each call)
- [ ] Amadeus API integration for real flight status polling


**Screenshots: **

One time ingestion of travel policy docs into Supabase (RAG) and storing as vectors:
<img width="1820" height="905" alt="image" src="https://github.com/user-attachments/assets/30e15e21-0e0e-4088-92de-9dac74a5475d" /> 

Supabase : vector DB
<img width="1862" height="615" alt="image" src="https://github.com/user-attachments/assets/c16ea8b3-a1af-4acc-a625-81b41bbe5159" />

Supabase - vector DB
<img width="1863" height="633" alt="image" src="https://github.com/user-attachments/assets/287132c7-b9b1-45ed-afc9-d0e796369c68" />

Adding the upload of a supplemental travel policy pdf, with chunking strategy and vector conversions of pdf:
<img width="1814" height="888" alt="image" src="https://github.com/user-attachments/assets/f9d85a3d-dc17-4f9b-9706-449327d743b2" />



