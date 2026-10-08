# 🎯 Virtual Interview Platform — Complete Operational Blueprint
## Every Service · Every Flow · Every Step · How Everything Works

---

## Part 1 — THE BIG PICTURE (How the Whole Thing Works)

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│                        WHO DOES WHAT AND WHEN                                    │
├───────────────┬──────────────────────────────────────────────────────────────────┤
│  RECRUITER    │ 1. Creates a Job Role                                             │
│               │ 2. Adds or generates questions for that role                      │
│               │ 3. Configures how the AI scores candidates                       │
│               │ 4. Sends invite link to candidate                                 │
│               │ 5. Waits for interview to complete                                │
│               │ 6. Reviews behavioral report + video replay                       │
│               │ 7. Makes hire/no-hire decision                                    │
├───────────────┼──────────────────────────────────────────────────────────────────┤
│  CANDIDATE    │ 1. Receives email invite (link valid 72 hrs)                      │
│               │ 2. Opens link → sees consent + camera check                      │
│               │ 3. Starts interview → speaks to AI avatar                        │
│               │ 4. Answers questions → AI probes / follows up                    │
│               │ 5. Interview ends → sees "Thank you" screen                      │
├───────────────┼──────────────────────────────────────────────────────────────────┤
│  AI SYSTEM    │ 1. Conducts interview (avatar speaks + listens)                  │
│               │ 2. Captures all audio/video in background                         │
│               │ 3. Analyses speech, behaviour, facial signals                    │
│               │ 4. Scores all 14 behavioural dimensions                          │
│               │ 5. Generates report + sends to recruiter                         │
└───────────────┴──────────────────────────────────────────────────────────────────┘
```

---

## Part 2 — ALL AZURE SERVICES: EXACTLY WHAT EACH ONE DOES

```
SERVICE                          │ EXACT ROLE IN THIS PLATFORM
─────────────────────────────────┼────────────────────────────────────────────────
Azure AI Foundry (Agent Service) │ The "brain" of the AI interviewer.
                                 │ Manages the conversation thread, decides what
                                 │ question to ask next, evaluates answers,
                                 │ generates follow-up probes, scores responses.
                                 │ Uses GPT-4o model underneath.
─────────────────────────────────┼────────────────────────────────────────────────
GPT-4o Realtime API              │ Powers the live two-way voice conversation.
(inside Azure AI Foundry)        │ The candidate speaks → audio goes here → AI
                                 │ responds in speech in real-time (< 1 second).
                                 │ Handles voice activity detection (knows when
                                 │ candidate starts/stops talking).
─────────────────────────────────┼────────────────────────────────────────────────
Azure AI Speech Service (STT)    │ Converts candidate's spoken audio into text.
                                 │ Produces word-by-word transcript with
                                 │ timestamps. Also measures pronunciation
                                 │ accuracy, speaking rate (WPM), and pauses.
─────────────────────────────────┼────────────────────────────────────────────────
Azure AI Speech Service (TTS)    │ Converts the AI agent's text responses into
                                 │ natural-sounding speech audio. Uses the
                                 │ interviewer's configured voice.
─────────────────────────────────┼────────────────────────────────────────────────
Azure Neural TTS Avatar          │ Takes the TTS audio and generates a
                                 │ photorealistic video of the AI interviewer
                                 │ speaking (lip-synced). This is the video the
                                 │ candidate sees on screen.
─────────────────────────────────┼────────────────────────────────────────────────
Azure AI Vision                  │ Analyses snapshot images (frame captures)
                                 │ from the candidate's camera every 5 seconds.
                                 │ Detects: number of faces in frame, head
                                 │ position, eye gaze direction, whether another
                                 │ person is visible.
─────────────────────────────────┼────────────────────────────────────────────────
GPT-4o Vision                    │ Same snapshot images sent to GPT-4o with a
(Azure OpenAI)                   │ vision prompt to judge: posture quality,
                                 │ engagement level, signs of reading from notes.
─────────────────────────────────┼────────────────────────────────────────────────
Azure Communication Services     │ Handles the WebRTC video call infrastructure.
(ACS)                            │ Manages STUN/TURN servers so the browser can
                                 │ make a call. Also automatically records the
                                 │ entire interview (audio + video) to Blob.
─────────────────────────────────┼────────────────────────────────────────────────
Azure Blob Storage               │ Stores everything large:
                                 │ - Interview recordings (MP4)
                                 │ - Camera frame snapshots (JPG)
                                 │ - Generated PDF reports
                                 │ - Cached avatar video clips (common phrases)
─────────────────────────────────┼────────────────────────────────────────────────
Azure Cosmos DB                  │ The main database. Stores:
                                 │ - Interview sessions (state, questions asked)
                                 │ - Question banks (per job role)
                                 │ - Candidate records
                                 │ - Behavioural scores + reports
                                 │ - Rubric configurations
                                 │ - Raw telemetry events
─────────────────────────────────┼────────────────────────────────────────────────
Azure Service Bus                │ Message queue between services.
                                 │ When interview ends → puts message on queue →
                                 │ triggers scoring worker, report worker, etc.
                                 │ Ensures nothing is lost even if a service
                                 │ crashes during processing.
─────────────────────────────────┼────────────────────────────────────────────────
Azure Cache for Redis            │ Fast in-memory cache. Stores:
                                 │ - Current session state (which Q is active)
                                 │ - Rubric configs (avoids DB hit every request)
                                 │ - Rate limit counters
                                 │ - Session heartbeat status
─────────────────────────────────┼────────────────────────────────────────────────
Azure SignalR Service            │ Pushes real-time notifications to the browser.
                                 │ E.g. "Interview complete" → recruiter dashboard
                                 │ instantly updates without page refresh.
─────────────────────────────────┼────────────────────────────────────────────────
Azure Container Apps             │ Hosts all the backend .NET services and
                                 │ background workers. Auto-scales based on load.
