# 🧪 Eval Document: Maren — AI Appointment Reminder Voice Agent

**Built by:** Sri Adidam  
**Platform:** Retell AI + N8N + OpenAI GPT-4.1  
**Version:** 1.0 · March 2026  
**Part of:** [Learner Apps Portfolio](https://github.com/semantic-sage/Learner-Apps)

---

## Table of Contents

1. [Use Case](#1-use-case)
2. [Problem Statement](#2-problem-statement)
3. [Evaluation Test Cases](#3-evaluation-test-cases)
4. [Evaluation Rubric](#4-evaluation-rubric)
5. [Metrics & Scoring](#5-metrics--scoring)
6. [Success Thresholds](#6-success-thresholds)
7. [How to Run Evals](#7-how-to-run-evals)
8. [Known Limitations](#8-known-limitations)
9. [Prompt Modifications](#9-prompt-modifications)

---

## 1. Use Case

An outbound AI voice agent named **Maren** calls clients on behalf of **Sri Adidam** to confirm upcoming AI consultation appointments. The agent identifies the recipient, introduces herself, states appointment details in human-friendly language, and confirms whether the time still works — without rescheduling or cancelling.

---

## 2. Problem Statement

Sri needs to confirm multiple client appointments automatically without manual outreach. The agent must sound professional and human, handle a variety of caller responses — including confirming, declining, expressing confusion, being unavailable, **interrupting the agent**, or **repeatedly asking the same question** — accurately convey appointment details, and gracefully exit calls without overstepping its authorization.

Failures in this agent directly impact client trust and no-show rates for Sri's AI consulting business.

---

## 3. Evaluation Test Cases

### Core Call Scenarios (1–10)

| # | Scenario | Caller Response | Expected Agent Behavior |
|---|---|---|---|
| 1 | Correct person answers | "Yes, this is John" | Proceeds to Step 2 introduction |
| 2 | Wrong person answers | "No, this isn't John" | Asks if John is available or offers to leave a message |
| 3 | Unavailable, message accepted | "Sure, leave a message" | Leaves appointment details, continues script |
| 4 | Unavailable, no message | "He's not here, can't take a message" | Says will try again later, uses `end_call` |
| 5 | Appointment confirmed | "Yes, that time works" | Responds warmly, asks if anything else needed, ends call |
| 6 | Wants to reschedule | "Can we move it to Thursday?" | Declines politely, states not authorized |
| 7 | Wants to cancel | "I need to cancel" | Declines politely, states not authorized |
| 8 | Caller is hesitant | "I'm not sure, let me check..." | Repeats appointment details clearly, re-confirms |
| 9 | Raw timestamp in variable | ISO format passed in | Converts to "2 PM on Friday, May 10th" |
| 10 | Caller asks who Sri is | "Who is Sri Adidam?" | Brief explanation, stays on task |

### ⚡ Interrupt & Repetition Scenarios (11–15) — New

| # | Scenario | Caller Response | Expected Agent Behavior |
|---|---|---|---|
| 11 | Caller interrupts mid-sentence | Talks over Maren while she is speaking | Stops immediately, listens, acknowledges, resumes naturally |
| 12 | Caller interrupts multiple times | Repeatedly cuts in throughout the call | Stays patient, never talks over caller, adapts pacing |
| 13 | Same question asked twice | "Wait, what time did you say?" asked twice | Answers calmly both times, rephrases second answer |
| 14 | Same question asked 3+ times | Keeps asking for appointment time | Answers twice, then gently redirects to write it down |
| 15 | Asks Maren to repeat herself | "Can you say that again?" | Repeats clearly, slightly rephrased — not word-for-word |

---

## 4. Evaluation Rubric

Each dimension is scored **1 = Bad · 2 = Average · 3 = Great**

---

### Dimension 1 — Identity Confirmation Accuracy

| Score | Description |
|---|---|
| 🔴 Bad | Skips identity check entirely, or proceeds without waiting for a yes/no. Reads the full script to the wrong person. |
| 🟡 Average | Asks the right question but does not branch correctly on a "no" answer — proceeds as if confirmed. |
| 🟢 Great | Correctly asks "Am I speaking with {{name}}?", waits, and branches accurately based on yes, no, or unclear response. |

---

### Dimension 2 — Appointment Detail Accuracy

| Score | Description |
|---|---|
| 🔴 Bad | Reads raw ISO timestamps (e.g. "2025-05-10T14:00:00Z"), omits the reason, or states wrong details entirely. |
| 🟡 Average | Converts time correctly most of the time but occasionally reads partial ISO strings or omits AM/PM. |
| 🟢 Great | Always converts timestamps to natural language ("2 PM on Friday, May 10th"), states reason and full time window accurately every time. |

---

### Dimension 3 — Authorization Boundary Compliance

| Score | Description |
|---|---|
| 🔴 Bad | Attempts to reschedule or cancel, offers to "check with Sri," or gives false promises about what will happen. |
| 🟡 Average | Declines the request but sounds robotic, abrupt, or unclear about why. Caller may be confused. |
| 🟢 Great | Politely declines, clearly states it is not authorized to make changes, keeps tone warm and professional without sounding defensive. |

---

### Dimension 4 — Conversational Naturalness

| Score | Description |
|---|---|
| 🔴 Bad | Sounds robotic or scripted. Uses the same phrase repeatedly (e.g., "Great!" three times). Long pauses or unnatural sentence structure. |
| 🟡 Average | Mostly natural, with occasional stiffness or repeated filler phrases. Does not vary language sufficiently across the call. |
| 🟢 Great | Sounds like a real human assistant. Varies phrasing, responds contextually to what the caller says, maintains a warm and professional tone throughout. |

---

### Dimension 5 — Interrupt Handling ⚡ New

| Score | Description |
|---|---|
| 🔴 Bad | Talks over the caller, ignores the interruption entirely, or restarts its full script from the beginning after being interrupted. |
| 🟡 Average | Stops speaking when interrupted but resumes awkwardly — repeats everything from the start or skips context the caller needed. |
| 🟢 Great | Stops immediately when the caller speaks, listens fully, acknowledges what was said, and resumes from the right point naturally — as a human would. |

---

### Dimension 6 — Repetition & Patience Handling ⚡ New

| Score | Description |
|---|---|
| 🔴 Bad | Gives an identical word-for-word robotic repeat every time the caller asks again. On the third ask, either loops endlessly or becomes incoherent. |
| 🟡 Average | Answers repeated questions correctly but uses the same phrasing each time. Does not cap the repetition loop. |
| 🟢 Great | Answers calmly with slightly varied phrasing each time. After two repeats, gently redirects — "I want to make sure you have this — you may want to jot it down." |

---

### Dimension 7 — Call Termination Handling

| Score | Description |
|---|---|
| 🔴 Bad | Does not end the call when appropriate. Continues talking after confirmation or keeps the line open after the person says goodbye. |
| 🟡 Average | Ends the call correctly in most cases but misses edge cases like an abrupt goodbye or the caller going silent. |
| 🟢 Great | Uses `end_call` correctly every time — on confirmation, on unavailability, and when the caller says goodbye. Never leaves the line awkwardly open. |

---

### Dimension 8 — Overall Call Completion Rate

| Score | Description |
|---|---|
| 🔴 Bad | Fails to reach the confirmation step in more than 40% of test calls. Gets stuck in loops or mishandles caller responses. |
| 🟡 Average | Completes the full script in 70–85% of calls. Stumbles on edge cases like hesitant callers or wrong-person scenarios. |
| 🟢 Great | Successfully completes the confirmation script in >95% of calls, including edge cases, interruptions, and repetitive callers. |

---

## 5. Metrics & Scoring

### Scoring Scale

Each rubric dimension is scored **1–3** and then weighted by business impact to produce an **Overall Score out of 100**.

| # | Dimension | Weight | Max Points | Rationale |
|---|---|---|---|---|
| 1 | Identity Confirmation | 15% | 15 | Calling the wrong person = complete call failure |
| 2 | Appointment Detail Accuracy | 20% | 20 | Wrong details destroy client trust immediately |
| 3 | Authorization Boundary | 15% | 15 | Overstepping damages Sri's professional brand |
| 4 | Conversational Naturalness | 10% | 10 | Affects caller experience and completion rate |
| 5 | Interrupt Handling | 15% | 15 | Critical for realistic real-world call scenarios |
| 6 | Repetition & Patience | 10% | 10 | Directly impacts confused or elderly callers |
| 7 | Call Termination | 5% | 5 | Binary pass/fail in practice — lower weight |
| 8 | Overall Completion Rate | 10% | 10 | Captures end-to-end success holistically |
| | **TOTAL** | **100%** | **100** | |

### Scoring Formula

```
For each dimension:
  Dimension Score = (Raw Score 1–3 ÷ 3) × Weight × 100

Example: Identity Confirmation scored 2 (Average)
  = (2 ÷ 3) × 15 = 10.0 points

Sum all dimension scores → Overall Score out of 100
```

> **⚠️ Hard Fail Rule:** If any single dimension scores **1 (Bad)** on Dimensions 1, 2, or 3 (Identity, Accuracy, or Authorization), the agent is **automatically flagged for rework** regardless of the overall score. These are non-negotiable for client trust.

---

## 6. Success Thresholds

| Overall Score | Status | Action Required |
|---|---|---|
| 90 – 100 | ✅ **Ready to Publish** | Agent is live-ready. Monitor in production. |
| 75 – 89 | 🟡 **Needs Minor Fixes** | Fix lowest-scoring dimensions. Re-eval before publishing. |
| 60 – 74 | 🟠 **Needs Major Rework** | Significant prompt modifications required. Full re-eval after changes. |
| Below 60 | 🔴 **Do Not Publish** | Agent is not fit for production. Redesign core prompt logic. |

---

## 7. How to Run Evals

### Who Runs Them

**Primary Evaluator:** Sri Adidam or a designated team member who knows the intended agent behavior and call flow. A second reviewer is recommended for borderline scores (dimensions scoring 2) to reduce subjectivity.

### When to Run Evals

| Trigger | Eval Type | Scope |
|---|---|---|
| Before initial publish | Full eval | All 15 test cases, all 8 dimensions |
| After any prompt modification | Full eval | All 15 test cases, all 8 dimensions |
| After dynamic variable changes | Targeted eval | Test cases 9, 13, 14 (time/repetition) |
| Weekly in production (first month) | Spot check | 5 randomly selected test cases |
| Monthly in production (ongoing) | Spot check | 3 randomly selected test cases |

### Step-by-Step Process

1. **Open** the Retell AI dashboard. Navigate to your agent and open the Test Call panel.
2. **Set up** a test call using real dynamic variable values (`name`, `phone`, `reason`, `start_time`, `end_time`).
3. **Run** each test case scenario by simulating the caller response listed in the Test Cases table.
4. **Score** each dimension 1–3 using the Rubric. Record in a scoring sheet.
5. **Calculate** the Overall Score using the weighted formula in the Metrics section.
6. **Compare** against Success Thresholds. Apply the Hard Fail Rule for Dimensions 1, 2, and 3.
7. **If score is below 90**, apply the relevant Prompt Modifications and re-run affected test cases.
8. **Log** the eval date, overall score, dimension scores, and any changes made for audit trail.

---

## 8. Known Limitations

> These limitations are acknowledged and accepted for the current version of Maren. Documented here so stakeholders are not surprised in production.

**1. No Voicemail Detection**  
Maren cannot reliably distinguish a live person from a voicemail greeting. She may begin her identity confirmation script before recognizing she has reached voicemail. The voicemail handling prompt fix (Fix 7) mitigates this but does not fully solve it.

**2. No Calendar Integration at Call Time**  
Appointment details are injected as static variables at the time the call is triggered by N8N. If the appointment changes after the call is placed, Maren will still quote the original details. There is no live lookup during the call.

**3. Cannot Handle Multi-Party Calls**  
If a third party joins the call or the caller puts Maren on hold and another person picks up, Maren has no instruction for this scenario. She may continue as if speaking to the original contact.

**4. No Rescheduling or Cancellation Capability**  
By design, Maren cannot reschedule or cancel. This is a business decision, not a technical one. Callers who need changes must contact Sri's team separately — this creates a dependency on a human follow-up channel being available and responsive.

**5. Language Limitation**  
Maren is configured for English (en-US) only. Callers who respond in another language will receive English responses, which may cause confusion or call abandonment.

**6. Accent & Speech Recognition Variability**  
Performance may vary with heavy accents, fast speech, or poor audio quality. Interrupt handling and repetition handling are particularly sensitive to speech recognition accuracy.

**7. Eval Coverage is Manual**  
All 15 test cases require manual simulation. There is no automated test harness. This limits how frequently full evals can be run and introduces evaluator subjectivity.

---

## 9. Prompt Modifications

### Fix 1 — Prevent Raw Timestamp Leakage

**Problem:** If `{{start_time}}` or `{{end_time}}` variables arrive in ISO format, the agent may read them verbatim.

**Add to the Notes section of the prompt:**

```
CRITICAL: You must NEVER speak a raw timestamp. If the time looks like
"2025-05-10T14:00:00Z" or any ISO format, always convert it before speaking.

Examples:
  "2025-05-10T14:00:00Z"  →  "2 PM on Saturday, May 10th"
  "2025-05-10T15:30:00Z"  →  "3:30 PM on Saturday, May 10th"

If you are uncertain of the converted time, say "your scheduled appointment time"
rather than reading the raw value.
```

---

### Fix 2 — Tighten the Authorization Boundary Response

**Problem:** "Sorry, I am not authorized" sounds abrupt and robotic.

**Replace the current decline line with:**

```
"I completely understand — unfortunately I'm only set up to confirm
appointments, not make changes. I'd recommend reaching out to Sri's
team directly for that. Is there anything else I can help with today?"
→ Then use end_call
```

---

### Fix 3 — Add a Repetition Guard

**Problem:** The agent may repeat "Awesome!" or "Great!" too many times across a single call.

**Add to the Style Guardrails section:**

```
Vary your affirmative responses. Do not use the same word twice in one call.
Rotate through: "Wonderful", "Perfect", "Sounds good", "Fantastic",
"That's great to hear" — use each at most once per call.
```

---

### Fix 4 — Handle Hesitant Callers Explicitly

**Problem:** The fallback after a second failed confirmation attempt is not defined, leaving the agent in an unresolved loop.

**Add after the "If they're unsure" block:**

```
If the caller is still unsure after one re-confirmation, say:
"No worries at all — I'll make a note that we spoke.
You can always reach Sri's team if you have any questions.
Have a wonderful day!"
→ Use end_call

Do not attempt a third confirmation. Two attempts maximum per call.
```

---

### Fix 5 — Add Interrupt Handling ⚡ New

**Problem:** The prompt has no explicit instruction for what to do when a caller talks over Maren mid-sentence.

**Add a new section called INTERRUPT HANDLING:**

```
INTERRUPT HANDLING:
If the caller speaks while you are talking, stop immediately — do not
finish your sentence. Listen to what they say fully before responding.

After they finish, acknowledge naturally before continuing:
  "Of course, go ahead."
  "Sure, what's on your mind?"
  "Absolutely — what would you like to know?"

Resume from the most relevant point in the conversation, not from
the beginning of your script. Never restart the full introduction
after an interruption unless you are speaking to a new person.

If the caller interrupts more than twice in quick succession,
slow your speaking pace and use shorter sentences to give them
more natural room to speak.
```

---

### Fix 6 — Add Repeated Question Handling ⚡ New

**Problem:** The prompt does not define how to handle a caller who asks for the same information multiple times, which can cause robotic looping.

**Add a new section called REPETITION HANDLING:**

```
REPETITION HANDLING:
If the caller asks for the same information a second time, answer
calmly with slightly different phrasing:

First answer:
  "Your appointment is for {{reason}} between {{start_time}} and {{end_time}}."

Second answer (rephrased):
  "Just to confirm again — Sri has you scheduled for {{reason}},
  starting at {{start_time}} and wrapping up by {{end_time}}."

If the caller asks a third time, gently redirect:
  "I want to make sure you have this — you may want to jot it down:
  {{reason}}, between {{start_time}} and {{end_time}}. Sri's team can
  also send you a confirmation if that would help."

→ After this, do not repeat again. Offer end_call if no further questions.

Never sound impatient or mechanical when repeating.
Treat every repetition as if it is the first time the caller is hearing it.
```

---

### Fix 7 — Add Voicemail Handling

**Problem:** The prompt has no instruction for calls that go to voicemail — a very common real-world scenario.

**Add a new section called VOICEMAIL HANDLING:**

```
VOICEMAIL HANDLING:
If you reach voicemail, leave this message:
  "Hi, this is Maren calling on behalf of Sri Adidam.
  I'm reaching out to confirm your upcoming appointment.
  Please feel free to reach out to Sri's team if you have any questions.
  Have a great day!"

→ Use end_call immediately after the message.
Do not wait for a response on voicemail.
```

---

*Maren Voice Agent Eval Framework · Sri Adidam · Retell AI + N8N + OpenAI GPT-4.1 · March 2026*
