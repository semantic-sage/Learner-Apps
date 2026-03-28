# 🧳 AI Online Travel Agent — Voice-First Trip Disruption Assistant

A voice-first AI travel agent (Call **1 320 361 3193**) that answers inbound calls with real trip data, live weather, connecting flight details, and RAG-powered travel policy answers (airline cancellation, baggage, and more).

Built as a learning project to explore multi-tool agentic orchestration using no-code/low-code platforms.

---

## 🏗️ Architecture Overview

```
Inbound Call
    │
    ▼
Retell AI (Voice Agent)
    │
    ├──► n8n Workflow 1: Inbound Call Handler
    │         └── Google Sheets lookup by caller phone number
    │               └── Returns dynamic variables to Retell AI
    │
    ├──► n8n Workflow 2: Live Weather Tool  [Custom Function]
    │         └── Geocoding (Nominatim / OpenStreetMap)
    │               └── Open-Meteo API → weather summary
    │
    └──► n8n Workflow 3: RAG Policy Lookup  [Custom Function]
              └── OpenAI Embeddings → Supabase Vector Store
                    └── Semantic search → policy text → Retell AI
```

**Stack:**
| Layer | Tool |
|---|---|
| Voice & Agent | Retell AI (Conversational Flow) |
| Orchestration | n8n |
| Caller Data | Google Sheets |
| Weather | Nominatim (geocoding) + Open-Meteo API |
| RAG Embeddings | OpenAI Embeddings API (text-embedding-ada-002) |
| Vector Store | Supabase (pgvector) |
| LLM | GPT-4.1 (via Retell AI) |

---

## ✨ What It Does

1. **Traveler calls** a Retell AI phone number
2. **n8n looks up** their trip from Google Sheets by caller ID (phone number)
3. **Agent greets** them with their real airline and confirmation number
4. **Caller verification** — agent asks for last name and validates it against the booking before proceeding
5. **Flight disruption briefing** — explains the delay/cancellation in plain language
6. **Connecting flight info** — if the caller has a connection, proactively provides that status
7. **Live weather** — answers destination weather questions via real-time API
8. **Travel policy Q&A** — answers policy questions (cancellation, baggage, loyalty points, etc.) via RAG pipeline

---

## 📁 Repository Structure

```
AI-Online-Travel-Agent/
├── n8n/
│   ├── inbound_call_handler.json        # Workflow 1: caller lookup via Google Sheets
│   ├── weather_tool.json                # Workflow 2: live weather custom function
│   └── rag_policy_lookup.json           # Workflow 3: RAG semantic search pipeline
├── supabase/
│   └── schema.sql                       # Vector table setup with pgvector
├── retell/
│   └── agent_prompt.md                  # Full Retell AI global prompt
├── policy_docs/                         # Source travel policy documents (PDF/text)
└── README.md
```

---

## 🚀 Setup Guide

### Prerequisites

- Retell AI account (with a purchased phone number)
- n8n instance (cloud or self-hosted)
- Supabase project (with pgvector extension enabled)
- OpenAI API key
- Google Sheets with traveler data

---

### Step 1: Supabase — Vector Store Setup

1. Create a new Supabase project
2. Enable the `pgvector` extension in the SQL editor:
   ```sql
   create extension if not exists vector;
   ```
3. Create the policies table:
   ```sql
   create table policies (
     id bigserial primary key,
     policy_type text,
     title text,
     content text,
     embedding vector(1536)
   );
   ```
4. Create an index for fast similarity search:
   ```sql
   create index on policies using ivfflat (embedding vector_cosine_ops);
   ```

> **Note:** The repo includes a `Policies Embedding Check` saved query in Supabase to verify embeddings were stored correctly.

---

### Step 2: n8n — Policy Ingestion (One-Time)

Run these workflows **once** to populate Supabase with embeddings:

**Workflow A — Main Policy Doc (airline, hotel, car):**
- Trigger: Manual / "Execute Workflow"
- Steps: Read policy rows → Loop over items → OpenAI Embeddings API → Supabase insert
- Chunking: Records are stored as single policy entries per type

**Workflow B — Supplemental Policy PDF (baggage, loyalty points, minors):**
- Trigger: Form submission (upload PDF)
- Steps: Default Data Loader → Embeddings OpenAI → Supabase Vector Store
- **Chunking strategy: Simple** (splits every 1000 characters with 200-character overlap)