─────────────────────────────────┼────────────────────────────────────────────────
Azure Static Web Apps            │ Hosts the Next.js frontend apps (candidate
                                 │ interview room + recruiter dashboard).
─────────────────────────────────┼────────────────────────────────────────────────
Azure API Management (APIM)      │ Single entry point for all API calls.
                                 │ Handles: rate limiting, API key validation,
                                 │ CORS, JWT token validation, request routing.
─────────────────────────────────┼────────────────────────────────────────────────
Azure Front Door + WAF           │ Global CDN + security layer. Blocks bad
                                 │ traffic (DDoS, bots), routes to nearest
                                 │ region, serves static assets from cache.
─────────────────────────────────┼────────────────────────────────────────────────
Microsoft Entra ID               │ Authentication for recruiters and admins.
(formerly Azure AD)              │ Recruiters log in with their company account.
                                 │ Candidates do NOT need an account.
─────────────────────────────────┼────────────────────────────────────────────────
Azure Key Vault                  │ Secure storage for all secrets (API keys,
                                 │ connection strings). Services never have
                                 │ secrets in code or config files.
─────────────────────────────────┼────────────────────────────────────────────────
Azure Monitor + App Insights     │ Logs, metrics, and alerts. Tells you when
                                 │ something breaks, how fast the system is,
                                 │ how many interviews are completing, etc.
─────────────────────────────────┴────────────────────────────────────────────────
```

---

## Part 3 — QUESTION BANK: HOW QUESTIONS WORK

### 3.1 Question Structure (Stored in Cosmos DB)

Every question in the system has these fields:

```
Question {
    id:               "q-001"
    tenantId:         "company-abc"
    jobRoleId:        "senior-engineer"
    
    text:             "Tell me about a time you had to debug a production issue 
                       under pressure. What was your approach?"
    
    type:             "Behavioral"        ← Behavioral | Technical | Situational | Warmup
    topic:            "Problem Solving"   ← What competency this tests
    difficulty:       "Standard"          ← Warmup | Standard | Advanced | Expert
    
    expectedDuration: 90                  ← Expected answer length in seconds
    
    followUpBank: [                       ← Pre-written follow-ups if answer is weak
        "Can you walk me through the specific steps you took?",
        "What would you do differently now?",
        "How did the rest of the team respond?"
    ]
    
    scoringHints:     "Look for: specific technical tool used, 
                       timeline pressure acknowledged, outcome measured"
    
    isActive:         true
    createdBy:        "recruiter-user-id"
    createdAt:        "2025-01-01T00:00:00Z"
}
```

### 3.2 How a Recruiter Adds Questions (3 Ways)

#### Way 1: Manual Entry (Recruiter types them)
```
Recruiter → Dashboard → Job Roles → "Senior Engineer" → Question Bank
         → Click "Add Question"
         → Fills in: question text, type, topic, difficulty
         → Optionally adds follow-up prompts
         → Saves → stored in Cosmos DB under jobRoleId
```

#### Way 2: AI-Assisted Generation
```
Recruiter → Clicks "Generate Questions with AI"
         → Provides: Job Description (paste text)
         → Selects: Number of questions (e.g. 3 Technical, 3 Behavioral, 2 Situational)
         → Selects: Difficulty mix

System calls GPT-4o with this prompt:
┌─────────────────────────────────────────────────────────────────────────┐
│ You are an expert technical recruiter. Based on this job description:   │
│ {jobDescription}                                                        │
│                                                                         │
│ Generate exactly:                                                       │
│ - 3 Technical questions (Advanced difficulty)                           │
│ - 3 Behavioral questions (Standard difficulty)                          │
│ - 2 Situational questions (Standard difficulty)                         │
│                                                                         │
│ For each question return JSON:                                          │
│ { text, type, topic, difficulty, followUpBank[3], scoringHints }       │
│                                                                         │
│ Questions must be specific to this role. Avoid generic questions.      │
└─────────────────────────────────────────────────────────────────────────┘

GPT-4o returns structured JSON → system saves each as a Question record
Recruiter reviews, edits, deletes any → approves the set
```

#### Way 3: Import from Template Library
```
System has a built-in template library:
  - "Software Engineer (Generic)" → 20 pre-built questions
  - "Product Manager" → 18 pre-built questions
  - "Sales Executive" → 15 pre-built questions
  - etc.

Recruiter selects a template → questions copied into their job role
They can then edit/delete/add to customise
```

### 3.3 Question Bank Structure Per Interview

A complete interview is configured like this:

```
Interview Config for "Senior Engineer" Role:
┌─────────────────────────────────────────────────────────────────────────┐
│  PHASE 1 — INTRO (0 questions, AI talks only)                          │
│  Duration: ~45 seconds                                                  │
│  Script: Fixed introduction text (customisable per tenant)              │
│                                                                         │
│  PHASE 2 — WARM-UP (1–2 questions)                                     │
│  Always asked: "Tell me a little about yourself and your background"   │
│  Type: Warmup, no follow-ups triggered                                  │
│                                                                         │
│  PHASE 3 — TECHNICAL MODULE (3–5 questions, recruiter configured)      │
│  Question pool: 10 technical questions in bank                          │
│  AI selects: 3 questions based on candidate's resume keywords           │
│  Difficulty: Starts Standard, adapts up/down based on answer quality   │
│                                                                         │
│  PHASE 4 — BEHAVIORAL MODULE (2–4 questions)                           │
│  Question pool: 8 behavioral questions in bank                          │
│  AI selects: 3 questions covering different competency topics           │
│  Follow-ups: Triggered if STAR score < 5                                │
│                                                                         │
│  PHASE 5 — CLOSING (0 questions, candidate can ask questions)          │
│  AI asks: "Do you have any questions for us?"                           │
│  Duration: ~2 minutes max                                               │
└─────────────────────────────────────────────────────────────────────────┘
```

### 3.4 How the AI Selects Which Questions to Ask

```
Step 1: At session start, AI agent receives the full question bank for the role
        (all 10+ questions in Cosmos DB for that jobRoleId)

