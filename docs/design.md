> **Methodology Note**
>
> This system architecture was designed using **Kiro**, following AWS Well-Architected Framework principles for AI/ML workloads, scalability, security, and operational excellence.

# System Architecture

## High-Level Architecture
> **Note**: Pilot-scale architecture demonstrating production-ready infrastructure for 1,000-user deployment. Includes optional performance optimizations (Redis caching) for scalability testing.

```
┌─────────────────────────────────────────────────────────────┐
│                     USER INTERFACE LAYER                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────────────┐  │
│  │  Voice   │  │   SMS    │  │  USSD Privacy Control    │  │
│  │  Calls   │  │ (Images) │  │   (*123*PRIVACY#)        │  │
│  │   🟡     │  │    🟢    │  │         🟡               │  │
│  └──────────┘  └──────────┘  └──────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   VOICE PROCESSING LAYER                     │
│  ┌──────────────────┐         ┌──────────────────────────┐ │
│  │ Amazon Transcribe│         │   Amazon Polly           │ │
│  │ Streaming STT    │◄───────►│   Neural TTS             │ │
│  │ Hindi            │         │   Aditi Voice            │ │
│  │      🟢         │         │        🟢                │ │
│  └──────────────────┘         └──────────────────────────┘ │
│           │                              ▲                   │
│           │  Amazon Connect (Simulated in Prototype /       │
│           │  Production Telephony) 🟡                       │
└─────────────────────────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              CONTEXTUAL REASONING ENGINE                     │
│  ┌──────────────────────────────────────────────────────┐  │
│  │            Amazon Bedrock                             │  │
│  │         Claude 3.5 Sonnet or Mistral 7B              │  │
│  │            Simple Prompt Chaining                     │  │
│  │                    🟢                                 │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   DATA & STORAGE LAYER                       │
│  ┌──────────────┐ ┌─────────────┐ ┌────────────────────┐  │
│  │  PostgreSQL  │ │ AWS Lambda  │ │  Amazon S3         │  │
│  │ User State + │ │ API Layer   │ │ Voice Recordings   │  │
│  │ pgvector for │ │ In-memory   │ │ SSE-KMS encrypted  │  │
│  │ embeddings   │ │ sessions    │ │                    │  │
│  │     🟢      │ │     🟢     │ │        🟢         │  │
│  └──────────────┘ └─────────────┘ └────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                  VOICE MEMORY NETWORK                        │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Success Stories Database (S3 + Metadata in PG)      │  │
│  │  Simple Semantic Matching (pgvector embeddings)      │  │
│  │                        🟢                             │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                  COMMUNICATION LAYER                         │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                Amazon SNS                             │  │
│  │              SMS Delivery                             │  │
│  │                  🟢                                   │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

🟢 Implemented in Prototype | 🟡 Simulated in Prototype | 🔵 Production-Only
```

## User Journey Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                USER JOURNEY FLOW                                           │
│                            (6 Stages - Discovery to Delivery)                              │
└─────────────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   1. DISCOVERY  │    │ 2. TRUST BUILD  │    │ 3. ELIGIBILITY  │    │ 4. DOCUMENT     │
│    (Day 0)      │───►│   (Day 1)       │───►│   DISCOVERY     │───►│   GUIDANCE      │
│      🟡         │    │   5 min call    │    │   (Day 1 cont.) │    │   (Day 1-3)     │
│                 │    │      🟢         │    │      🟢         │    │      🟢         │
└─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │                       │
         ▼                       ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Input:          │    │ Trigger:        │    │ AI asks:        │    │ SMS sent:       │
│ • Gov database  │    │ • Auto call     │    │ • Land size?    │    │ • Checklist     │
│ • NGO referral  │    │ • "Namaste,     │    │ • Crops grown?  │    │ • 📄 icons      │
│ • Manual list   │    │   main Sahaya"  │    │ • KCC status?   │    │ • Land records  │
│                 │    │ • Verification  │    │                 │    │ • Aadhaar       │
│ Action:         │    │   code shown    │    │ System matches: │    │ • Bank passbook │
│ • ID eligible   │    │                 │    │ • KCC scheme    │    │                 │
│   farmer        │    │ If hesitation:  │    │ • 92% eligible  │    │ User gathers    │
│                 │    │ • Play success  │    │                 │    │ at own pace     │
│ Output:         │    │   story (30s)   │    │ Output:         │    │                 │
│ • Phone # added │    │                 │    │ "You qualify    │    │ Output:         │
│   to queue      │    │ Output:         │    │  for ₹3 lakh    │    │ "Documents      │
│                 │    │ • Continue OR   │    │  credit. Need   │    │  ready"         │
│                 │    │ • Opt out       │    │  3 documents"   │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘

         │                                              │
         ▼                                              ▼