> ⚠️ **Known Bug:** Semantic search across all policy types does not work as designed — when a caller asks for a specific airline cancellation policy, the vector search returned generic policy copy instead of the airline-specific entry. Workaround: a filtered copy of the Supabase table containing **only airline policy** rows is used for the RAG lookup. This is the table the n8n RAG workflow queries.

---

### Step 3: n8n — Inbound Call Handler

This workflow is triggered automatically by Retell AI when a call comes in.

**Flow:**
```
Webhook (POST) → Google Sheets lookup by phone → Return dynamic variables
```

**Dynamic variables returned to Retell AI:**
| Variable | Description |
|---|---|
| `airline` | Caller's main airline |
| `confirmation_number` | Booking confirmation code |
| `last_name_expected` | Expected last name for verification |
| `has_connection` | "yes" or "no" |
| `connecting_airline` | Connecting carrier (if applicable) |
| `connecting_confirmation` | Connecting booking code |
| `connection_city` | Layover city |
| `connection_update` | Status/update for the connection |

**Setup:**
1. Import `inbound_call_handler.json` into n8n
2. Set your Google Sheets credentials
3. Copy the webhook Production URL
4. In Retell AI → Phone Number → paste as the **Inbound Webhook URL**

---

### Step 4: n8n — Weather Tool

**Flow:**
```
Webhook (POST) → Nominatim geocoding → Open-Meteo API → Return weather summary
```

**Setup:**
1. Import `weather_tool.json` into n8n
2. Copy the webhook Production URL

---

### Step 5: n8n — RAG Policy Lookup

**Flow:**
```
Webhook (POST) → rag_query node → OpenAI Embeddings → Supabase Vector Store → Respond to Webhook
```

**Key configuration:**
- The `rag_query` node extracts the `question` field from the Retell AI function call payload
- OpenAI converts the question into a 1536-dimension embedding vector
- Supabase performs cosine similarity search against the filtered airline-only table
- The `Respond to Webhook` node returns:
  ```json
  { "response": "<matched policy text>" }
  ```

> **Critical:** In the `Respond to Webhook` node, make sure the response body field is set to **Expression mode** (toggle the `=` button). If left as static text, the n8n template expression `{{ $('Supabase Vector Store').item.json.document.pageContent }}` will be returned literally as a string instead of being evaluated.

**Setup:**
1. Import `rag_policy_lookup.json` into n8n
2. Add your OpenAI API credentials
3. Add your Supabase credentials (URL + service key)
4. Copy the webhook Production URL

---

### Step 6: Retell AI Agent Setup

#### 6a. Create the Agent
1. Go to Retell AI → Agents → Create Agent
2. Choose **Conversational Flow** type
3. Set execution mode to **Flex Mode**
4. Set LLM to **GPT-4.1**

#### 6b. Paste the Global Prompt
See `retell/agent_prompt.md` for the full prompt. Key behaviors:
- Greets with `{{airline}}` and `{{confirmation_number}}`
- Verifies caller using `{{last_name_expected}}`
- Proactively mentions connection status if `{{has_connection}}` is "yes"
- Always calls `get_weather` tool for weather questions — never answers from its own knowledge
- Always calls `get_travel_policy` tool for policy questions — never answers from its own knowledge

#### 6c. Add Custom Functions

**get_weather:**
| Field | Value |
|---|---|
| Name | `get_weather` |
| Description | Get current weather for a city when the caller asks about weather at their destination |
| API Endpoint | POST `https://<your-n8n>/webhook/weather-tool` |
| Timeout | 120000 ms |
| Parameter | `city` (string, required) — "The destination city to get weather for" |

**get_travel_policy:**
| Field | Value |
|---|---|
| Name | `get_travel_policy` |
| Description | Do semantic search on travel policy using RAG |
| API Endpoint | POST `https://<your-n8n>/webhook/policy-lookup` |
| Timeout | 120000 ms |
| Payload: args only | **ON** |
| Parameter | `question` (string, required) — "The user's exact policy question. Copy what they just asked." |
| Store Fields as Variables | Key: `response` / Value: `response` |

> **Important — Store Fields as Variables:** The key `response` tells Retell which JSON field to extract from the n8n response. The value `response` is the variable name available in your prompt as `{{response}}`. Both fields should say `response`.

#### 6d. Assign Phone Number
1. Retell AI → Phone Numbers → select your number
2. Assign the agent
3. Paste the **Inbound Webhook URL** from n8n Workflow 1

---

## 🧪 Testing

### Retell AI Simulation
Use the built-in **Simulation** tab to run test cases without making real calls:

| Test Case | User Prompt | Success Criteria |
|---|---|---|
| get travel policy airline delay | "Can I cancel my flight? What is the airline cancellation policy?" | Calls `get_travel_policy`, returns correct airline policy copy |
| get travel policy simulation test | "What if my luggage is delayed?" | Calls `get_travel_policy`, returns baggage policy |
| get weather simulation test | "What's the weather in Washington DC?" | Calls `get_weather`, returns current temperature |

### Call History Logs
Retell AI → Call History → select a call → view full transcript including tool invocations and their return values. This is your best debugging tool for tracing function call responses from n8n.

---

## ⚠️ Known Issues & Lessons Learned

### 1. RAG Semantic Search Not Working as Designed
**Problem:** When asking for airline-specific cancellation policy, the vector search returned the most similar chunk across all policy types — which was sometimes a generic or wrong policy.

**Root cause (suspected):** Chunking strategy and/or insufficient metadata filtering. The "simple" chunking (1000 chars / 200 overlap) on the supplemental PDF may produce chunks that score similarly across policy types.

**Workaround:** Created a filtered copy of the Supabase `policies` table containing only airline policy rows. The RAG workflow queries this table exclusively.

**Proper fix (TODO):** Add a `policy_type` metadata filter to the Supabase vector search query so the caller's question is matched only within the relevant policy category.

### 2. n8n Respond to Webhook — Unresolved Template Expression
**Problem:** Retell AI received `{"response":"{{ $('Supabase Vector Store').item.json.pageContent }}"}` literally.

**Fix:** In the Respond to Webhook node, toggle the response body field to **Expression mode** using the `=` button. Without this, n8n treats the expression as static text.

**Also:** The correct path to the content is `document.pageContent` (nested), not just `pageContent`.

### 3. Retell AI Store Fields as Variables — Value Field
**Problem:** If the value field in "Store Fields as Variables" contains descriptive text instead of the actual JSON key name, the variable will not be populated.

**Fix:** Both the key and value fields should contain `response` — the key is the JSON field name returned by n8n, and the value is the Retell variable name.

### 4. Other lessons learnt
- Retell KYC verification required before outbound calls work — contact support@retellai.com
- Phone numbers in Google Sheets must NOT have a + prefix (sheets treats + as a formula)
- n8n Cloud IP is blocked by Open-Meteo geocoding API — use Nominatim instead
- Always use Production webhook URLs in Retell — not the n8n test URLs
- After any Retell agent changes, click Publish for changes to take effect on live calls
---

## 📋 Retell AI Global Prompt

See last section of this ReadMe file for the complete prompt. Summary of key instructions:

- Use `{{airline}}`, `{{confirmation_number}}`, `{{last_name_expected}}` from dynamic variables
- Verify last name before sharing any trip details
- If `has_connection` is "yes", mention `{{connection_update}}` after the main flight situation
- **Weather:** always call `get_weather` — never use own knowledge
- **Policy:** always call `get_travel_policy` — relay `{{response}}` back to caller

---

## 🔮 Future Improvements

- [ ] Fix RAG chunking strategy to support proper multi-policy semantic search
- [ ] Add `policy_type` metadata filter to Supabase vector query
- [ ] Evaluate alternative chunking strategies (recursive, semantic) for the supplemental PDF

---

## 📖 Build Log

See `build_log.md` for a full step-by-step record of how this was built, including all bugs encountered and how they were fixed. Great resource for anyone learning n8n and Retell AI.
---

## 📖 Screenshots:

One time ingestion of travel policy docs into Supabase (RAG) and storing as vectors:
<img width="1820" height="905" alt="image" src="https://github.com/user-attachments/assets/30e15e21-0e0e-4088-92de-9dac74a5475d" /> 

Supabase : vector DB - travel policies stored in plain text
<img width="1862" height="615" alt="image" src="https://github.com/user-attachments/assets/c16ea8b3-a1af-4acc-a625-81b41bbe5159" />

Supabase - vector DB- travel policy copy converted into embeddings and inserted from the N8N flow
<img width="1863" height="633" alt="image" src="https://github.com/user-attachments/assets/287132c7-b9b1-45ed-afc9-d0e796369c68" />

Adding the upload of a supplemental travel policy pdf, with chunking strategy and vector conversions of pdf:
<img width="1814" height="888" alt="image" src="https://github.com/user-attachments/assets/f9d85a3d-dc17-4f9b-9706-449327d743b2" />

Chunking Strategy = Simple
<img width="1815" height="643" alt="image" src="https://github.com/user-attachments/assets/fe96965d-9d58-41c9-ad60-e37b8f020d0e" />