Step 2: Agent applies selection logic:
        - If candidate resume was uploaded → extract key skills
          → prioritise questions matching those skill keywords
        - Remove any questions already asked (deduplication list in session)
        - Select questions across different topics (avoid asking 2 Q about same topic)

Step 3: During interview, difficulty is adaptive:
        - Answer score ≥ 8/10  → next question from "Advanced" pool
        - Answer score 5–7/10  → stay at same difficulty
        - Answer score < 5/10  → drop to easier question or probe current one

Step 4: Follow-up decision (per answer):
        STAR score < 5 AND no follow-up yet asked for this Q:
            → Agent generates a contextual follow-up:
               e.g. Candidate said: "I optimised the database"
               Follow-up: "Can you tell me specifically what optimisations you made
                            and what impact they had on performance?"
            → Follow-up is generated dynamically (not from pre-written bank)
              using the candidate's actual words from their answer
        
        STAR score ≥ 5 OR follow-up already asked:
            → Move to next question
```

---

## Part 4 — RECRUITER WORKFLOW: STEP BY STEP

### Step 1: Recruiter Login
```
Recruiter visits → app.yourdomain.com
Browser → MSAL.js → Microsoft Entra ID login page
Recruiter logs in with their company Microsoft account (SSO)
Entra ID returns JWT token → stored in browser memory (not localStorage)
All API calls include: Authorization: Bearer {token}
API validates token signature against Entra ID's public keys
```

### Step 2: Create a Job Role
```
Recruiter → "New Job Role" form
Fills in:
  - Role Title: "Senior Software Engineer"
  - Department: "Engineering"  
  - Job Description: (paste full JD)
  - Interview Duration: 30 minutes
  - Modules to include: [Warm-up, Technical, Behavioral, Closing]
  - Technical questions count: 4
  - Behavioral questions count: 3
  - Language: English (US)
  - Avatar style: "business" (professional)

System creates:
  - JobRole record in Cosmos DB
  - Empty QuestionBank linked to this role
```

### Step 3: Build the Question Bank
```
Recruiter → Question Bank tab for this role
Options:
  A. Auto-generate (AI generates questions from job description)
  B. Manual entry
  C. Import from template

After questions are added, recruiter can:
  - Preview each question (how the AI will ask it)
  - Reorder questions (drag and drop)
  - Mark questions as "Required" (always asked) or "Optional" (AI picks from pool)
  - Set per-question time limit (max 3 minutes to answer)
  - Add follow-up prompts per question (or let AI generate dynamically)
```

### Step 4: Configure the Scoring Rubric
```
Recruiter → Scoring tab for this role

Sees a list of competencies with sliders (weights must total 100%):
┌─────────────────────────────────────────────────────────────┐
│ COMPETENCY WEIGHTS                                          │
│                                                             │
│ Technical Depth          [══════════════════░░░░] 35%      │
│ Problem Solving          [═════════════░░░░░░░░░] 25%      │
│ Communication            [══════════░░░░░░░░░░░░] 20%      │
│ Behavioral (STAR)        [═══════░░░░░░░░░░░░░░░] 15%      │
│ Engagement & Presence    [══░░░░░░░░░░░░░░░░░░░░]  5%      │
│                                              TOTAL: 100%   │
└─────────────────────────────────────────────────────────────┘

Optional toggles:
  ☐ Enable Blind Mode (exclude visual signals from scoring)
  ☐ Include eye contact in scoring
  ☐ Include vocal confidence in scoring
  ☐ Flag scripted/AI-generated responses

Minimum score to pass: [  70  ] %
Decision labels:
  ≥ 85: "Strong Hire"   ≥ 70: "Hire"   55–69: "Borderline"   < 55: "No Hire"
```

### Step 5: Send Invite to Candidate
```
Recruiter → Candidates tab → "Invite Candidate"
Fills in: Candidate name + email address
System:
  1. Creates Candidate record in Cosmos DB
  2. Creates InterviewSession record (Status: WaitingCandidate)
  3. Generates signed invite URL:
     https://interview.yourdomain.com/session/abc123?token=HMAC_SIGNED_TOKEN
     Token encodes: sessionId + candidateId + expiryTimestamp (72 hrs)
     Token is HMAC-SHA256 signed (secret in Key Vault)
  4. Sends email via SendGrid:
     - Recruiter-configured email template
     - Contains invite link + instructions
     - Contains: what to prepare, technical requirements (camera, mic)
     - Contains: interview duration estimate

Recruiter dashboard shows: "Invited - Waiting for candidate"
```

### Step 6: Candidate Completes Interview
```
(See Part 5 for full candidate flow)
```

### Step 7: Recruiter Reviews Results
```
After interview completes, recruiter receives:
  - Browser notification (via SignalR push)
  - Email: "Interview complete for Jane Smith — Score: 82/100"

Recruiter opens dashboard → clicks candidate name → sees:

┌─────────────────────────────────────────────────────────────────────┐
│  JANE SMITH — Senior Software Engineer Interview                    │
│  Duration: 28 mins  │  Completed: Today 3:45 PM  │  Score: 82/100 │
├──────────────────────────────┬──────────────────────────────────────┤
│  VIDEO REPLAY                │  SCORECARD                          │
│                              │                                      │
│  [════▶ 00:00 / 28:00 ]      │  Technical Depth:    88/100         │
│                              │  Problem Solving:    85/100         │
│  Timeline:                   │  Communication:      79/100         │
│  [🟢🟢🟡🟢🟢🟡🟢🟢🟢🟢🟡]  │  Behavioral STAR:    76/100         │
│   Q1  Q2  Q3  Q4  Q5  Q6    │  Engagement:         72/100         │
│                              │                                      │
│  🟢 Confident answer         │  BEHAVIOURAL SIGNALS                │
│  🟡 Hesitation detected      │  WPM: 142  (Optimal: 110-180)       │
│  🔴 Long pause               │  Filler words: 3.2/min  ⚠️          │
│                              │  Eye contact: 68%                   │
│                              │  Pauses: avg 1.9s                   │
│  TRANSCRIPT                  │  Focus events lost: 1               │
│  [Q1] Tell me about...       │                                      │
│  [A]  "I worked at..."       │  DECISION: 🟢 HIRE                  │
│  [Follow-up] Can you...      │                                      │
│  [A]  "Specifically I..."    │  [Download PDF]  [Share Report]     │
└──────────────────────────────┴──────────────────────────────────────┘