┌─────────────────┐    ┌─────────────────────────────────────────────────────────────────┐
│ 5. APPLICATION  │    │                    6. BENEFIT CONFIRMATION                      │
│   SUBMISSION    │───►│                         (Day 14)                                │
│   (Day 3-7)     │    │                           🟡                                   │
│      🟢         │    └─────────────────────────────────────────────────────────────────┘
└─────────────────┘                                   │
         │                                            ▼
         ▼                              ┌─────────────────────────────────────┐
┌─────────────────┐                    │ Final call:                         │
│ AI guides:      │                    │ • "Congratulations! KCC approved"  │
│ • Gov portal    │                    │ • "₹3 lakh credited to account"    │
│   steps         │                    │                                     │
│ • Form filling  │                    │ Request consent:                    │
│                 │                    │ • "Can I record your 30-second     │
│ Persistent      │                    │    success story?"                  │
│ follow-up if    │                    │                                     │
│ user stalls     │                    │ Output:                             │
│                 │                    │ • New story added to Voice Memory   │
│ Output:         │                    │   Network for future users         │
│ • Application   │                    │ • User becomes success advocate     │
│   reference #   │                    │                                     │
│   generated     │                    │                                     │
└─────────────────┘                    └─────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                    FLOW NOTES                                              │
│                                                                                             │
│ • Prototype demonstrates stages 2-6 in single demo session                                 │
│ • Production spans 14 days with real government processing times                           │
│ • Voice Memory Network creates flywheel effect: each success story helps 100+ future users│
│ • USSD privacy controls (*123*PRIVACY#) accessible at any stage                           │
│                                                                                             │
│ 🟢 Implemented in Prototype | 🟡 Simulated in Prototype | 🔵 Production-Only              │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

# Data Models

## User Profile
```sql
CREATE TABLE users (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    phone_number_hash VARCHAR(64) UNIQUE NOT NULL,  -- SHA-256 hashed
    language_preference VARCHAR(10) NOT NULL,        -- ISO 639-1 code
    created_at TIMESTAMP DEFAULT NOW(),
    last_interaction TIMESTAMP,
    opt_in_status BOOLEAN DEFAULT TRUE,
    data_retention_expires_at TIMESTAMP DEFAULT NOW() + INTERVAL '90 days'
);
```

## Conversation Session
```sql
CREATE TABLE conversation_sessions (
    session_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id),
    start_time TIMESTAMP DEFAULT NOW(),
    end_time TIMESTAMP,
    scheme_discussed VARCHAR(100),
    eligibility_status VARCHAR(50),  -- 'eligible', 'ineligible', 'pending'
    confidence_score FLOAT,           -- 0.0 to 1.0
    follow_up_scheduled TIMESTAMP,
    status VARCHAR(50)                -- 'active', 'completed', 'abandoned'
);
```

## Scheme Information
```sql
CREATE TABLE schemes (
    scheme_id UUID PRIMARY KEY,
    scheme_name VARCHAR(255) NOT NULL,
    description TEXT,
    eligibility_criteria JSONB,       -- Flexible eligibility rules
    required_documents TEXT[],
    benefit_amount NUMERIC(12,2),
    application_url VARCHAR(500),
    language VARCHAR(10),
    embedding VECTOR(1536)             -- pgvector for semantic search
);

-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Create index for vector similarity search
CREATE INDEX ON schemes USING ivfflat (embedding vector_cosine_ops);
```

## Success Stories
```sql
CREATE TABLE success_stories (
    story_id UUID PRIMARY KEY,
    audio_file_s3_key VARCHAR(255) NOT NULL,
    scheme_id UUID REFERENCES schemes(scheme_id),
    language VARCHAR(10),
    region VARCHAR(50),
    demographic_tags TEXT[],          -- ['farmer', 'widow', 'Karnataka']
    duration_seconds INT,
    consent_verified BOOLEAN DEFAULT FALSE,
    times_played INT DEFAULT 0,
    effectiveness_score FLOAT,        -- Calculated from user feedback
    created_at TIMESTAMP DEFAULT NOW(),
    embedding VECTOR(1536)            -- pgvector for semantic matching
);

-- Create index for vector similarity search
CREATE INDEX ON success_stories USING ivfflat (embedding vector_cosine_ops);
```

## Application Tracking
```sql
CREATE TABLE applications (
    application_id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(user_id),
    scheme_id UUID REFERENCES schemes(scheme_id),
    status VARCHAR(50),               -- 'initiated', 'documents_pending', 'submitted', 'approved'
    reference_number VARCHAR(100),
    submitted_at TIMESTAMP,
    approved_at TIMESTAMP,
    benefit_amount NUMERIC(12,2)
);
```

# API Specifications

## Voice Conversation API

### POST /api/v1/conversation/initiate
**Description:** Initiates a new conversation session

**Request:**
```json
{
  "phone_number": "+91XXXXXXXXXX",
  "language_preference": "kn",
  "call_type": "proactive_outreach"
}
```

**Response:**
```json
{
  "session_id": "uuid-here",
  "status": "initiated",
  "estimated_wait_seconds": 30
}
```

### POST /api/v1/conversation/message
**Description:** Processes user speech input and returns AI response

**Request:**
```json
{
  "session_id": "uuid-here",
  "audio_data": "base64-encoded-audio",
  "audio_format": "wav"
}
```

**Response:**
```json
{
  "session_id": "uuid-here",
  "transcribed_text": "Mujhe crop insurance chahiye",
  "ai_response_text": "Main aapki help kar sakti hoon...",
  "ai_response_audio": "base64-encoded-audio",
  "detected_intent": "scheme_inquiry",
  "suggested_schemes": [
    {
      "scheme_id": "uuid",
      "name": "Pradhan Mantri Fasal Bima Yojana",
      "eligibility_score": 0.92
    }
  ]
}
```

## Scheme Discovery API

### GET /api/v1/schemes/search
**Description:** Semantic search for relevant welfare schemes using pgvector

**Query Parameters:**
```
?query=farmer with 2 acres land
&language=hi
&limit=5
```

**Response:**
```json
{
  "schemes": [
    {
      "scheme_id": "uuid",
      "name": "Kisan Credit Card",
      "description": "Credit facility for farmers",
      "eligibility_score": 0.89,
      "required_documents": ["Land records", "Aadhaar", "Bank account"],
      "benefit_amount": 300000
    }
  ],
  "total_results": 12
}
```

## Voice Memory API

### POST /api/v1/stories/retrieve
**Description:** Retrieves semantically relevant success story

**Request:**
```json
{
  "user_context": {
    "scheme_discussed": "crop_insurance",
    "user_region": "Karnataka",
    "user_demographics": ["farmer", "2_acres"],
    "conversation_sentiment": "skeptical"
  },
  "language": "kn"
}
```

**Response:**
```json
{
  "story_id": "uuid",
  "audio_url": "s3://presigned-url",
  "transcript": "Namaskara, I'm Manjula from Tumakuru...",
  "duration_seconds": 28,
  "relevance_score": 0.94
}
```

# Deployment Architecture

## Pilot Deployment (AWS Cloud)
> **Note**: Pilot-scale architecture demonstrating production-ready infrastructure for 1,000-user deployment. Includes optional performance optimizations (Redis caching) for scalability testing.

```
┌─────────────────────────────────────────┐
│         AWS Cloud Infrastructure        │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │   FastAPI Backend (ECS Fargate)  │  │
│  │   - Conversation endpoints       │  │
│  │   - Bedrock API integration      │  │
│  │   - RDS connection               │  │
│  │   - Auto-scaling enabled         │  │
│  └──────────────────────────────────┘  │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │   Amazon RDS PostgreSQL          │  │
│  │   (db.t3.micro, Multi-AZ)       │  │
│  │   Automated backups enabled      │  │
│  │   - pgvector extension           │  │
│  └──────────────────────────────────┘  │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │   Web UI (S3 + CloudFront)       │  │
│  │   - Simulated phone interface    │  │
│  │   - Voice recording/playback     │  │
│  └──────────────────────────────────┘  │
└─────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────┐
│          AWS Cloud Services             │
│  - Amazon Bedrock (Claude 3.5/Mistral) │
│  - Amazon Transcribe (Speech-to-Text)   │
│  - Amazon Polly (Text-to-Speech)        │
│  - Amazon S3 (Voice file storage)       │
│  - AWS Lambda (Serverless functions)    │
│  - Amazon SNS (SMS delivery)            │
└─────────────────────────────────────────┘
```

## Optional Performance Layer
- **Amazon ElastiCache Redis** (cache.t3.micro): Session state caching for <100ms response times
- **Amazon CloudFront**: CDN for static assets and audio file delivery
- Enabled based on performance requirements during pilot phase

## Production Deployment (Future)
- Multi-region AWS deployment for latency optimization
- Amazon Connect for real telephony infrastructure
- Auto-scaling Lambda functions for serverless compute
- RDS PostgreSQL with pgvector extension and read replicas
- CloudFront CDN for static assets and audio files
- Optional: ElastiCache Redis for high-performance session caching

## Prototype Cost Estimate (16-Day Hackathon)

### AWS Service Usage
- Amazon Bedrock (1,000 test conversations)
- Amazon Connect (100 test calls, 5 min avg)
- Amazon Transcribe (500 minutes)
- Amazon Polly (100,000 characters)
- Amazon SNS (100 test SMS)
- AWS Lambda (10,000 invocations)
- Amazon RDS PostgreSQL (16-day deployment)
- Amazon S3 (5GB storage)
- Bhashini API (regional language processing)

**Total Prototype Cost**: ₹2,000-3,000

# Testing Strategy

## Unit Tests
- **Coverage Target**: 70% code coverage
- **Framework**: pytest for Python backend
- **Key Test Cases**:
  - Speech transcription accuracy (>90% on test audio clips)
  - Scheme matching relevance (precision@5 >80%)
  - Privacy controls (USSD code simulation)
  - Success story semantic retrieval

## Integration Tests
- **Bedrock API Integration**: Mock responses for deterministic testing
- **Database Operations**: Test CRUD operations on all tables
- **Voice Pipeline**: End-to-end test from audio input to AI response
- **SMS Delivery**: Mock SNS service for SMS testing

## Demo Scenarios
1. **Happy Path**: User qualifies for KCC, completes application successfully
2. **Trust Building**: Skeptical user convinced by success story
3. **Privacy Control**: User accesses privacy dashboard via USSD
4. **Multi-language**: Conversation starts in Hindi, user requests Kannada, AI switches seamlessly
5. **Error Recovery**: Poor audio quality, system gracefully degrades to DTMF

## Performance Tests
- **Load Testing**: 10 concurrent simulated conversations
- **Latency Benchmarks**: API response times under load
- **Memory Profiling**: Check for memory leaks in long-running processes
```

---

## Updated File Structure You Need
```
community-voice-bridge/
├── requirements.md          ← CREATE THIS (use template above)
├── design.md               ← EXPAND CURRENT FILE
├── README.md               ← Overview for GitHub
├── ARCHITECTURE.md         ← Optional: Detailed tech architecture
├── docs/
│   ├── api-specification.md
│   ├── data-models.md
│   └── deployment-guide.md
├── src/
│   ├── backend/
│   ├── voice-processing/
│   └── db/
├── tests/
└── demo/

## AWS Technical Stack Mapping

This section bridges the Community Voice Bridge system design with the AWS technical stack, providing a clear definition of AWS services used across the architecture.

### Large Language Model (Reasoning & Orchestration)
- **Amazon Bedrock**
  - Models: Claude 3.5 Sonnet or Mistral 7B
  - Used for conversational reasoning, eligibility inference, and simple prompt chaining.
  - Enables secure, managed LLM access without infrastructure overhead.

### Voice Processing Layer
The Voice Processing Layer enables real-time, voice-first interaction over low-bandwidth networks, supporting Hindi, English, and regional languages (Kannada, Tamil, Telugu) for the prototype.

- **Amazon Transcribe Streaming (Primary STT)**
  - Performs real-time streaming speech-to-text for citizen calls.
  - Optimized for Indian accents and low-bitrate 2G audio conditions.
  - Integrated directly with Amazon Connect for production readiness.

- **Amazon Polly (Neural Text-to-Speech)**
  - Generates natural, human-like responses in Hindi (Aditi voice) and English.
  - Used for AI responses and playback of peer success stories.
  - Optimized for clarity over 2G network conditions.

- **Multi-language Expansion**
  - Bhashini API integration enables 5 languages in prototype (Hindi, English, Kannada, Tamil, Telugu).
  - Production roadmap: Expand to 20+ Indian languages based on regional demand.

- **Fallback & Resilience Strategy**
  - If transcription confidence drops below threshold:
    - User is prompted to move to a quieter environment.
    - System falls back to **DTMF (keypad) input** via Amazon Connect IVR.
  - Ensures conversation continuity even under poor network or audio conditions.

### Telephony & Communication
- **Amazon Connect**
  - Handles proactive outbound calling and IVR fallback.
- **Amazon SNS**
  - Sends SMS messages with document instructions and follow-up notifications.

### Backend Logic & APIs
- **AWS Lambda**
  - Serverless execution of backend logic, including:
    - Conversation state management
    - Eligibility checks
    - Follow-up scheduling
    - API orchestration
  - Automatically scales during traffic spikes.

### Storage & Databases
- **Amazon S3**
  - Encrypted storage for success story audio files and generated voice responses.
- **Amazon RDS (PostgreSQL with pgvector)**
  - Stores user state, scheme metadata, and application tracking data.
  - pgvector extension enables semantic search without external vector databases.
- **AWS Lambda (In-memory sessions)**
  - Stateless session management for hackathon prototype.
  - Production may add ElastiCache Redis for distributed session storage.

### Retrieval & Knowledge Layer
- **PostgreSQL with pgvector**
  - Semantic retrieval of welfare schemes and success stories using vector similarity search.
  - Eliminates need for external vector databases in prototype.
- **Amazon Bedrock Embeddings**
  - Generates embeddings for semantic search and similarity matching.

## Security and Privacy Design

Security and privacy are core non-functional requirements of the Community Voice Bridge system.

### Data Security
- All data is encrypted at rest using **AWS KMS**.
- All data in transit is encrypted using **TLS 1.3**.
- Audio files stored in Amazon S3 use **Server-Side Encryption with KMS (SSE-KMS)**.

### Privacy-by-Design Principles
- No Aadhaar numbers, bank details, or sensitive personal data are stored.
- Phone numbers are stored only as **hashed identifiers**.
- **No Personally Identifiable Information (PII)** is included in LLM prompts or logs.
- LLM interaction logs contain only anonymized, non-identifiable context.

### Access Control & Auditing
- Role-based access control using AWS IAM.
- Least-privilege policies enforced across all AWS services.
- Audit logs maintained for all data access and system actions.

### User Consent & Control
- Users can opt out of the system at any time.
- Data retention is time-bound and automatically expired.
- Privacy controls are accessible via USSD without requiring internet access.

This design ensures compliance with India’s Digital Personal Data Protection (DPDP) Act while maintaining trust for underserved communities.

# Observability & Guardrails

This section ensures the AI system remains reliable, safe, and auditable during operation.

## System Monitoring
- **Amazon CloudWatch**
  - Monitors API latency, Lambda execution errors, and call-processing failures.
  - Tracks system health during peak outreach campaigns.
  - Alerts triggered on abnormal failure rates or latency spikes.

## AI Safety & Guardrails
- **Amazon Bedrock Guardrails**
  - Prevents the AI from providing financial advice, legal guidance, or inappropriate responses.
  - Enforces tone, language, and content boundaries suitable for public welfare interactions.
  - Blocks hallucinated scheme details or unsupported claims.

## Human-in-the-Loop (HITL)
- AI responses include a confidence score for eligibility and guidance.
- If confidence falls below **0.6**, the system:
  - Flags the interaction for review.
  - Escalates the case to a human agent or support workflow.
- Ensures high trust and correctness for sensitive welfare interactions.

This observability framework ensures Community Voice Bridge remains transparent, safe, and trustworthy at scale.
