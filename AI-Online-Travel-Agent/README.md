🎙️ AI Online Travel Agent — Proactive Voice Assistant
A voice-first travel assistant that proactively calls travelers when their trip is disrupted — and answers inbound calls with real trip data, live weather, and connecting flight details.
Built using Retell AI (voice), n8n (orchestration), and Google Sheets (data).

🎯 What It Does
Inbound Flow (traveler calls in)

Traveler calls a Retell AI phone number
n8n looks up their trip details from Google Sheets by caller ID
The voice agent greets them with their real airline, confirmation number, and connecting flight info
Agent can answer weather questions using live weather API data

Outbound Flow (system calls the traveler)

A disruption signal (webhook POST) triggers n8n
n8n looks up the affected traveler in Google Sheets
Retell AI initiates an outbound call to the traveler
Agent proactively explains what changed and guides next steps


🏗️ Architecture
Inbound:  Phone Call → Retell AI → n8n Webhook → Google Sheets → Dynamic Variables → Voice Agent

Outbound: Disruption Webhook → n8n → Google Sheets → Retell API → Outbound Call → Voice Agent

Weather:  Caller asks → Retell Custom Function → n8n → Nominatim + Open-Meteo → Voice Agent

🛠️ Tech Stack
ComponentToolVoice (STT/TTS)Retell AIOrchestrationn8n CloudDataGoogle SheetsGeocodingNominatim (OpenStreetMap)WeatherOpen-Meteo APIPhone NumberRetell AI

📁 Files in This Repo
FileDescriptionRetell Inbound.jsonn8n workflow — handles inbound calls, looks up traveler by phoneRetell Outbound.jsonn8n workflow — triggered by disruption, initiates outbound callWeather Tool.jsonn8n workflow — fetches live weather for any citybuild_log.mdFull build log with all steps, bugs, and lessons learnedREADME.mdThis file

🗂️ Google Sheet Structure
Sheet tab: Passenger list
ColumnDescriptionphoneCaller phone number (no + prefix, e.g. 17037312723)airlineMain airline nameconfirmationBooking confirmation codelast_name_expectedLast name for security verificationhas_connectionyes or noconnecting_airlineConnecting flight airlineconnecting_confirmationConnecting flight confirmationconnection_cityConnection city nameconnection_updatePlain text disruption update for the connection

🔄 n8n Workflows
1. Retell Inbound
Webhook path: POST /retell-inbound
Triggered by Retell when a call comes in. Looks up the caller by phone number in Google Sheets and returns dynamic variables to personalize the conversation.
Flow: Webhook → Get row(s) in sheet → Respond to Webhook
2. Retell Outbound
Webhook path: POST /retell-outbound
Triggered by a disruption signal. Looks up the traveler by confirmation number and initiates an outbound call via Retell API.
Flow: Webhook → Get row(s) in sheet → HTTP Request (Retell create-phone-call) → Respond to Webhook
3. Weather Tool
Webhook path: POST /weather-tool
Called by Retell as a Custom Function when the caller asks about weather. Geocodes the city then fetches live weather data.
Flow: Webhook → Get Coordinates (Nominatim) → Get Weather (Open-Meteo) → Respond to Webhook

⚙️ Setup Instructions
Prerequisites

n8n Cloud account
Retell AI account + purchased phone number
Google account with Google Sheets
Retell API Key (for outbound calls)

Steps
1. Google Sheets

Create a sheet with the columns listed above
Format the phone column as Plain Text (no + prefix)
Share with the Google account used in n8n

2. n8n

Import all three workflow JSON files
Set up Google Sheets OAuth2 credential
Add Retell API key as Authorization Bearer header in the Outbound workflow
Activate all three workflows
Copy Production webhook URLs for each workflow

3. Retell AI

Create a Conversation Flow voice agent
Set global prompt (see below)
Add get_weather as a Custom Function pointing to Weather Tool Production URL
Assign agent to your purchased phone number
Paste Retell Inbound Production URL as Inbound webhook on the phone number


🤖 Retell Agent Prompt
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
- Keep replies brief — usually 1 to 3 short sentences
- Never invent confirmation codes, flight times, or refund amounts
- If they are upset, acknowledge once, then move to helpful next steps

Style: Calm, confident, and human. Repeat important facts clearly.

When the caller asks about weather, use the get_weather tool with the city name.
Never answer weather questions from your own knowledge — always call the tool.

🌤️ Weather Custom Function
Function name: get_weather
Parameter schema:
json{
  "type": "object",
  "properties": {
    "city": {
      "type": "string",
      "description": "The destination city to get weather for"
    }
  },
  "required": ["city"]
}

⚠️ Known Issues & Notes

Retell KYC verification required before outbound calls work
Phone numbers in Google Sheets must NOT have a + prefix
n8n Cloud IP is blocked by Open-Meteo geocoding API — use Nominatim instead
Always use Production webhook URLs in Retell — not n8n test URLs
After Retell agent changes, always click Publish


🔮 Roadmap

 RAG for airline, hotel, and car rental policies
 Mock flight status API integration
 Human escalation / handoff flow
 Session memory after each call
 Amadeus API for real flight status polling