Clicking any Q on the timeline jumps the video to that moment.
Clicking any flagged event (🟡) jumps to that timestamp.
```

---

## Part 5 — CANDIDATE JOURNEY: STEP BY STEP

### Step 1: Candidate Opens Invite Link
```
Candidate clicks link in email
Browser loads: https://interview.yourdomain.com/session/abc123?token=...

System validates:
  - Token signature valid (HMAC check)
  - Token not expired (< 72 hrs old)
  - Session status is "WaitingCandidate" (not already completed/cancelled)
  
If valid → loads interview preparation page
If invalid → "This link has expired. Please contact the recruiter."
```

### Step 2: Consent & Equipment Check
```
Screen 1: CONSENT
┌─────────────────────────────────────────────────────────────────────┐
│  Before we start, please review the following:                      │
│                                                                     │
│  ☐ This interview will be recorded (audio and video)               │
│  ☐ AI analysis will be performed on your responses                 │
│  ☐ Results will be shared with [Company Name]                      │
│  ☐ Your data will be retained for [X] years (deletable on request) │
│                                                                     │
│  By clicking Continue, you agree to these terms.                   │
│                                          [Decline] [Continue →]    │
└─────────────────────────────────────────────────────────────────────┘
Consent record saved to Cosmos DB immediately.

Screen 2: EQUIPMENT CHECK
  - Camera test: shows live preview
  - Microphone test: waveform animation (speak to test)
  - Speed check: tests upload speed (minimum 2 Mbps)
  - Browser check: confirms Chrome/Edge/Firefox
  - Lighting check: GPT-4o Vision rates frame brightness
  
All checks pass → "You're ready!" → [Start Interview]
Any fail → shows specific fix instruction
```

### Step 3: Interview Begins
```
Candidate clicks "Start Interview"

System does (all at once, ~3 seconds):
  1. Calls POST /sessions/abc123/start
  2. Backend: creates Azure Communication Services (ACS) call
             → gets ACS token for this candidate (30 min TTL)
             → starts call recording (automatic, to Blob Storage)
  3. Backend: calls Azure AI Foundry → creates new Agent Thread
             → sends initial system prompt with:
               - Interviewer persona name and style
               - Candidate's name
               - Job role context
               - Question bank for this role
               - Phase: starting at "Intro"
  4. Browser: opens WebSocket to GPT-4o Realtime API endpoint
             → sends session configuration (voice, VAD settings)
             → starts streaming microphone audio
  5. Browser: starts capturing camera frames every 5s
             → sends to Telemetry API → stored in Blob → queued for Vision analysis

Candidate screen shows:
  - AI avatar video (playing greeting)
  - Candidate's own camera (small PiP in corner)
  - Progress bar (Phase: Introduction)
  - Timer (28:00 remaining)
  - Mute button (can mute but recorded regardless via ACS)
```

### Step 4: The Interview Conversation Loop

```
Each question cycle works like this:

┌─────────────────────────────────────────────────────────────────────────────┐
│ 1. AGENT ASKS QUESTION                                                      │
│                                                                             │
│    Agent thread generates question text                                     │
│    → Sent to GPT-4o Realtime → converted to speech audio                  │
│    → Speech audio also sent to Azure Neural Avatar API                     │
│    → Avatar generates lip-synced video of interviewer speaking             │
│    → Video streamed to candidate browser <video> element                   │
│    → Candidate hears interviewer and sees them speak                       │
│                                                                             │
│    QuestionRecord saved to session: { text, askedAt, orderIndex }          │
│    Redis updated: session:{id}:currentQuestion = "q-003"                   │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ 2. CANDIDATE RESPONDS                                                       │
│                                                                             │
│    Microphone audio streamed via WebSocket to GPT-4o Realtime              │
│    GPT-4o Realtime VAD detects when candidate stops speaking                │
│    (1.5s of silence = end of response)                                     │
│                                                                             │
│    Simultaneously:                                                          │
│    → Azure Speech SDK transcribes audio → adds to running transcript       │
│    → Speech SDK measures: WPM, pause durations, filler words               │
│    → Browser Vision API captures frames → sends to Telemetry API           │
│    → Browser logs: response start time, response end time                  │
│                                                                             │
│    Events sent to Telemetry API as batches every 3 seconds:               │
│    { type: "WpmSample", value: 148, timestamp: ... }                       │
│    { type: "FillerWordDetected", word: "um", timestamp: ... }              │
│    { type: "PauseDetected", durationMs: 2300, timestamp: ... }             │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ 3. AGENT EVALUATES RESPONSE                                                 │
│                                                                             │
│    GPT-4o Realtime transcribes full response                               │
│    → Sends to Agent thread with evaluation prompt:                         │
│                                                                             │
│    "Candidate answered: '{response_text}'                                  │
│     Question was: '{question_text}'                                        │
│     Score this on STAR scale 0-10, identify missing elements,              │
│     decide: follow_up_needed (bool), next_action (probe|advance|close)"   │
│                                                                             │
│    Agent returns JSON:                                                      │
│    {                                                                        │
│      "star_score": 4,                                                      │
│      "missing": ["Result not quantified", "No timeline mentioned"],        │
│      "follow_up_needed": true,                                             │
│      "follow_up_text": "That's interesting. Could you tell me the         │
│                          specific impact this had on your team's output?"  │
│    }                                                                        │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ 4a. IF FOLLOW-UP NEEDED                                                     │
│                                                                             │
│    Agent speaks the follow-up question (same avatar loop as Step 1)        │
│    Candidate answers → evaluation runs again                               │
│    If 2nd attempt still weak → agent acknowledges + moves on               │
│    (never asks more than 1 follow-up per question)                         │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ 4b. IF ANSWER IS GOOD (score ≥ 5)                                          │
│                                                                             │
│    Agent gives brief acknowledgement:                                      │
│    "Great, thank you for sharing that."                                    │
│    "That's a really thorough answer."                                      │
│    (Varies to avoid repetition — agent chooses from transition bank)       │
│                                                                             │
│    Agent thread updates phase if all Qs in module complete                 │
│    → Selects next question from appropriate pool                           │
│    → Cycle repeats                                                          │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Step 5: Interview Ends
```
All phases complete OR time limit reached:

Agent delivers closing:
  "We've come to the end of our interview, {CandidateName}.
   Do you have any questions for me about the role or the team?"

Candidate asks questions (optional, max 2 min)

Agent delivers goodbye:
  "Thank you so much for your time today. We'll be in touch
   with next steps. Take care!"

System actions:
  1. POST /sessions/abc123/complete called (triggered by agent)
  2. Session status → Completed, completedAt recorded
  3. ACS call ends → recording finalised in Blob Storage
  4. WebSocket closed
  5. Message published to Service Bus: post-interview topic
  6. Candidate browser shows: "Thank you" screen
```

