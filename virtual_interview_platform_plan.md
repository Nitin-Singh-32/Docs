# 🏗️ AI Virtual Interview Platform — Production-Ready Engineering Blueprint
### Azure AI Foundry · .NET 9 · Next.js 14 · Enterprise-Grade Architecture

---

## Table of Contents
1. [Product Vision & Scope](#1-product-vision--scope)
2. [Multi-Tenant SaaS Architecture](#2-multi-tenant-saas-architecture)
3. [Complete System Architecture](#3-complete-system-architecture)
4. [All Azure Services & Configuration](#4-all-azure-services--configuration)
5. [Data Models & Schema Design](#5-data-models--schema-design)
6. [Complete REST API Contract](#6-complete-rest-api-contract)
7. [Interview Agent Intelligence Design](#7-interview-agent-intelligence-design)
8. [Behavioral Analysis Engine (14 Dimensions)](#8-behavioral-analysis-engine-14-dimensions)
9. [Real-Time AV Pipeline](#9-real-time-av-pipeline)
10. [Security, Auth & Compliance](#10-security-auth--compliance)
11. [Anti-Cheat & Integrity System](#11-anti-cheat--integrity-system)
12. [Observability & Monitoring](#12-observability--monitoring)
13. [Infrastructure as Code (Bicep)](#13-infrastructure-as-code-bicep)
14. [CI/CD Pipeline (GitHub Actions)](#14-cicd-pipeline-github-actions)
15. [Resilience & Fault Tolerance](#15-resilience--fault-tolerance)
16. [Cost Model & Optimization](#16-cost-model--optimization)
17. [Testing Strategy](#17-testing-strategy)
18. [Complete Project Structure](#18-complete-project-structure)
19. [16-Week Implementation Roadmap](#19-16-week-implementation-roadmap)

---

## 1. Product Vision & Scope

### What Is Being Built
A **multi-tenant, enterprise-grade, AI-powered virtual interview platform** that:
- Conducts adaptive, conversational video interviews via a photorealistic AI avatar
- Analyzes candidate behaviour across 14 dimensions in real-time
- Produces structured, bias-audited competency reports for recruiters
- Integrates with existing ATS (Applicant Tracking Systems) via webhooks

### User Personas
| Persona | Role | Primary Interface |
|:---|:---|:---|
| **Candidate** | Takes the interview | Interview Room (browser, no install) |
| **Recruiter** | Reviews results | Recruiter Dashboard |
| **Hiring Manager** | Approves shortlists | Read-only Report View |
| **Platform Admin** | Configures tenants, rubrics | Admin Portal |
| **System** | Automated processing | Background Workers |

### Scope Boundaries
**In Scope:** Session management, real-time audio/video, behavioral analysis, avatar, scoring, recruiter dashboard, anti-cheat, multi-tenancy, GDPR compliance, reporting.

**Out of Scope (Phase 1):** Native mobile apps, direct ATS integrations (Phase 2 webhooks only), custom avatar face training.

---

## 2. Multi-Tenant SaaS Architecture

### Tenancy Model: **Shared Infrastructure, Isolated Data**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PLATFORM CONTROL PLANE                              │
│  Admin Portal  │  Tenant Provisioning Service  │  Billing & Usage Metering  │
└──────────────────────────────┬──────────────────────────────────────────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
      ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
      │  Tenant A    │ │  Tenant B    │ │  Tenant C    │
      │  (Acme Corp) │ │  (TechCorp)  │ │  (StartupXYZ)│
      │  Own DB      │ │  Own DB      │ │  Shared DB   │
      │  Own Keys    │ │  Own Keys    │ │  (SMB Tier)  │
      └──────────────┘ └──────────────┘ └──────────────┘
```

### Tenancy Tiers
| Tier | DB Isolation | Storage | Compute | Price |
|:---|:---|:---|:---|:---|
| **Enterprise** | Dedicated Cosmos DB | Dedicated Blob Container | Reserved Container App instances | Custom |
| **Business** | Shared DB, partitioned by tenantId | Shared + path isolation | Shared compute with HPA | $$$  |
| **Starter** | Shared DB, partitioned | Shared | Shared | $ |

### Tenant Resolution Flow
```
Request → API Gateway → Extract X-Tenant-Id header OR subdomain
       → TenantContextMiddleware resolves TenantConfig
       → Injects tenant-scoped services (DB, Keys, Storage)
       → All queries auto-filtered by tenantId
```

---

## 3. Complete System Architecture

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                         AZURE FRONT DOOR (Global CDN + WAF)                 ║
╚══════════════════════════════════════════╦═══════════════════════════════════╝
                                           ║
              ┌────────────────────────────┼────────────────────────────┐
              ▼                            ▼                            ▼
  ┌─────────────────────┐    ┌─────────────────────┐    ┌─────────────────────┐
  │  CANDIDATE APP       │    │  RECRUITER DASHBOARD │    │  ADMIN PORTAL        │
  │  Next.js 14          │    │  Next.js 14          │    │  Next.js 14          │
  │  interview.{host}   │    │  app.{host}          │    │  admin.{host}        │
  │  Azure Static Web   │    │  Azure Static Web    │    │  Azure Static Web    │
  └────────┬────────────┘    └──────────┬───────────┘    └──────────┬──────────┘
           │ HTTPS/WSS                  │ HTTPS                     │ HTTPS
           ▼                            ▼                            ▼
╔══════════════════════════════════════════════════════════════════════════════╗
║                    AZURE API MANAGEMENT (APIM)                               ║
║        Rate Limiting · API Keys · OAuth2 Validation · Routing                ║
╚═══════════════════════════════════════╦════════════════════════════════════ ═╝
                                        ║
              ┌─────────────────────────┼──────────────────────────┐
              ▼                         ▼                          ▼
  ┌─────────────────────┐  ┌─────────────────────┐  ┌──────────────────────┐
  │  INTERVIEW API       │  │  TELEMETRY API       │  │  ADMIN API            │
  │  (.NET 9 + ASP.NET) │  │  (.NET 9 + ASP.NET) │  │  (.NET 9 + ASP.NET)  │
  │  Session, Agents,   │  │  Behavioral events,  │  │  Tenant mgmt, Rubrics│
  │  Scoring, Reports   │  │  Vision frames, Logs │  │  User mgmt, Config   │
  │  Azure Container App│  │  Azure Container App │  │  Azure Container App │
  └────────┬────────────┘  └──────────┬───────────┘  └──────────────────────┘
           │                          │
     ┌─────┴──────────────────────────┴───────┐
     │           INTERNAL SERVICE LAYER        │
     ├─────────────────────────────────────────┤
     │                                         │
     ▼                   ▼                     ▼
┌──────────────┐  ┌──────────────┐    ┌──────────────────┐
│ Azure AI     │  │ Azure AI     │    │ Azure AI Vision   │
│ Foundry      │  │ Speech Svc   │    │ + GPT-4o Vision   │
│ Agent Svc    │  │ STT/TTS/     │    │ Frame analysis    │
│ GPT-4o RT    │  │ Pronunciation│    │ (async batch)     │
│ Evaluation   │  │ Assessment   │    │                   │
└──────────────┘  └──────────────┘    └──────────────────┘
     │                   │                     │
     └───────────────────┴─────────────────────┘
                         │
          ┌──────────────┴───────────────┐
          ▼                              ▼
┌─────────────────────┐       ┌───────────────────────┐
│ Azure Service Bus   │       │  EVENT PROCESSING WORKERS│
│ Topics:             │       │  Azure Container Jobs   │
│ · interview-events  │──────▶│  · TelemetryAggregator  │
│ · telemetry-raw     │       │  · PostInterviewProcessor│
│ · notifications     │       │  · RecordingArchiver     │
│ · scoring-complete  │       │  · ReportGenerator       │
└─────────────────────┘       │  · NotificationDispatcher│
                              └───────────────┬───────────┘
                                              │
              ┌───────────────────────────────┼──────────────────────────────┐
              ▼                               ▼                              ▼
  ┌─────────────────────┐       ┌──────────────────────┐      ┌──────────────────┐
  │  AZURE COSMOS DB     │       │  AZURE BLOB STORAGE   │      │ AZURE CACHE      │
  │  (Multi-container)  │       │  Interview recordings  │      │ FOR REDIS        │
  │  · sessions          │       │  Frame snapshots       │      │ Session state    │
  │  · candidates        │       │  Generated reports     │      │ Rubric cache     │
  │  · telemetry         │       │  Avatar video cache    │      │ Rate limit state │
  │  · scores            │       │                        │      │                  │
  │  · tenants           │       │                        │      │                  │
  └─────────────────────┘       └──────────────────────┘      └──────────────────┘
```

### Real-Time Audio/Video Sub-Architecture

```
CANDIDATE BROWSER                     AZURE CLOUD
┌────────────────────┐                ┌─────────────────────────────────────┐
│  getUserMedia()    │                │  Azure Communication Services (ACS)  │
│  Camera + Mic      │◄──── WebRTC ──▶│  · STUN/TURN servers                │
│  Stream            │                │  · Call Recording (MP4)             │
└─────────┬──────────┘                │  · Callback webhooks                │
          │                           └──────────────┬──────────────────────┘
          │ WebSocket                                │
          ▼                                          │
┌────────────────────┐                              │
│  GPT-4o Realtime   │◄─────────────────────────────┘
│  Audio API         │  Bidirectional PCM16 audio
│  (via Azure AOAI)  │  
│  · VAD built-in    │──── Text Events ────▶ Interview API
│  · Transcripts     │                      (session state update)
└────────────────────┘
          │
          ▼
┌────────────────────┐
│  Azure Neural TTS  │
│  Avatar Service    │
│  · Lip-sync video  │──── Video Stream ────▶ Candidate Browser
│  · SSML speech     │                        <video> element
└────────────────────┘
```

---

## 4. All Azure Services & Configuration

### 4.1 Azure AI Foundry
**Purpose:** Core interview agent orchestration, GPT-4o, Evaluation SDK

```
Deployment: Hub → Project → Agent
- Hub: interview-platform-hub (East US 2)
- Project: InterviewPlatform
- Agent: InterviewerAgent (per-tenant configurable persona)
- Models deployed:
  - gpt-4o (2024-11-20) → primary interview + scoring
  - gpt-4o-realtime-preview → voice streaming
  - text-embedding-3-large → transcript embedding + similarity
```

**Quota planning:**
```
gpt-4o:         500K TPM per tenant (Enterprise), 100K TPM (Business)
gpt-4o-realtime: 10 concurrent sessions (Enterprise), 3 (Business)
```

### 4.2 Azure AI Speech Service
**Purpose:** STT, TTS, Pronunciation Assessment, Speaker Diarization

```
SKU: S0 (Standard) — required for custom neural voice + avatar
Region: East US (align with ACS for lowest latency)
Features enabled:
  - Real-time speech-to-text (en-US + multi-language)
  - Custom Neural Voice (branded interviewer voice)
  - Pronunciation Assessment (accuracy, fluency, prosody)
  - Speaker Recognition (anti-spoofing verification)
  - Azure Neural TTS Avatar (lisa / jenny — business style)
```

### 4.3 Azure Communication Services (ACS)
**Purpose:** WebRTC signaling, call recording, identity tokens

```
Features:
  - Calling SDK: WebRTC for candidate media
  - Call Recording: MP4 to Blob Storage (automatic)
  - Communication Identity: Ephemeral tokens per candidate session
  - Event Grid integration: Recording ready → Service Bus → Worker
```

### 4.4 Azure API Management (APIM)
**Purpose:** Gateway, rate limiting, API versioning, auth validation

```
Tier: Standard v2
Policies applied:
  - JWT validation (Entra ID)
  - Rate limit: 100 req/min per tenant (Candidate API)
  - Rate limit: 500 req/min per tenant (Recruiter API)
  - CORS: tenant-allowlisted origins only
  - Request/Response logging → Application Insights
  - Circuit breaker via retry policies
```

### 4.5 Azure Cosmos DB
**Purpose:** Primary operational database

```
API: NoSQL (Core SQL API)
Consistency: Session consistency
Regions: East US 2 (primary), West Europe (read replica)
Containers:
  - sessions        → partitionKey: /tenantId
  - candidates      → partitionKey: /tenantId
  - telemetry       → partitionKey: /sessionId  (TTL: 90 days raw)
  - scores          → partitionKey: /tenantId
  - transcripts     → partitionKey: /sessionId
  - rubrics         → partitionKey: /tenantId
  - tenants         → partitionKey: /id
  - notifications   → partitionKey: /tenantId
Throughput: Autoscale (400–40,000 RU/s) per container
```

### 4.6 Azure Blob Storage
**Purpose:** Media storage (recordings, frames, reports)

```
Containers:
  - interview-recordings/{tenantId}/{sessionId}/recording.mp4
  - frame-snapshots/{tenantId}/{sessionId}/{timestamp}.jpg
  - generated-reports/{tenantId}/{candidateId}/{date}.pdf
  - avatar-cache/{character}/{text-hash}.mp4
Lifecycle policies:
  - Raw frames: delete after 7 days (processed)
  - Recordings: move to Cool tier after 30 days
  - Reports: retain for 2 years (compliance)
Access: Private, accessed via SAS tokens (short-lived, scoped)
```

### 4.7 Azure Cache for Redis
**Purpose:** Session state, distributed locks, rate limit counters

```
SKU: C2 (Standard, 6GB) — with geo-replication
Patterns:
  - Session heartbeat: key = session:{id}:heartbeat, TTL = 30s
  - Distributed lock: key = lock:session:{id}, TTL = 10s (Redlock)
  - Rate limiting: key = ratelimit:{tenantId}:{endpoint}, sliding window
  - Rubric cache: key = rubric:{tenantId}:{roleId}, TTL = 5min
  - Feature flags: key = features:{tenantId}, TTL = 60s
```

### 4.8 Azure Service Bus
**Purpose:** Async event pipeline

```
Namespace: interview-platform-bus
SKU: Premium (required for VNet integration + sessions)
Topics & Subscriptions:
  - topic: interview-events
      sub: session-state-machine (Interview API)
      sub: recruiter-notifications (Notification Worker)
  - topic: telemetry-raw
      sub: aggregator (Telemetry Aggregator Worker)
  - topic: post-interview
      sub: vision-processor (Vision Worker)
      sub: scoring-engine (Scoring Worker)
      sub: report-generator (Report Worker)
  - topic: notifications
      sub: email-sender
      sub: webhook-dispatcher (ATS integrations)
Dead-letter queues: all topics (with 7-day retention)
```

### 4.9 Azure Container Apps
**Purpose:** Hosting all backend services

```
Environment: interview-platform-env
Services:
  - interview-api          → min: 2, max: 20 replicas, CPU trigger
  - telemetry-api          → min: 2, max: 50 replicas, HTTP trigger
  - admin-api              → min: 1, max: 5 replicas
  - telemetry-worker       → min: 1, max: 30, Service Bus trigger
  - post-interview-worker  → min: 0, max: 20, Service Bus trigger
  - vision-worker          → min: 0, max: 10, Service Bus trigger
  - scoring-worker         → min: 0, max: 20, Service Bus trigger
  - report-worker          → min: 0, max: 10, Service Bus trigger
  - notification-worker    → min: 1, max: 5, Service Bus trigger
KEDA scalers: Azure Service Bus message count
```

### 4.10 Azure Static Web Apps
**Purpose:** Hosting Next.js frontends

```
3 separate apps:
  - interview-candidate-app (interview.{yourdomain}.com)
  - recruiter-dashboard-app (app.{yourdomain}.com)
  - admin-portal-app (admin.{yourdomain}.com)
Region: Global (CDN-backed)
Auth: Linked to Entra ID (recruiter/admin apps)
Custom domains + managed SSL certificates
```

### 4.11 Azure Front Door + WAF
**Purpose:** Global routing, DDoS, WAF, CDN

```
WAF Policy (Prevention mode):
  - OWASP 3.2 ruleset
  - Custom rules: block IPs with > 50 failed auth attempts
  - Bot protection enabled
  - Rate limit: 1000 req/10min per IP
Routing rules:
  - interview.* → Static Web App (Candidate)
  - app.* → Static Web App (Recruiter)
  - api.* → APIM → Container Apps
  - admin.* → Static Web App (Admin)
```

### 4.12 Azure Monitor + Application Insights
**Purpose:** Full-stack observability

```
App Insights per service (interview-api, telemetry-api, workers)
Custom metrics pushed:
  - session.duration, session.completionRate
  - behavioral.confidenceScore, behavioral.fillerRate
  - agent.tokenUsage, agent.responseLatency
  - scoring.processingTime, report.generationTime
Workbooks: Custom operational dashboards
Alerts: P1 (SMS+Email), P2 (Email), P3 (Dashboard)
Log Analytics workspace: 30-day hot, 90-day archive
```

### 4.13 Microsoft Entra ID (Azure AD)
**Purpose:** Authentication and authorization

```
App Registrations:
  - InterviewPlatform.RecruiterApp (SPA — PKCE flow)
  - InterviewPlatform.AdminApp (SPA — PKCE flow)
  - InterviewPlatform.API (daemon — client credentials)
  - InterviewPlatform.CandidateApp (public client — anonymous start)
Roles defined:
  - Platform.Admin
  - Tenant.Admin
  - Recruiter
  - HiringManager (read-only)
Groups: Per-tenant mapped groups
```

### 4.14 Azure Key Vault
**Purpose:** Secrets management

```
One Key Vault per environment (dev, staging, prod)
Secrets stored:
  - cosmos-connection-string
  - speech-subscription-key
  - acs-connection-string
  - redis-connection-string
  - service-bus-connection-string
  - ai-foundry-api-key
  - jwt-signing-key
  - webhook-signing-secrets/{tenantId}
Access: Managed Identity for all Container Apps (no secret in env vars)
```

---

## 5. Data Models & Schema Design

### 5.1 InterviewSession (Cosmos DB)

```csharp
public record InterviewSession
{
    [JsonPropertyName("id")]
    public string Id { get; init; } = Guid.NewGuid().ToString();
    
    public string TenantId { get; init; }
    public string JobRoleId { get; init; }
    public string CandidateId { get; init; }
    public string AgentThreadId { get; init; }     // Foundry Agent thread
    public string AcsCallId { get; init; }          // ACS call recording ref
    
    public SessionStatus Status { get; set; }       // Created|WaitingCandidate|Active|Completed|Abandoned|Error
    public SessionPhase CurrentPhase { get; set; }  // Intro|Warmup|Technical|Behavioral|Closing|Complete
    
    public DateTimeOffset CreatedAt { get; init; }
    public DateTimeOffset? StartedAt { get; set; }
    public DateTimeOffset? CompletedAt { get; set; }
    public int DurationSeconds { get; set; }
    
    public List<QuestionRecord> Questions { get; set; } = [];
    public SessionConfig Config { get; init; }       // Rubric, question count, time limits
    public SessionMetadata Metadata { get; set; }    // Browser info, IP (hashed), device
    public AntiCheatLog AntiCheatLog { get; set; } = new();
    public string? RecordingBlobUri { get; set; }
    public string? TranscriptBlobUri { get; set; }
    
    // TTL: -1 (never) for completed sessions, 7 days for abandoned
    public int? Ttl { get; set; }
}

public enum SessionStatus { Created, WaitingCandidate, Active, Completed, Abandoned, Error }
public enum SessionPhase  { Intro, Warmup, Technical, Behavioral, Closing, Complete }

public record QuestionRecord
{
    public string QuestionId { get; init; }
    public string QuestionText { get; init; }
    public QuestionType Type { get; init; }   // Technical|Behavioral|Situational|Warmup
    public int OrderIndex { get; init; }
    public DateTimeOffset AskedAt { get; init; }
    public DateTimeOffset? AnsweredAt { get; set; }
    public int ResponseDurationSeconds { get; set; }
    public string? FollowUpQuestionId { get; set; }  // null if not probed
}
```

### 5.2 CandidateScore (Cosmos DB)

```csharp
public record CandidateScore
{
    public string Id { get; init; } = Guid.NewGuid().ToString();
    public string TenantId { get; init; }
    public string SessionId { get; init; }
    public string CandidateId { get; init; }
    public string JobRoleId { get; init; }
    
    // Overall
    public double OverallScore { get; set; }          // 0–100, weighted composite
    public ScoreGrade Grade { get; set; }             // A|B|C|D|F
    public ScoreConfidence Confidence { get; set; }   // High|Medium|Low (based on data completeness)
    
    // Competency scores (0–100 each)
    public CompetencyScores Competencies { get; set; }
    
    // Behavioral signal scores (0–100 each)
    public BehavioralScores BehavioralSignals { get; set; }
    
    // Per-question scores
    public List<QuestionScore> QuestionScores { get; set; } = [];
    
    // AI-generated insights
    public string ExecutiveSummary { get; set; }
    public List<string> KeyStrengths { get; set; } = [];
    public List<string> DevelopmentAreas { get; set; } = [];
    public List<string> RedFlags { get; set; } = [];
    public string RecommendedDecision { get; set; }   // StrongHire|Hire|Borderline|NoHire
    
    // Bias audit trail
    public BiasAuditRecord BiasAudit { get; set; }
    
    public DateTimeOffset ScoredAt { get; set; }
    public string ScoringVersion { get; set; }         // rubric version used
}

public record CompetencyScores
{
    public double TechnicalDepth { get; set; }
    public double ProblemSolving { get; set; }
    public double Communication { get; set; }
    public double CriticalThinking { get; set; }
    public double Adaptability { get; set; }
    public double TeamCollaboration { get; set; }
    public double LeadershipPotential { get; set; }   // optional, role-dependent
    public double RoleSpecificSkill { get; set; }     // role-defined rubric
}

public record BehavioralScores
{
    public double VocalConfidence { get; set; }
    public double SpeechClarity { get; set; }
    public double PacingScore { get; set; }
    public double FillerWordScore { get; set; }        // inverted (lower fillers = higher score)
    public double HesitationScore { get; set; }       // inverted
    public double EyeContactScore { get; set; }
    public double PostureScore { get; set; }
    public double EmotionalResilience { get; set; }
    public double ResponseLatencyScore { get; set; }
    public double VocabularyRichness { get; set; }
    public double StarAdherenceScore { get; set; }
    public double FocusScore { get; set; }            // inverted (tab switches)
    public double ConsistencyScore { get; set; }      // cross-question consistency
    public double AuthenticityScore { get; set; }     // anti-scripted-response detection
}
```

### 5.3 BehavioralTelemetry (Cosmos DB — high write)

```csharp
public record BehavioralTelemetryEvent
{
    public string Id { get; init; } = Guid.NewGuid().ToString();
    public string SessionId { get; init; }
    public string TenantId { get; init; }
    public string QuestionId { get; init; }
    public TelemetryEventType EventType { get; init; }
    public DateTimeOffset Timestamp { get; init; }
    public double? OffsetSeconds { get; init; }    // position in recording
    public JsonElement Payload { get; init; }       // flexible per event type
    public int Ttl { get; init; } = 7776000;       // 90 days TTL
}

// EventType enum:
// SpeechStart, SpeechEnd, FillerWordDetected, LongPause, WpmSample,
// GazeLost, GazeRestored, PostureAlert, TabFocusLost, TabFocusRestored,
// MultiplePersonDetected, AgentQuestionDelivered, CandidateResponseComplete,
// EmotionSample, AudioLevelSample
```

---

## 6. Complete REST API Contract

### Interview API (Base: /api/v1)

#### Sessions
```
POST   /sessions                    → Create session, returns sessionId + candidate join URL
GET    /sessions/{id}               → Get session details + current status
PATCH  /sessions/{id}/status        → Update session status (admin/internal only)
DELETE /sessions/{id}               → Cancel session (before start)
POST   /sessions/{id}/start         → Candidate calls: starts ACS call, opens agent thread
POST   /sessions/{id}/complete      → Signals interview complete, triggers post-processing
GET    /sessions/{id}/join-token    → Returns ACS token + WebSocket config for candidate

POST   /sessions/{id}/heartbeat     → Candidate keep-alive (every 15s)
GET    /sessions/{id}/status-stream → SSE endpoint for real-time status updates
```

#### Scoring & Reports
```
GET    /sessions/{id}/score         → Get candidate score (processed)
GET    /sessions/{id}/report        → Get full behavioral report JSON
GET    /sessions/{id}/report/pdf    → Download PDF report (SAS URL)
GET    /sessions/{id}/recording     → Get recording SAS URL (time-limited)
GET    /sessions/{id}/transcript    → Get full transcript
```

#### Telemetry (Telemetry API — high throughput)
```
POST   /telemetry/events            → Batch behavioral events (candidate browser → API)
POST   /telemetry/frame             → Vision frame upload (browser captures every 5s)
POST   /telemetry/audio-metrics     → Speech SDK metrics from browser/SDK
GET    /telemetry/sessions/{id}/live → SSE: live behavioral summary for monitoring
```

#### Administration
```
GET    /rubrics                     → List rubrics for tenant
POST   /rubrics                     → Create new scoring rubric
GET    /rubrics/{id}                → Get rubric detail
PUT    /rubrics/{id}                → Update rubric
DELETE /rubrics/{id}                → Delete rubric

GET    /roles                       → List job roles
POST   /roles                       → Create job role (links rubric + question bank)
GET    /roles/{id}/questions        → Get question bank for role
POST   /roles/{id}/questions        → Add questions

GET    /candidates                  → List candidates (paginated, filterable)
POST   /candidates                  → Create candidate record + generate invite link
GET    /candidates/{id}/sessions    → Get all sessions for candidate

GET    /analytics/overview          → Dashboard summary stats
GET    /analytics/sessions          → Session analytics (completion rate, avg score, etc.)
GET    /analytics/candidates/compare → Multi-candidate comparison for a role
```

#### Webhooks (Outbound to ATS)
```
Events emitted:
  - interview.completed    → { sessionId, candidateId, score, grade, reportUrl }
  - interview.abandoned    → { sessionId, candidateId, reason }
  - interview.flagged      → { sessionId, candidateId, flags: [...] }
Delivery: HMAC-SHA256 signed, retry with exponential backoff
```

---

## 7. Interview Agent Intelligence Design

### 7.1 Agent State Machine

```
                    ┌─────────────────┐
                    │  SESSION CREATED │
                    └────────┬────────┘
                             │ Candidate joins
                    ┌────────▼────────┐
                    │     INTRO        │ Avatar introduces itself, explains format
                    │  (30-60 seconds) │
                    └────────┬────────┘
                             │ Intro complete
                    ┌────────▼────────┐
                    │   WARM-UP (1-2Q)│ Lightweight Qs: "Tell me about yourself"
                    └────────┬────────┘
                             │
               ┌─────────────┴─────────────┐
               ▼                           ▼
    ┌────────────────────┐     ┌─────────────────────┐
    │  TECHNICAL MODULE  │     │  BEHAVIORAL MODULE  │
    │  (3-5 questions)   │     │  (2-4 questions)    │
    │  Adaptive depth    │     │  STAR-based probing  │
    └─────────┬──────────┘     └──────────┬──────────┘
              └────────────┬──────────────┘
                           ▼
                  ┌─────────────────┐
                  │    CLOSING      │ Candidate Q&A, thank you
                  └────────┬────────┘
                           │
                  ┌────────▼────────┐
                  │   COMPLETE      │ Triggers post-processing pipeline
                  └─────────────────┘

Each state transition logged as event on Service Bus
```

### 7.2 Adaptive Question Logic

```
Per question cycle:
  1. Agent evaluates last response (streaming GPT-4o call)
  2. Computes QuestionScore [0-10]:
       < 4  → Generate follow-up probe (max 1 probe per Q)
       4-7  → Acknowledge + next question
       > 7  → Acknowledge warmly + escalate difficulty next Q
  3. Maintains difficulty level [Foundational|Standard|Advanced|Expert]
     - Difficulty auto-adjusts each question based on rolling average
  4. Detects topic exhaustion (same answer pattern twice) → moves on

Follow-up prompt template:
  "Thanks for sharing that. Could you give me a specific example of
   a time when [extracted key point from their answer]?"
```

### 7.3 System Prompt Architecture

```
LAYER 1 — Platform system prompt (immutable, hardcoded)
  - Core persona rules, safety guardrails
  - NEVER reveal rubric, NEVER discuss scoring
  - Maintain professional courtesy always
  - End session if candidate is abusive

LAYER 2 — Tenant customization (per-tenant config)
  - Company name, interviewer persona name
  - Brand voice guidelines
  - Prohibited topics

LAYER 3 — Role customization (per-job-role)
  - Job description summary
  - Required competencies to assess
  - Technical domain context
  - Question bank + fallback questions

LAYER 4 — Session context (per-session, injected at start)
  - Candidate name
  - Submitted resume summary (optional)
  - Current session phase
  - Questions already asked (deduplication)
```

### 7.4 STAR Scoring Rubric (LLM-evaluated)

```
Per answer, GPT-4o evaluates against:
  Situation (0-2.5): Did they set clear context?
  Task      (0-2.5): Did they identify their specific responsibility?
  Action    (0-2.5): Did they describe concrete steps THEY took?
  Result    (0-2.5): Did they quantify the outcome?
  
  STAR Total: 0–10

Additional overlays:
  Specificity Score (0–5): Vague generalizations vs. concrete examples
  Relevance Score  (0–5): Answer relevance to the question asked
  Depth Score      (0–5): Surface-level vs. insightful response
```

---

## 8. Behavioral Analysis Engine (14 Dimensions)

### Real-Time (During Interview)
Processed continuously via Azure Speech SDK + WebSocket events:

| # | Dimension | Measurement | Thresholds (Red/Yellow/Green) |
|:---|:---|:---|:---|
| 1 | **Words Per Minute (WPM)** | Word count / speaking time | <80 or >200 / 80-110 or 180-200 / 110-180 |
| 2 | **Filler Word Rate** | (fillers / total words) × 60 | >8/min / 5-8/min / <5/min |
| 3 | **Pause Duration** | Silence gaps > 1.5s, avg duration | >4s avg / 2-4s avg / <2s avg |
| 4 | **Vocal Pitch Stability** | Pitch variance (Hz) across response | High variance / Med / Low (stable) |
| 5 | **Speech Clarity Score** | Azure Pronunciation Assessment fluency | <60 / 60-80 / >80 |
| 6 | **Response Latency** | Time-to-first-word after Q ends (ms) | >5000ms / 2000-5000 / <2000 |
| 7 | **Tab Focus Loss** | Visibility API hidden events | >3 events / 1-3 / 0 |

### Post-Processing (After Interview)
Runs async via workers using stored transcript + frames:

| # | Dimension | Extraction Method | Tool |
|:---|:---|:---|:---|
| 8 | **Eye Contact Ratio** | Frame analysis: face landmark gaze vectors | Azure AI Vision + GPT-4o Vision |
| 9 | **Posture & Engagement** | Head tilt, forward/backward lean, fidgeting | GPT-4o Vision (key frames at Q transitions) |
| 10 | **Vocabulary Richness (TTR)** | Type-Token Ratio on transcript | Local NLP (ML.NET / custom) |
| 11 | **STAR Adherence** | GPT-4o rubric scoring per answer | Azure AI Foundry evaluation |
| 12 | **Response Consistency** | Embedding similarity: cross-Q contradictions | text-embedding-3-large cosine similarity |
| 13 | **Authenticity Score** | Detect scripted/memorized responses (low perplexity, robotic cadence) | GPT-4o + perplexity heuristic |
| 14 | **Emotional Arc** | Sentiment trajectory across interview timeline | GPT-4o segment scoring |

### Signal Aggregation Pipeline

```
Raw Events (Service Bus: telemetry-raw)
    │
    ▼
TelemetryAggregatorWorker
    ├── Groups by sessionId + questionId
    ├── Computes rolling averages (WPM, filler rate, pause stats)
    ├── Stores aggregated metrics → CosmosDB (telemetry container)
    └── Publishes AggregationComplete event → post-interview topic

PostInterviewProcessingWorker (triggers after session complete)
    ├── Fetches full transcript from Blob
    ├── Sends transcript to GPT-4o → STAR scoring, sentiment, consistency
    ├── Triggers VisionWorker (frame batch to Azure AI Vision)
    ├── Computes TTR, vocabulary score
    └── Publishes AllSignalsReady → scoring topic

ScoringWorker
    ├── Loads rubric for role (from Redis cache or CosmosDB)
    ├── Applies weighted formula to all 14 dimensions + competencies
    ├── Calls GPT-4o → generate executive summary + key strengths/flags
    ├── Performs bias audit (flags if demographics-correlated signals used)
    ├── Writes CandidateScore to CosmosDB
    └── Publishes ScoringComplete → notifications topic

ReportWorker
    ├── Generates PDF report (QuestPDF)
    ├── Uploads to Blob Storage
    └── Publishes ReportReady → notifications topic

NotificationWorker
    ├── Sends email to recruiter (SendGrid)
    ├── Sends in-app notification (SignalR push)
    └── Fires webhook to ATS (if configured)
```

---

## 9. Real-Time AV Pipeline

### 9.1 Candidate Connection Sequence

```
1. Candidate clicks invite link → Next.js loads
2. Browser calls GET /sessions/{id}/join-token
   → API validates token, returns:
     { acsToken, acsCallId, wsEndpoint, sessionConfig }

3. Browser initialises ACS Calling SDK
   → getUserMedia({ video: true, audio: true })
   → Joins ACS call → ACS activates recording (auto to Blob)

4. Browser opens WebSocket to GPT-4o Realtime endpoint
   → Sends session.update: { voice, instructions, turn_detection }
   → Starts streaming microphone PCM16 audio

5. GPT-4o Realtime:
   → Listens (VAD handles candidate → agent turn switching)
   → Generates agent speech audio chunks
   → Sends audio_delta events back to browser

6. Browser receives audio_delta → AudioContext.decodeAudio → playback
   SIMULTANEOUSLY: Browser sends audio chunks to Azure TTS Avatar API
   → Avatar generates lip-synced video stream
   → Plays in <video> element (candidate sees interviewer)

7. Periodic from browser every 5s during active speech:
   → Canvas.captureStream() frame → resize to 720p → POST /telemetry/frame
   → Azure AI Vision analysis triggered (async)
```

### 9.2 Avatar Integration Sequence

```csharp
// AvatarCoordinator.cs — coordinates TTS + Avatar API

public async Task<string> SynthesizeAvatarAsync(
    string text,
    string sessionId,
    CancellationToken ct)
{
    // Build SSML with natural pauses, emphasis
    var ssml = BuildInterviewerSsml(text);
    
    // Call Azure Neural TTS Avatar REST API
    var avatarRequest = new AvatarSynthesisRequest
    {
        TalkingAvatarCharacter = _config.AvatarCharacter,  // "lisa"
        TalkingAvatarStyle = _config.AvatarStyle,           // "business"
        VideoFormat = "mp4",
        VideoCodec = "h264",
        SubtitleType = "soft_embedded",
        BackgroundColor = "#1a1a2e"
    };
    
    // Stream resulting video URL back to browser via SignalR
    await _signalR.SendToSessionAsync(sessionId, "avatar:play", videoUrl, ct);
}
```

### 9.3 Graceful Degradation
If avatar service is unavailable:
1. Fall back to audio-only mode (TTS plays without video)
2. Static avatar image displayed
3. Interview continues uninterrupted
4. Incident logged + alert fired

---

## 10. Security, Auth & Compliance

### 10.1 Authentication Flows

```
RECRUITER / ADMIN LOGIN:
  Browser → MSAL.js → Entra ID (PKCE + OIDC)
  → ID token + access token returned
  → API validates JWT (Entra ID JWKS endpoint)
  → RBAC roles checked per endpoint

CANDIDATE SESSION JOIN:
  Candidate receives time-limited signed URL (HMAC-SHA256)
  URL contains: sessionId + candidateId + expiry + signature
  → No Entra ID login required (public link)
  → API validates signature + checks session status
  → Returns short-lived ACS token (30 min TTL)
  → WebSocket authenticated via session token (header: X-Session-Token)

SERVICE-TO-SERVICE:
  All Container Apps use Managed Identity
  → No API keys in environment variables
  → Key Vault references in Container App config
  → MI has Reader on Key Vault + access policies per service
```

### 10.2 RBAC Matrix

| Action | Platform.Admin | Tenant.Admin | Recruiter | HiringManager | Candidate |
|:---|:---:|:---:|:---:|:---:|:---:|
| Create/delete tenants | ✅ | ❌ | ❌ | ❌ | ❌ |
| Manage tenant config | ✅ | ✅ | ❌ | ❌ | ❌ |
| Create job roles + rubrics | ✅ | ✅ | ✅ | ❌ | ❌ |
| Create interview sessions | ✅ | ✅ | ✅ | ❌ | ❌ |
| View all sessions in tenant | ✅ | ✅ | ✅ | ❌ | ❌ |
| View report + recording | ✅ | ✅ | ✅ | ✅ (assigned) | ❌ |
| Download PDF | ✅ | ✅ | ✅ | ✅ | ❌ |
| Conduct interview | ❌ | ❌ | ❌ | ❌ | ✅ (own) |
| Delete session data | ✅ | ✅ | ❌ | ❌ | ❌ |

### 10.3 GDPR & Data Privacy Compliance

```
Data Classification:
  - PII: candidate name, email, recording, transcript (encrypted at rest + in transit)
  - Sensitive: behavioral scores, competency scores
  - Operational: session metadata, telemetry events

Consent Framework:
  - Candidate shown clear consent screen BEFORE camera/mic access
  - Consent record stored: { candidateId, timestamp, consentVersion, ipHash }
  - Consent required for: recording, AI analysis, data retention

Data Retention Policy (configurable per tenant):
  - Recordings: 2 years (compliance), deletable on candidate request
  - Transcripts: 2 years
  - Raw telemetry: 90 days (TTL in Cosmos)
  - Aggregated scores: 5 years (business records)
  - System logs: 30 days

Right to Erasure (GDPR Article 17):
  POST /candidates/{id}/erasure-request
  → Triggers ErasureWorker:
    - Deletes/anonymizes all Cosmos records for candidateId
    - Deletes recordings + transcripts from Blob
    - Purges Redis keys
    - Emits erasure-complete webhook
    - Logs erasure event (audit trail maintained without PII)

Encryption:
  - Data at rest: Azure-managed keys (CMK available for Enterprise tier)
  - Data in transit: TLS 1.3 enforced (APIM + Front Door)
  - API keys: Key Vault only, never in config files
  - Recordings: Client-side encryption before upload (optional, Enterprise)
```

### 10.4 Bias Audit & Fairness

```
BiasAuditRecord stored with every score:
  - blindModeEnabled: bool (if enabled, visual signals excluded from score)
  - signalsUsed: list of dimensions included in scoring
  - signalsExcluded: list and reason
  - demographicDataPresent: false (platform never collects demographics)
  - rubricVersion: string (rubric used, for reproducibility audit)
  - aiModelVersion: string (GPT-4o version)
  
Configurable blind mode:
  - Tenant can enable "Blind Interview" mode
  - Vision analysis (eye contact, posture) excluded from score
  - Only verbal/content-based dimensions scored
  - Prevents appearance-based bias
```

---

## 11. Anti-Cheat & Integrity System

### Signals Monitored (Real-Time)

| Signal | Detection Method | Action |
|:---|:---|:---|
| **Tab/Window switch** | Visibility API + `blur` event | Log event, warn candidate |
| **Multiple persons in frame** | Azure AI Vision face count | Log event + flag session |
| **Screen sharing detected** | `getDisplayMedia` API presence | Log event |
| **Audio playback in background** | Audio analysis (multiple voices) | Flag for review |
| **AI-generated response** | Low perplexity + robotic cadence heuristic | Authenticity score reduction |
| **Response pre-reading** | Eye movement pattern (unusual rapid scanning) | Log + authenticity flag |
| **Scripted memorized response** | Embedding similarity vs. known scripted answers | Authenticity score reduction |
| **Impersonation (voice)** | Speaker verification vs. initial enrollment | Flag if mismatch > threshold |

### Anti-Cheat Scoring

```csharp
public record AntiCheatLog
{
    public int TabSwitchCount { get; set; }
    public List<DateTimeOffset> TabSwitchTimestamps { get; set; } = [];
    public double TotalFocusLostSeconds { get; set; }
    public int MultiplePersonsAlertCount { get; set; }
    public bool SuspectedScriptedResponses { get; set; }
    public List<string> Flags { get; set; } = [];    // human-readable flag descriptions
    public IntegrityLevel IntegrityLevel { get; set; }  // Clean|Suspicious|Compromised
}

// IntegrityLevel determines:
//   Clean       → Normal processing
//   Suspicious  → Recruiter notified, score includes integrity caveat
//   Compromised → Session flagged, manual review required before decision
```

---

## 12. Observability & Monitoring

### 12.1 Metrics (Custom + Azure Platform)

```
SERVICE-LEVEL METRICS (Application Insights custom metrics):
  - session.created.count             (rate)
  - session.completed.count           (rate)
  - session.abandonRate               (gauge, %)
  - session.avgDurationSeconds        (histogram)
  - agent.responseLatencyMs           (histogram, p50/p95/p99)
  - agent.tokenUsage                  (counter, by model)
  - scoring.processingTimeMs          (histogram)
  - vision.frameProcessingTimeMs      (histogram)
  - report.generationTimeMs           (histogram)

INFRASTRUCTURE METRICS (Azure Monitor):
  - Container App: CPU%, Memory%, replica count, request latency
  - Cosmos DB: RU/s consumed, throttled requests, p99 latency
  - Service Bus: active messages, dead-letter count, processing rate
  - Speech API: concurrent connections, error rate
  - GPT-4o: TPM usage, rate limit events
```

### 12.2 Alerting Policy

```
P1 — CRITICAL (PagerDuty + SMS + Slack):
  - Interview API error rate > 5% (5min window)
  - GPT-4o Realtime connection failure rate > 10%
  - Cosmos DB throttling > 100 req/min
  - Service Bus dead-letter count > 50

P2 — HIGH (Email + Slack):
  - Session completion rate drops > 10% vs. 7-day baseline
  - Scoring worker processing latency p95 > 5min
  - Redis cache miss rate > 30%
  - Container App replica count at maximum

P3 — WARNING (Dashboard + Slack):
  - Agent token usage > 80% of quota
  - Storage approaching capacity thresholds
  - Any P1 alert resolved (close notification)
```

### 12.3 Distributed Tracing

```
All services emit OpenTelemetry traces:
  - Trace parent propagated via W3C TraceContext headers
  - Key trace spans:
    - Session.Start → Thread.Create → Avatar.Greet
    - Question.Deliver → Candidate.Response → Agent.Evaluate → Question.Next
    - PostProcess.Start → STAR.Score → Vision.Analyze → Score.Compute → Report.Generate
  - Traces visible in Application Insights Transaction Search
  - Correlated with logs via trace ID
```

---

## 13. Infrastructure as Code (Bicep)

```bicep
// main.bicep — top-level orchestration

targetScope = 'subscription'

param environment string  // dev | staging | prod
param location string = 'eastus2'
param tenantId string

var prefix = 'interview-${environment}'

module cosmosDb 'modules/cosmos.bicep' = {
  name: '${prefix}-cosmos'
  params: {
    accountName: '${prefix}-cosmos'
    location: location
    containers: [
      { name: 'sessions',    partitionKey: '/tenantId', ttl: -1 }
      { name: 'candidates',  partitionKey: '/tenantId', ttl: -1 }
      { name: 'telemetry',   partitionKey: '/sessionId', ttl: 7776000 }
      { name: 'scores',      partitionKey: '/tenantId', ttl: -1 }
      { name: 'transcripts', partitionKey: '/sessionId', ttl: -1 }
      { name: 'rubrics',     partitionKey: '/tenantId', ttl: -1 }
      { name: 'tenants',     partitionKey: '/id',        ttl: -1 }
    ]
  }
}

module serviceBus 'modules/servicebus.bicep' = {
  name: '${prefix}-bus'
  params: {
    namespaceName: '${prefix}-bus'
    sku: environment == 'prod' ? 'Premium' : 'Standard'
    topics: ['interview-events', 'telemetry-raw', 'post-interview', 'notifications']
  }
}

module containerAppsEnv 'modules/containerapps-env.bicep' = {
  name: '${prefix}-aca-env'
  params: {
    environmentName: '${prefix}-env'
    logAnalyticsWorkspaceId: monitoring.outputs.workspaceId
  }
}

module interviewApi 'modules/container-app.bicep' = {
  name: '${prefix}-interview-api'
  params: {
    appName: 'interview-api'
    image: 'acrregistry.azurecr.io/interview-api:${imageTag}'
    minReplicas: environment == 'prod' ? 2 : 1
    maxReplicas: environment == 'prod' ? 20 : 5
    env: {
      ASPNETCORE_ENVIRONMENT: environment
      KeyVault__Uri: keyVault.outputs.vaultUri
    }
  }
  dependsOn: [containerAppsEnv]
}

// ... additional modules for each service
```

---

## 14. CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/deploy.yml
name: Build & Deploy Interview Platform

on:
  push:
    branches: [main, staging]
  pull_request:
    branches: [main]

env:
  REGISTRY: interviewplatform.azurecr.io

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup .NET 9
        uses: actions/setup-dotnet@v4
        with: { dotnet-version: '9.0.x' }
      
      - name: Restore + Build
        run: dotnet build ./backend/InterviewAgent.sln --configuration Release
      
      - name: Unit Tests
        run: dotnet test ./backend/InterviewAgent.sln
               --filter "Category=Unit"
               --collect "Code Coverage"
               --results-directory ./test-results
      
      - name: Integration Tests
        run: dotnet test ./backend/InterviewAgent.sln
               --filter "Category=Integration"
               --environment "ConnectionStrings__CosmosDB=${{ secrets.TEST_COSMOS_CONNECTION }}"
      
      - name: Setup Node.js 20
        uses: actions/setup-node@v4
        with: { node-version: '20.x' }
      
      - name: Build Next.js apps
        run: |
          cd frontend && npm ci
          npm run build --workspace=candidate-app
          npm run build --workspace=recruiter-dashboard
          npm run lint

  security-scan:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
      - name: Trivy vulnerability scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          security-checks: 'vuln,secret'
          severity: 'HIGH,CRITICAL'
          exit-code: '1'
      
      - name: OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@main

  docker-push:
    needs: [build-and-test, security-scan]
    if: github.ref == 'refs/heads/main' || github.ref == 'refs/heads/staging'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: [interview-api, telemetry-api, admin-api, 
                  telemetry-worker, scoring-worker, report-worker,
                  post-interview-worker, notification-worker]
    steps:
      - name: Login to ACR
        uses: azure/docker-login@v1
        with:
          login-server: ${{ env.REGISTRY }}
          username: ${{ secrets.ACR_USERNAME }}
          password: ${{ secrets.ACR_PASSWORD }}
      
      - name: Build and push ${{ matrix.service }}
        run: |
          docker build -t ${{ env.REGISTRY }}/${{ matrix.service }}:${{ github.sha }} \
            -f ./backend/${{ matrix.service }}/Dockerfile ./backend
          docker push ${{ env.REGISTRY }}/${{ matrix.service }}:${{ github.sha }}

  deploy-staging:
    needs: docker-push
    if: github.ref == 'refs/heads/staging'
    environment: staging
    runs-on: ubuntu-latest
    steps:
      - name: Azure Login
        uses: azure/login@v2
        with: { creds: '${{ secrets.AZURE_CREDENTIALS_STAGING }}' }
      
      - name: Deploy Bicep Infrastructure
        run: |
          az deployment sub create \
            --location eastus2 \
            --template-file ./infra/main.bicep \
            --parameters environment=staging imageTag=${{ github.sha }}
      
      - name: Deploy Container Apps
        run: |
          for service in interview-api telemetry-api admin-api; do
            az containerapp update --name $service \
              --resource-group interview-staging-rg \
              --image ${{ env.REGISTRY }}/$service:${{ github.sha }}
          done
      
      - name: Run Smoke Tests
        run: dotnet test ./backend/InterviewAgent.sln --filter "Category=Smoke"
           --environment "ApiBaseUrl=https://api.staging.yourdomain.com"

  deploy-production:
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    environment: production      # Requires manual approval in GitHub Environments
    runs-on: ubuntu-latest
    steps:
      - name: Blue-Green Deploy
        run: |
          # Deploy to inactive slot (green)
          # Run health checks on green
          # Swap traffic (0% → 10% → 50% → 100% over 15 min)
          # Automated rollback if error rate spikes
```

---

## 15. Resilience & Fault Tolerance

### 15.1 Patterns Applied

```
RETRY with exponential backoff:
  - Azure AI Foundry API calls: 3 retries, 1s/2s/4s delays
  - Speech API: 2 retries, 500ms/1s delays
  - Cosmos DB writes: 3 retries (on 429 throttle)
  - Service Bus publish: 3 retries

CIRCUIT BREAKER (Polly):
  - GPT-4o Realtime: opens after 5 failures in 30s, half-open after 60s
  - Speech API: opens after 3 failures in 10s

FALLBACK CHAINS:
  - Avatar unavailable → Audio-only mode (TTS without video)
  - GPT-4o Realtime unavailable → Text-based chat mode (degrade gracefully)
  - Speech STT unavailable → Browser Web Speech API fallback
  - Vision API unavailable → Skip visual analysis, flag report as partial

TIMEOUT POLICIES:
  - Agent response: 8s timeout (then graceful: "Take your time...")
  - Vision frame analysis: 30s timeout
  - PDF generation: 60s timeout
  - Session heartbeat: if missed 3× (45s) → session marked abandoned

SESSION RECOVERY:
  - Browser reconnects within 2 min → session resumes from last question
  - Redis stores session state (last question, phase, score so far)
  - Agent thread preserved (Foundry Agent thread ID in session record)

DEAD LETTER QUEUE PROCESSING:
  - Dedicated DLQ inspector worker (runs every 5min)
  - Alerts on DLQ count > 10
  - Admin endpoint to re-queue failed messages
```

---

## 16. Cost Model & Optimization

### Estimated Monthly Cost (100 interviews/day × 30 days = 3,000 sessions/month)

| Service | Unit | Monthly Cost (est.) |
|:---|:---|:---:|
| Azure AI Foundry / GPT-4o | 3,000 sessions × ~30K tokens = 90M tokens | ~$360 |
| GPT-4o Realtime Audio | 3,000 sessions × ~15min avg = 45,000 min | ~$900 |
| Azure AI Speech (STT+TTS+Avatar) | 3,000 sessions × 15min | ~$450 |
| Azure AI Vision | 3,000 × 36 frames (5s interval × 180s) = 108K calls | ~$108 |
| Azure Cosmos DB | Autoscale, 3,000 sessions × ~500KB = 1.5GB | ~$80 |
| Azure Blob Storage | 3,000 × ~200MB recordings = 600GB | ~$12 |
| Azure Container Apps | 8 services × average sizing | ~$200 |
| Azure Service Bus (Premium) | 1 namespace | ~$670 |
| Azure Communication Services | 3,000 recordings × 15min | ~$135 |
| Azure Cache for Redis | C2 Standard | ~$100 |
| Azure APIM | Standard v2 | ~$150 |
| Azure Front Door | ~$50 | ~$50 |
| **TOTAL** | | **~$3,215/month** |

### Cost Optimization Strategies
- **Avatar video caching:** Cache common phrases (greetings, transitions) in Blob Storage (~30% avatar cost reduction)
- **Recording compression:** H.264 720p instead of 1080p (~50% storage cost)
- **Telemetry batching:** Browser sends events in batches of 10 (reduce API calls 10×)
- **Frame sampling rate:** Reduce from 5s to 10s intervals during non-active speech (~50% Vision cost)
- **GPT-4o mini for STAR scoring:** Use mini model for per-answer scoring, GPT-4o only for executive summary (~40% LLM cost)
- **Cosmos DB TTL:** Auto-delete raw telemetry after 90 days

---

## 17. Testing Strategy

### Test Pyramid

```
                    ╱╲
                   ╱  ╲  E2E TESTS (Playwright)
                  ╱    ╲  - Full interview flow (mocked AI)
                 ╱──────╲ - Recruiter dashboard
                ╱        ╲
               ╱          ╲ INTEGRATION TESTS
              ╱            ╲ - API endpoints (TestContainers)
             ╱              ╲ - Worker pipeline (in-memory Service Bus)
            ╱────────────────╲ - Cosmos DB queries
           ╱                  ╲
          ╱                    ╲ UNIT TESTS
         ╱                      ╲ - Scoring engine logic
        ╱                        ╲ - Behavioral metric calculators
       ╱                          ╲ - STAR rubric evaluation
      ╱────────────────────────────╲ - State machine transitions
```

### Test Categories

```csharp
// Unit test example: Scoring Engine
[Fact]
[Trait("Category", "Unit")]
public void ComputeOverallScore_WithFullMetrics_ReturnsWeightedAverage()
{
    var rubric = TestRubrics.SeniorEngineerRubric();
    var signals = TestSignals.HighPerformerSignals();
    var engine = new ScoringEngine(rubric);
    
    var result = engine.ComputeOverallScore(signals);
    
    Assert.InRange(result.OverallScore, 80, 100);
    Assert.Equal(ScoreGrade.A, result.Grade);
}

// Integration test example: Session API
[Fact]
[Trait("Category", "Integration")]
public async Task CreateSession_WithValidRequest_ReturnsSessionWithJoinUrl()
{
    var response = await _client.PostAsJsonAsync("/api/v1/sessions", new
    {
        jobRoleId = "senior-engineer-role",
        candidateId = "test-candidate-001",
        candidateName = "Test User"
    });
    
    response.EnsureSuccessStatusCode();
    var session = await response.Content.ReadFromJsonAsync<SessionResponse>();
    
    Assert.NotNull(session.JoinUrl);
    Assert.Equal(SessionStatus.WaitingCandidate, session.Status);
}
```

### Load Testing (Azure Load Testing)

```yaml
# load-test.yaml — Simulate 100 concurrent interview sessions
scenarios:
  - name: ConcurrentInterviews
    requests:
      - name: CreateSession
        url: /api/v1/sessions
        method: POST
      - name: Heartbeat (every 15s)
        url: /api/v1/sessions/{sessionId}/heartbeat
        method: POST
      - name: BatchTelemetry (every 5s)
        url: /api/v1/telemetry/events
        method: POST
        body: "{{ generateTelemetryBatch() }}"

acceptance:
  - metric: response_time_ms_p95
    condition: '< 500'
  - metric: error_rate
    condition: '< 0.1%'
  - metric: concurrent_users
    target: 100
```

---

## 18. Complete Project Structure

```
InterviewPlatform/
├── infra/                                     # Bicep IaC
│   ├── main.bicep
│   └── modules/
│       ├── cosmos.bicep
│       ├── servicebus.bicep
│       ├── containerapps-env.bicep
│       ├── container-app.bicep
│       ├── apim.bicep
│       ├── keyvault.bicep
│       └── monitoring.bicep
│
├── frontend/                                  # Next.js 14 Monorepo (pnpm workspaces)
│   ├── package.json                           # Workspace root
│   ├── apps/
│   │   ├── candidate/                         # Interview room
│   │   │   ├── app/
│   │   │   │   ├── layout.tsx
│   │   │   │   ├── [sessionId]/
│   │   │   │   │   ├── page.tsx               # Main interview room
│   │   │   │   │   ├── consent/page.tsx       # GDPR consent screen
│   │   │   │   │   └── complete/page.tsx      # Interview complete screen
│   │   │   └── components/
│   │   │       ├── InterviewRoom.tsx          # Main container component
│   │   │       ├── AvatarPlayer.tsx           # AI avatar video player
│   │   │       ├── CameraPreview.tsx          # Candidate's own camera view
│   │   │       ├── AudioVisualizer.tsx        # Real-time audio waveform
│   │   │       ├── ProgressTracker.tsx        # Phase/question progress
│   │   │       ├── ConsentModal.tsx
│   │   │       └── hooks/
│   │   │           ├── useRealtimeSession.ts  # GPT-4o Realtime WebSocket
│   │   │           ├── useACSCall.ts          # Azure Communication Services
│   │   │           ├── useAvatarStream.ts     # Avatar video management
│   │   │           ├── useTelemetryEmitter.ts # Batched event sending
│   │   │           ├── useFocusTracker.ts     # Anti-cheat tab monitoring
│   │   │           └── useSessionHeartbeat.ts
│   │   │
│   │   ├── recruiter/                         # Recruiter dashboard
│   │   │   ├── app/
│   │   │   │   ├── dashboard/page.tsx         # Overview + recent sessions
│   │   │   │   ├── sessions/
│   │   │   │   │   ├── page.tsx               # Session list (filterable)
│   │   │   │   │   └── [id]/
│   │   │   │   │       ├── page.tsx           # Session detail + replay
│   │   │   │   │       └── report/page.tsx    # Full behavioral report
│   │   │   │   ├── candidates/
│   │   │   │   │   ├── page.tsx               # Candidate list
│   │   │   │   │   └── compare/page.tsx       # Multi-candidate comparison
│   │   │   │   └── roles/
│   │   │   │       ├── page.tsx               # Job role management
│   │   │   │       └── [id]/rubric/page.tsx   # Rubric configuration
│   │   │   └── components/
│   │   │       ├── VideoReplayPlayer.tsx      # Player with behavioral overlay
│   │   │       ├── BehavioralTimeline.tsx     # Color-coded timeline bar
│   │   │       ├── CompetencyRadar.tsx        # Radar chart (Recharts)
│   │   │       ├── ScoreCard.tsx              # Candidate score summary
│   │   │       ├── TranscriptViewer.tsx       # Searchable transcript
│   │   │       ├── AntiCheatReport.tsx        # Integrity flags panel
│   │   │       └── CandidateComparisonGrid.tsx
│   │   │
│   │   └── admin/                             # Platform admin
│   │       ├── app/
│   │       │   ├── tenants/page.tsx
│   │       │   ├── monitoring/page.tsx        # Live session monitoring
│   │       │   └── usage/page.tsx             # Billing + quota tracking
│   │       └── components/
│   │           ├── LiveSessionMonitor.tsx     # Real-time active sessions map
│   │           └── UsageMetrics.tsx
│   │
│   └── packages/
│       ├── ui/                                # Shared component library
│       │   ├── Button, Card, Modal, Badge, etc.
│       │   └── design-tokens.ts              # Colors, typography, spacing
│       ├── api-client/                        # Type-safe API client (generated from OpenAPI)
│       └── types/                             # Shared TypeScript types
│
├── backend/                                   # .NET 9 Solution
│   ├── InterviewAgent.sln
│   │
│   ├── InterviewAgent.Api/                    # ASP.NET Core Web API
│   │   ├── Controllers/
│   │   │   ├── SessionsController.cs
│   │   │   ├── TelemetryController.cs
│   │   │   ├── ScoresController.cs
│   │   │   ├── ReportsController.cs
│   │   │   ├── RubricsController.cs
│   │   │   ├── CandidatesController.cs
│   │   │   └── AnalyticsController.cs
│   │   ├── Middleware/
│   │   │   ├── TenantContextMiddleware.cs
│   │   │   ├── RequestLoggingMiddleware.cs
│   │   │   └── ExceptionHandlingMiddleware.cs
│   │   ├── SignalR/
│   │   │   └── SessionHub.cs                 # Real-time status push
│   │   └── Program.cs
│   │
│   ├── InterviewAgent.TelemetryApi/           # Separate high-throughput API
│   │   └── Controllers/
│   │       ├── EventsController.cs            # Batch event ingestion
│   │       ├── FramesController.cs            # Vision frame upload
│   │       └── AudioMetricsController.cs
│   │
│   ├── InterviewAgent.Core/                   # Domain logic (no Azure dependencies)
│   │   ├── Models/
│   │   │   ├── InterviewSession.cs
│   │   │   ├── CandidateScore.cs
│   │   │   ├── BehavioralTelemetry.cs
│   │   │   ├── CompetencyRubric.cs
│   │   │   ├── JobRole.cs
│   │   │   ├── Tenant.cs
│   │   │   └── AntiCheatLog.cs
│   │   ├── Services/
│   │   │   └── ScoringEngine.cs              # Pure scoring logic, testable
│   │   ├── StateMachine/
│   │   │   └── InterviewStateMachine.cs      # Phase transition logic
│   │   └── Interfaces/
│   │       ├── IAgentService.cs
│   │       ├── ISpeechAnalysisService.cs
│   │       ├── IVisionAnalysisService.cs
│   │       ├── ISessionRepository.cs
│   │       ├── IScoreRepository.cs
│   │       ├── ITelemetryRepository.cs
│   │       ├── IBlobStorageService.cs
│   │       ├── IReportGeneratorService.cs
│   │       └── INotificationService.cs
│   │
│   ├── InterviewAgent.Infrastructure/         # Azure service implementations
│   │   ├── AzureFoundry/
│   │   │   ├── FoundryAgentService.cs
│   │   │   ├── FoundryEvaluationService.cs
│   │   │   └── SystemPromptBuilder.cs
│   │   ├── AzureSpeech/
│   │   │   ├── SpeechAnalysisService.cs
│   │   │   ├── AvatarCoordinator.cs
│   │   │   └── PronunciationAssessmentService.cs
│   │   ├── AzureVision/
│   │   │   └── VisionAnalysisService.cs
│   │   ├── AzureCommunication/
│   │   │   └── AcsSessionService.cs
│   │   ├── CosmosDb/
│   │   │   ├── CosmosSessionRepository.cs
│   │   │   ├── CosmosScoreRepository.cs
│   │   │   └── CosmosTelemetryRepository.cs
│   │   ├── Storage/
│   │   │   └── AzureBlobStorageService.cs
│   │   ├── Messaging/
│   │   │   └── ServiceBusPublisher.cs
│   │   ├── Cache/
│   │   │   └── RedisCacheService.cs
│   │   └── Reports/
│   │       └── QuestPdfReportGenerator.cs
│   │
│   ├── InterviewAgent.Workers/                # Background workers
│   │   ├── TelemetryAggregatorWorker.cs
│   │   ├── PostInterviewProcessingWorker.cs
│   │   ├── VisionAnalysisWorker.cs
│   │   ├── ScoringWorker.cs
│   │   ├── ReportWorker.cs
│   │   ├── NotificationWorker.cs
│   │   ├── DeadLetterInspectorWorker.cs
│   │   └── ErasureWorker.cs                  # GDPR right-to-erasure
│   │
│   └── InterviewAgent.Tests/
│       ├── Unit/
│       │   ├── ScoringEngineTests.cs
│       │   ├── StateMachineTests.cs
│       │   └── BehavioralMetricTests.cs
│       ├── Integration/
│       │   ├── SessionApiTests.cs
│       │   ├── TelemetryApiTests.cs
│       │   └── WorkerPipelineTests.cs
│       └── E2E/
│           └── FullInterviewFlowTests.cs
│
├── .github/
│   └── workflows/
│       ├── deploy.yml
│       ├── pr-checks.yml
│       └── load-test.yml
│
└── docs/
    ├── architecture/                          # ADRs (Architecture Decision Records)
    ├── api/                                   # OpenAPI spec (auto-generated)
    ├── runbooks/                              # Incident response procedures
    └── gdpr/                                 # Data processing documentation
```

---

## 19. 16-Week Implementation Roadmap

### Phase 1 — Foundation & Core API (Weeks 1–4)
```
Week 1: Infrastructure setup
  ✓ Bicep modules for all Azure services (dev environment)
  ✓ GitHub Actions CI pipeline (build + unit test)
  ✓ .NET solution structure + Core domain models
  ✓ CosmosDB schema + Cosmos repository pattern

Week 2: Session API
  ✓ POST /sessions — create session, generate signed candidate URL
  ✓ GET /sessions/{id}/join-token — ACS token + WebSocket config
  ✓ Session heartbeat + status SSE endpoint
  ✓ Tenant context middleware + RBAC setup

Week 3: Agent Integration
  ✓ Azure AI Foundry Agent Service integration
  ✓ System prompt builder (4-layer architecture)
  ✓ Interview state machine implementation
  ✓ Question cycle: deliver → receive → evaluate → next

Week 4: Basic Candidate UI
  ✓ Next.js candidate app scaffold
  ✓ Camera + microphone capture (getUserMedia)
  ✓ WebSocket connection to GPT-4o Realtime
  ✓ Text-mode interview works end-to-end (no avatar yet)
```

### Phase 2 — Real-Time Voice & Avatar (Weeks 5–7)
```
Week 5: Voice Streaming
  ✓ GPT-4o Realtime audio stream integration (browser WebSocket)
  ✓ VAD (Voice Activity Detection) integration
  ✓ Azure ACS WebRTC call integration
  ✓ ACS recording → Blob Storage pipeline

Week 6: Avatar Integration
  ✓ Azure Neural TTS Avatar API integration
  ✓ SSML-enhanced speech with natural pauses
  ✓ Avatar video stream display in browser <video>
  ✓ Avatar ↔ Candidate turn-taking synchronisation

Week 7: Speech Metrics (Real-Time)
  ✓ Azure Speech SDK: real-time STT with timestamps
  ✓ WPM calculation + filler word detection
  ✓ Pause detection + response latency measurement
  ✓ Telemetry batch API + telemetry aggregator worker
```

### Phase 3 — Behavioral Analysis Pipeline (Weeks 8–11)
```
Week 8: Post-Interview Processing
  ✓ STAR scoring via GPT-4o (per-question, post-session)
  ✓ Vocabulary richness (TTR) calculation
  ✓ Response consistency (embedding similarity)
  ✓ Service Bus post-interview pipeline working

Week 9: Vision Analysis
  ✓ Browser frame capture (Canvas API every 5s)
  ✓ Frame upload API + Azure Blob batch job
  ✓ Azure AI Vision: face count + gaze estimation
  ✓ GPT-4o Vision: posture + engagement classification
  ✓ Vision worker processing + metric aggregation

Week 10: Scoring Engine
  ✓ Full 14-dimension scoring engine
  ✓ Configurable rubric system (per-tenant, per-role)
  ✓ Weighted composite score calculation
  ✓ Bias audit trail
  ✓ GPT-4o: executive summary + key strengths + red flags generation

Week 11: Anti-Cheat System
  ✓ Tab focus loss detection + browser event tracking
  ✓ Multiple person detection (Vision API)
  ✓ Speaker verification enrollment + mismatch detection
  ✓ Authenticity score (scripted response detection)
  ✓ IntegrityLevel classification logic
```

### Phase 4 — Recruiter Dashboard & Reports (Weeks 12–14)
```
Week 12: Core Dashboard
  ✓ Session list view (paginated, searchable, filterable)
  ✓ Candidate scorecard page
  ✓ Competency radar charts (Recharts)
  ✓ Transcript viewer (searchable, word-confidence coloring)

Week 13: Video Replay + Analytics
  ✓ Video player with behavioral timeline overlay
  ✓ Question navigator (click → jump to timestamp)
  ✓ Anti-cheat report panel (integrity flags + events)
  ✓ Multi-candidate comparison view (radar + table)

Week 14: Reports & Notifications
  ✓ PDF report generation (QuestPDF) with full scorecard
  ✓ Email notifications (SendGrid) to recruiters
  ✓ SignalR real-time push (interview complete notification)
  ✓ ATS webhook integration (interview.completed event)
```

### Phase 5 — Production Hardening (Weeks 15–16)
```
Week 15: Security & Compliance
  ✓ Microsoft Entra ID integration (recruiter/admin login)
  ✓ GDPR consent flow (candidate) + erasure endpoint
  ✓ Key Vault migration (all secrets out of config)
  ✓ WAF rules + APIM rate limiting
  ✓ Penetration testing (manual + automated)

Week 16: Production Launch
  ✓ Staging environment full E2E test
  ✓ Load testing (100 concurrent sessions)
  ✓ Monitoring alerts configured + runbooks written
  ✓ GitHub Actions prod deploy with blue-green swap
  ✓ Disaster recovery drill
  ✓ Go-live! 🚀
```

---

> **This is your complete production-ready blueprint.** Every system, flow, data model, API contract, Azure service, and implementation detail is documented above. Start with Phase 1 and use `/plan` to scaffold the first code module.