N8N workflow where question from retell AI is parsed and converted into embeddings and queried from vector store
bug: returns generic policy copy instead of specific- see output results at the bottom
<img width="1826" height="901" alt="image" src="https://github.com/user-attachments/assets/22618721-0776-48d0-9487-95ea102dbd44" />

Workaround fix: N8N workflow where airline copy was forced to be returned to retell AI (this needs to be fixed in next release)
<img width="1824" height="823" alt="image" src="https://github.com/user-attachments/assets/fbfc2080-28ce-4a83-99a2-e759676b2950" />


Retell AI: Conversational Voice Agent Flow
<img width="1850" height="906" alt="image" src="https://github.com/user-attachments/assets/4c071b09-361e-46b4-9ccd-8b8d7587bf71" />

Two function calls in Retell AI flow - to retrieve weather from N8N workflow and to retrieve travel policy from n8n rag flow
<img width="1795" height="790" alt="image" src="https://github.com/user-attachments/assets/bcb0a3a6-c4f9-4132-9ac4-d3cf8fbd4310" />

Retell AI weather function call set up
<img width="1003" height="857" alt="image" src="https://github.com/user-attachments/assets/b55697b1-071a-4155-a499-5da594b3a35d" />

Retell AI Travel Policy function call set up
<img width="734" height="839" alt="image" src="https://github.com/user-attachments/assets/42bf2783-02e0-4a7b-9b5e-957315a22afe" />

Retell AI :  Actual Call History log to help troubleshoot and trace function call return responses from N8N
<img width="1863" height="882" alt="image" src="https://github.com/user-attachments/assets/6031f842-019e-479f-8182-3e3efea50d3c" />

Retell AI: Run simulation test/test cases to validate intermediate results.
<img width="1842" height="895" alt="image" src="https://github.com/user-attachments/assets/e32b0d12-969e-4ad0-818e-4bb4cd9acf8e" />

## 📖 Full Retell AI Global Prompt Copy:
You are an AI phone agent for an online travel agency. You are calling the traveler proactively because their trip may be affected (for example: a flight delay or cancellation, or a hotel issue).
You first provide the user's airline and flight confirmation number. Then you ask for last name. If last name does not match the booking, apologize and end the call. if it matches, proceed with the call. If the caller has a connection, provide information about their specific connection.

When you speak, you must use the caller-specific values from Retell: {{airline}}, {{confirmation_number}}, and {{last_name_expected}} when they are present.
Always say the airline and confirmation number using those variables in your first substantive turn—do not invent different airlines or confirmation codes.
Last-name check: treat {{last_name_expected}} as the only last name that should unlock the “flight delay” path for this demo. If the user’s last name does not match, follow your existing rule to apologize and end.


Your goals:
- Clearly explain what changed in plain language, without causing panic.
- Say what it means for their trip in practical terms (connections, check-in timing, next steps).
- Ask one short question at a time if you need to clarify which booking or segment they mean.
- Offer sensible options when possible (for example: wait for updates, confirm alternate plans, contact the airline/hotel) without making up specific policies or guarantees.
- Keep replies brief: this is a phone call—usually 1–3 short sentences, unless they ask for details.
- Never invent confirmation codes, flight times, or refund amounts. If you don’t have the details, say what you can confirm and what you still need to verify.
- If they are upset, acknowledge it once, then move to helpful next steps.
- If the request is outside travel support or unsafe, politely decline and offer general assistance.

Style:
- Calm, confident, and human. Avoid jargon. Repeat important facts (times, cities) clearly.
You have access to the following trip details for this caller:
- Main airline: {{airline}}
- Confirmation number: {{confirmation_number}}
- Has connecting flight: {{has_connection}}
- Connecting airline: {{connecting_airline}}
- Connecting confirmation: {{connecting_confirmation}}
- Connection city: {{connection_city}}
- Connection update: {{connection_update}}

If has_connection is "yes", proactively mention the connecting flight status using connection_update after explaining the main flight situation.

When the caller asks about weather at their destination, use the get_weather tool with the destination city name. Then relay the weather_summary back to the caller in a natural, conversational way. You do NOT know current weather conditions. You must ALWAYS call the get_weather tool when the caller asks about weather. Never answer weather questions from your own knowledge. 

When the caller asks about any travel policy related questions, use the get_travel_policy tool. Then tell the user: {{response}}. Then relay the relevant policy summary back to the caller. You must always use the tool and never answer travel policy questions from your own knowledge. 