---

## Part 6 — POST-INTERVIEW PROCESSING: EVERY STEP

After the session completes, 5 parallel workers process the data:

```
Service Bus: post-interview topic receives message
{
  sessionId: "abc123",
  tenantId: "company-abc", 
  jobRoleId: "senior-engineer",
  candidateId: "cand-001",
  completedAt: "2025-01-01T15:30:00Z",
  recordingBlobUri: "blob://interview-recordings/company-abc/abc123/recording.mp4",
  transcriptText: "full interview transcript..."
}

This triggers 4 workers SIMULTANEOUSLY:
```

### Worker 1: Telemetry Aggregator
```
Reads all raw telemetry events from Cosmos DB for this sessionId
(Events were saved in real-time during interview)

Computes aggregated metrics:
  Average WPM per question:      [142, 155, 138, 161, 149]
  Filler words per question:     [2, 5, 1, 3, 7]  ← Q5 high → red flag
  Pause count per question:      [1, 3, 0, 2, 4]
  Avg pause duration per Q:      [1.2s, 2.8s, 0.9s, 1.5s, 3.1s]
  Response latency per Q:        [1.8s, 3.2s, 1.4s, 2.1s, 4.8s] ← Q5 long
  Total focus loss events:       1 (7 seconds, during Q3)
  Camera frame count:            336 frames captured

Saves AggregatedTelemetry record to Cosmos DB
Publishes: telemetry-aggregation-complete event
```

### Worker 2: Vision Analyser
```
Fetches all JPG frames from Blob Storage for this session
Groups frames by question (uses timestamp to determine which Q)

Processes every frame (336 frames total):
  
  For each frame → Azure AI Vision call:
    - Face count: 1 (pass) or 0 (no face) or 2+ (alert: multiple people)
    - Gaze direction: forward / left / right / down
    - Head pose: pitch, yaw, roll values

  For every 20th frame → GPT-4o Vision call (more expensive):
    Prompt: "Analyse this interview candidate. Rate on scale 1-10:
             1. Posture (upright vs. slouched)
             2. Engagement (attentive vs. distracted)
             3. Eye contact quality
             4. Professional appearance
             Return JSON: {posture, engagement, eyeContact, notes}"
    
Results computed:
  Eye contact ratio: 68% of frames showing forward gaze
  Multiple persons detected: 0 events
  Posture quality avg: 7.2/10
  Engagement avg: 7.8/10
  Frames with no face (looking away): 108/336 = 32%

Saves VisionAnalysisResult to Cosmos DB
```

### Worker 3: STAR & Content Scorer
```
Takes full transcript split by question

For each Q+A pair → GPT-4o call:
Prompt:
┌─────────────────────────────────────────────────────────────────────────┐
│ You are an expert interview evaluator. Score this answer:               │
│                                                                         │
│ Question: "Tell me about a time you handled a difficult stakeholder"    │
│                                                                         │
│ Answer: "At my last job, a product manager kept changing requirements.  │
│ I scheduled a weekly alignment meeting and created a shared doc for    │
│ decisions. This reduced last-minute changes by about 60%."             │
│                                                                         │
│ Score on these dimensions (JSON only, no explanation):                  │
│ {                                                                       │
│   "situation": 2.0,     // 0-2.5: Was context clear?                  │
│   "task": 2.0,          // 0-2.5: Was their role clear?               │
│   "action": 2.5,        // 0-2.5: Were concrete steps described?      │
│   "result": 2.0,        // 0-2.5: Was outcome quantified?             │
│   "specificity": 4,     // 0-5: Concrete vs. vague                    │
│   "relevance": 5,       // 0-5: On-topic                              │
│   "depth": 4            // 0-5: Insight level                         │
│ }                                                                       │
└─────────────────────────────────────────────────────────────────────────┘

Also computes across all answers:
  Vocabulary richness (Type-Token Ratio on full transcript)
  Response consistency (embeds each answer → cosine similarity checks)
  Authenticity score (detects scripted/memorised patterns)

Saves ContentScores to Cosmos DB
```

### Worker 4: Scoring Engine
```
Waits for Workers 1, 2, and 3 to complete
(listens for all 3 completion events on Service Bus)

Loads rubric from Redis (or Cosmos DB if not cached):
  technicalDepth: 35%, problemSolving: 25%, communication: 20%,
  behavioral: 15%, engagement: 5%

Calculates final score:

TECHNICAL DEPTH (35%):
  Average STAR score across technical Qs: 7.8/10
  Content depth score: 8.2/10
  Technical depth component: (7.8 + 8.2) / 2 × 35 = 28.0 points

PROBLEM SOLVING (25%):
  Specificity scores avg: 4.1/5
  Relevance scores avg: 4.6/5
  Problem solving component: ((4.1+4.6)/(10)) × 25 = 21.75 points

COMMUNICATION (20%):
  WPM score: 145avg → 90/100 (optimal range)
  Filler rate: 3.6/min → 70/100 (slightly high)
  Pause score: avg 1.8s → 85/100
  Vocabulary TTR: 0.68 → 82/100
  Communication component: avg(90,70,85,82) × 20% = 16.55 points

BEHAVIORAL STAR (15%):
  Avg STAR total across behavioral Qs: 7.6/10 → 76/100
  Behavioral component: 76 × 15% = 11.4 points

ENGAGEMENT (5%):
  Eye contact: 68% → 75/100
  Posture: 7.2/10 → 72/100
  Engagement component: avg(75,72) × 5% = 3.68 points

TOTAL: 28.0 + 21.75 + 16.55 + 11.4 + 3.68 = 81.38 → rounds to 81/100

Anti-cheat deductions (if any):
  Multiple persons detected: -10 points per event (0 events, no deduction)
  Excessive focus loss (> 30s total): -5 points (1 event × 7s = no deduction)
  Scripted response flag: -8 points per Q (0 flags, no deduction)

FINAL SCORE: 81/100 → Grade: B → Decision: "Hire"
```

### Worker 5: Report Generator
```
Receives ScoringComplete event from Service Bus
Loads: CandidateScore, session transcript, AggregatedTelemetry, AntiCheatLog

Calls GPT-4o to generate written sections:
  
  Input: all scores + transcript excerpts
  Output:
  {
    "executiveSummary": "Jane demonstrated strong technical depth with clear
                          problem-solving ability. She consistently provided
                          specific examples with measurable outcomes. Communication
                          was effective though slightly above optimal speaking pace.
                          Recommend for technical round.",
    
    "keyStrengths": [
      "Excellent technical specificity — always cited specific tools and metrics",
      "Strong STAR structure across all behavioral questions",
      "Calm and measured under hypothetical pressure scenarios"
    ],
    
    "developmentAreas": [
      "Filler word usage above average (3.6/min) — suggestion: deliberate pausing",
      "Closing answers slightly rushed — could expand on outcomes more"
    ],
    
    "redFlags": [],
    
    "questionInsights": [
      { "questionId": "q-003", "insight": "Hesitated 4.8s before answering system
                                            design question — possible knowledge gap" },
      { "questionId": "q-005", "insight": "Strongest answer — excellent quantification
                                            of business impact" }
    ]
  }

Generates PDF using QuestPDF:
  Page 1: Candidate info, overall score, decision
  Page 2: Competency radar chart + detailed scores
  Page 3: Behavioural signals breakdown + charts
  Page 4: Per-question scores + AI insights
  Page 5: Full transcript (condensed)

PDF saved to Blob Storage
SAS URL (48hr expiry) generated for secure access

Notifications sent:
  → SignalR push to recruiter browser (instant)
  → SendGrid email to recruiter with PDF attached
  → Webhook to ATS (if configured): POST to recruiter's ATS endpoint
    { sessionId, candidateId, score: 81, grade: "B", 
      decision: "Hire", reportUrl: "https://...", completedAt: "..." }
```

---

## Part 7 — REAL-TIME DATA FLOW DURING AN INTERVIEW

```
What is flowing where, every second:

Browser ──────────────────────────────────────── Azure
   │                                               │
   │  Microphone audio (PCM16, 24kHz)              │
   │ ─────────────────────────────────────────────▶│ GPT-4o Realtime WebSocket
   │                                               │   ↓ STT (converts to text)
   │                                               │   ↓ LLM (agent evaluates)
   │                                               │   ↓ TTS (generates speech)
   │  AI speech audio chunks                       │
   │◀─────────────────────────────────────────────  │ GPT-4o Realtime WebSocket
   │  Play via AudioContext                         │
   │                                               │
   │  Avatar video stream (WebRTC/HTTPS)            │
   │◀─────────────────────────────────────────────  │ Azure Neural Avatar Service
   │  Render in <video> element                     │
   │                                               │
   │  ACS call (separate audio/video stream)        │
   │ ─────────────────────────────────────────────▶│ Azure Communication Services
   │                                               │   ↓ Recording to Blob Storage
   │                                               │   ↓ (continuous, full session)
   │  Telemetry events (batch every 3s)             │
   │ ─────────────────────────────────────────────▶│ Telemetry API
   │  { wpm, fillerWords, pauses, focusEvents }     │   ↓ Cosmos DB (raw events)
   │                                               │
   │  Camera frame (JPG, every 5s)                  │
   │ ─────────────────────────────────────────────▶│ Telemetry API
   │                                               │   ↓ Blob Storage
   │                                               │   ↓ Service Bus → Vision Worker
   │  Heartbeat (every 15s)                         │
   │ ─────────────────────────────────────────────▶│ Interview API
   │  { sessionId, timestamp }                      │   ↓ Redis: session alive = true
   │                                               │
   │  Session state updates (SignalR, inbound)       │
   │◀─────────────────────────────────────────────  │ Azure SignalR Service
   │  { phase: "Technical", questionIndex: 3 }      │   (from backend state machine)
   │  Progress bar + phase indicator updates        │
```

---

## Part 8 — QUESTION TIME LIMITS & WHAT HAPPENS

```
Per-question time limit (recruiter-configured, default 3 mins):

Timer starts when agent finishes speaking the question.

  0:00  Candidate starts answering
  2:30  Gentle visual cue: progress bar turns amber "30 seconds remaining"
  2:50  Avatar says: "Take your time, no rush" (if candidate is still mid-answer)
  3:00  Timer ends:
          → If candidate still speaking: audio capture continues 10s more
          → "Thank you, let's move on" spoken by avatar
          → Answer marked as "time-limited" in question record
          → Next question starts
          
  Note: Candidates CAN take longer if recruiter disables time limits
        Some roles (Creative, Strategic) benefit from unconstrained depth
```

---

## Part 9 — ANTI-CHEAT: WHAT IS DETECTED AND HOW

```
DURING INTERVIEW (real-time):

Detection 1: TAB SWITCH / WINDOW FOCUS LOSS
  How: Browser `document.addEventListener('visibilitychange', ...)` 
  What: Every time candidate switches to another tab/app
  Action:
    - Log event with timestamp
    - After 2nd event: Avatar says "I noticed some activity on your screen.
                                    Please keep this window in focus."
    - After 3rd event: Integrity flag added to session
    - > 5 events: IntegrityLevel → "Suspicious"

Detection 2: MULTIPLE PEOPLE IN FRAME
  How: Azure AI Vision face count on every captured frame
  What: If frame shows 2+ faces
  Action:
    - Log event with screenshot reference
    - Avatar says: "I want to make sure this is your personal interview.
                    Please ensure you are alone."
    - > 2 events: IntegrityLevel → "Compromised", recruiter alerted

Detection 3: AUDIO FROM ANOTHER SOURCE
  How: GPT-4o Realtime detects multiple voice signatures in audio
  What: Someone coaching the candidate audibly
  Action: Flag event, mark as "possible external coaching"

AFTER INTERVIEW (post-processing):

Detection 4: SCRIPTED / MEMORISED RESPONSES
  How: Compute perplexity score of each answer using GPT-4o
       Low perplexity = very predictable text = possibly scripted
       Compare sentence structure across answers (suspiciously uniform style)
  Action: Authenticity score reduced, flag added to report

Detection 5: AI-GENERATED ANSWERS
  How: Multiple signals:
       - Unusually formal/perfect grammar throughout
       - Zero filler words (real humans have some)
       - Zero hesitation pauses
       - Vocabulary complexity inconsistent with speaking style
  Action: "Possible AI-assisted responses" flag in report (not automatic disqualify)

RECRUITER SEES in report:
  ┌─────────────────────────────────────────────────────────┐
  │  INTEGRITY REPORT                                       │
  │  Overall Integrity Level: ✅ Clean                      │
  │                                                         │
  │  ✅ No multiple persons detected (336 frames checked)   │
  │  ⚠️ 1 tab switch event (7 seconds, during Question 3)   │
  │  ✅ No scripted response patterns detected              │
  │  ✅ Audio analysis: single voice throughout             │
  │                                                         │
  │  Note: 1 tab switch is within normal tolerance.        │
  └─────────────────────────────────────────────────────────┘
```

---

## Part 10 — MULTI-LANGUAGE SUPPORT

```
Interview can be conducted in any language Azure Speech supports:
  - English (US, UK, AU, IN)
  - Spanish, French, German, Portuguese, Hindi, Japanese, etc.

Per job role config: 
  - Language: "en-US"
  - Avatar voice: "en-US-JennyNeural" (built-in) or custom neural voice
  - STT model: en-US (Azure Speech language model)
  - GPT-4o evaluation: conducted in the same language
  - Report: generated in recruiter's configured language (can differ)

Multi-language interview:
  - Not supported in Phase 1 (language fixed per session)
  - Phase 2: Candidate can indicate preferred language at start
    → system switches avatar voice + STT model mid-session
```

---

## Part 11 — SCALABILITY: HOW THE SYSTEM HANDLES LOAD

```
100 simultaneous interviews running:

Each interview consumes:
  - 1 GPT-4o Realtime WebSocket connection
  - 1 ACS call
  - 1 Agent thread (Foundry)
  - ~20 telemetry API calls/min
  - 12 vision frames/min

Container App autoscaling:
  - interview-api: scales up to 20 replicas (CPU-triggered)
  - telemetry-api: scales up to 50 replicas (handles highest volume)
  
GPT-4o Realtime quota management:
  - Each tenant has configured quota (e.g. 10 concurrent sessions)
  - Session start is rejected with 429 if quota exceeded
  - Queuing system for peak periods (session scheduled for next available slot)

Cosmos DB:
  - Autoscale handles spikes automatically
  - Partition key = tenantId (ensures hot partitions don't occur)
  - telemetry container can handle 10,000 events/sec across partitions

Service Bus:
  - Premium tier: isolated capacity, no noisy neighbours
  - Workers scale horizontally based on queue depth (KEDA)
  - 100 interviews completing simultaneously → 100 post-processing jobs
    Workers scale to handle: 30 scoring workers, 10 report workers
```

---

## Part 12 — WHAT GETS STORED WHERE (COMPLETE REFERENCE)

```
COSMOS DB CONTAINERS:
┌─────────────────┬────────────────────────────────────────────────────┐
│ Container       │ What's stored                                       │
├─────────────────┼────────────────────────────────────────────────────┤
│ tenants         │ Company account details, subscription tier,         │
│                 │ Azure service quotas, branding config               │
├─────────────────┼────────────────────────────────────────────────────┤
│ users           │ Recruiter accounts, roles, permissions              │
├─────────────────┼────────────────────────────────────────────────────┤
│ roles           │ Job role definitions, interview config,             │
│                 │ question bank references, phase structure           │
├─────────────────┼────────────────────────────────────────────────────┤
│ questions       │ All questions per tenant/role, follow-up banks,     │
│                 │ scoring hints, difficulty levels                    │
├─────────────────┼────────────────────────────────────────────────────┤
│ rubrics         │ Scoring weights, decision thresholds,               │
│                 │ enabled signal list, blind mode settings            │
├─────────────────┼────────────────────────────────────────────────────┤
│ candidates      │ Candidate records, contact info (encrypted),        │
│                 │ consent records, GDPR erasure requests              │
├─────────────────┼────────────────────────────────────────────────────┤
│ sessions        │ Interview session state, phase tracking,            │
│                 │ questions asked, agent thread ID, ACS call ID       │
├─────────────────┼────────────────────────────────────────────────────┤
│ transcripts     │ Full word-by-word transcript with timestamps        │
│                 │ per session, word confidence scores                 │
├─────────────────┼────────────────────────────────────────────────────┤
│ telemetry       │ Raw behavioural events (WPM samples, filler words,  │
│                 │ pauses, focus events) — TTL: 90 days               │
├─────────────────┼────────────────────────────────────────────────────┤
│ scores          │ Final CandidateScore records with all 14 dimensions,│
│                 │ executive summary, GDPR-safe (no PII)               │
├─────────────────┼────────────────────────────────────────────────────┤
│ notifications   │ Outbound webhook log, delivery status, retries      │
└─────────────────┴────────────────────────────────────────────────────┘

BLOB STORAGE:
┌─────────────────────────────────┬────────────────────────────────────┐
│ Path                            │ Contents                           │
├─────────────────────────────────┼────────────────────────────────────┤
│ interview-recordings/           │ Full MP4 recording of each session │
│   {tenantId}/{sessionId}/       │ (audio + candidate video + avatar) │
│   recording.mp4                 │ Access: SAS token only             │
├─────────────────────────────────┼────────────────────────────────────┤
│ frame-snapshots/                │ JPG frames from candidate camera   │
│   {tenantId}/{sessionId}/       │ Used for vision analysis           │
│   {timestamp}.jpg               │ Deleted after 7 days               │
├─────────────────────────────────┼────────────────────────────────────┤
│ generated-reports/              │ PDF scorecard reports              │
│   {tenantId}/{candidateId}/     │ Retained per GDPR policy           │
│   {date}.pdf                    │ Access: SAS token only             │
├─────────────────────────────────┼────────────────────────────────────┤
│ avatar-cache/                   │ Pre-rendered avatar video clips    │
│   {character}/                  │ for common phrases (greetings,     │
│   {text-hash}.mp4               │ transitions) — avoids re-rendering │
└─────────────────────────────────┴────────────────────────────────────┘

REDIS CACHE:
┌─────────────────────────────────┬────────────────────────────────────┐
│ Key Pattern                     │ Contents / TTL                     │
├─────────────────────────────────┼────────────────────────────────────┤
│ session:{id}:state              │ Current phase, last Q, phase index │
│                                 │ TTL: 4 hours (session duration)    │
├─────────────────────────────────┼────────────────────────────────────┤
│ session:{id}:heartbeat          │ "alive" → TTL: 30s                 │
│                                 │ Missing = session abandoned        │
├─────────────────────────────────┼────────────────────────────────────┤
│ rubric:{tenantId}:{roleId}      │ Cached rubric JSON                 │
│                                 │ TTL: 5 minutes                     │
├─────────────────────────────────┼────────────────────────────────────┤
│ ratelimit:{tenantId}:{endpoint} │ Request counter for rate limiting  │
│                                 │ TTL: sliding 1-minute window       │
├─────────────────────────────────┼────────────────────────────────────┤
│ quota:{tenantId}:concurrent     │ Number of active sessions          │
│                                 │ TTL: never (decremented on end)    │
└─────────────────────────────────┴────────────────────────────────────┘
```

---

## Part 13 — SUMMARY: END-TO-END TIMELINE

```
T+0:00    Recruiter sends invite email to candidate
T+0:00    Cosmos: Session created (Status: WaitingCandidate)
T+0:00    SendGrid: Invite email delivered

T+{varies} Candidate opens link, passes consent + equipment check
T+{varies} Browser: getUserMedia() grants camera + mic

T+{s}:00  Candidate clicks "Start Interview"
T+{s}:01  Interview API: ACS call created, recording starts → Blob
T+{s}:01  Azure AI Foundry: Agent thread created, system prompt injected
T+{s}:01  Browser: WebSocket opened to GPT-4o Realtime
T+{s}:02  Avatar: Greeting video starts playing to candidate
T+{s}:02  Cosmos: Session status → Active

T+{s}:02  ——— INTERVIEW RUNS (avg 28 minutes) ———
          Every 3s: telemetry batch → Cosmos
          Every 5s: camera frame → Blob + Vision queue
          Every 15s: heartbeat → Redis
          Continuous: audio → GPT-4o Realtime ↔ avatar response loop

T+{e}:00  Interview complete (all questions done)
T+{e}:00  ACS: Call recording finalised → Blob MP4 written
T+{e}:00  Service Bus: post-interview message published
T+{e}:00  Cosmos: Session status → Completed
T+{e}:00  Candidate: "Thank you" screen shown

T+{e}:01  Workers start (parallel):
          Worker 1: Telemetry aggregation
          Worker 2: Vision frame analysis (336 frames)
          Worker 3: STAR + content scoring (per Q, via GPT-4o)
          Worker 4: (waits for 1+2+3)

T+{e}+03m Vision analysis complete
T+{e}+04m STAR scoring complete
T+{e}+04m Telemetry aggregation complete

T+{e}+04m Worker 4: Scoring engine runs → CandidateScore written
T+{e}+05m Worker 5: GPT-4o generates executive summary + insights
T+{e}+06m Worker 5: PDF report generated + uploaded to Blob
T+{e}+06m SignalR: push to recruiter browser "Report ready"
T+{e}+06m Email: recruiter receives notification with link
T+{e}+06m Webhook: ATS system receives interview.completed event

Total post-interview processing time: ~6 minutes
```

---

> **This is the full operational blueprint.** Every service, every flow, every data store, every decision point is documented. Ready to begin implementation starting from the .NET API scaffold and Azure service provisioning.
