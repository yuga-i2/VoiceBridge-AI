> **Tools & Methodology Note**
>
> This requirements document and the accompanying design specifications were conceptualized and structured using **Kiro**, ensuring alignment with AWS-native architectural patterns and hackathon documentation standards.

# Functional Requirements

## Core User Stories

### US-001: Proactive Outreach
**As a** rural citizen (e.g., Ramesh, a 55-year-old farmer living in a low-connectivity zone without smartphone access)
**I want** to receive a call in my local language about welfare schemes
**So that** I can access benefits without navigating complex portals

**Acceptance Criteria:**
- System simulates outbound call (web UI triggers call flow; production uses Amazon Connect)
- Call delivered in user's preferred language (Hindi, English, Kannada, Tamil, Telugu supported in prototype)
- Works on basic feature phone with 2G connectivity
- User can opt-out immediately if not interested
- Privacy choice offered within first 30 seconds of call

### US-002: Eligibility Discovery
**As a** farmer with limited literacy
**I want** AI to identify schemes I'm eligible for through conversation
**So that** I don't need to search or fill complex forms

**Acceptance Criteria:**
- AI asks natural questions about land ownership, crops, family size
- System matches against 50 curated schemes using PostgreSQL keyword search
  (production scales to 20K+ via Bedrock Knowledge Bases)
- Returns top 3 most relevant schemes with eligibility percentage
- Explains why user qualifies in simple terms
- Identifies missing documents needed for application

### US-003: Trust Building Through Peer Stories
**As a** skeptical first-time user
**I want** to hear success stories from people like me
**So that** I can trust the system isn't a scam

**Acceptance Criteria:**
- System detects hesitation through conversation patterns
- Retrieves semantically relevant success story (30 seconds max)
- Story matches user's language, region, and situation
- Story includes authentic voice recording with consent
- User can request to skip or hear another story

### US-004: Privacy Control
**As a** privacy-conscious user
**I want** to control my data through simple phone codes
**So that** I can participate without losing control

**Acceptance Criteria:**
- USSD code simulation in web UI (production requires telecom partnership for real USSD)
- Shows what data is stored (phone number, language preference only)
- Allows immediate data deletion request
- Confirms deletion within 24 hours via SMS
- No sensitive data (Aadhaar, bank details) ever stored

### US-005: Follow-up Assistance
**As a** user in the application process
**I want** proactive reminders and guidance
**So that** I don't forget steps or miss deadlines

**Acceptance Criteria:**
- Scheduled follow-up calls simulated in demo timeline (production uses EventBridge scheduler)
- SMS reminders with visual document checklists
- Progress tracking: "You've completed 2 of 4 steps"
- Escalation to human agent if stuck for >7 days
- Celebration call upon successful benefit receipt

## System Requirements (Hackathon Deliverables)

### SR-001: Voice Memory Network (Unique Differentiator)
**Requirement**: System MUST demonstrate peer success story playback as trust-building mechanism
**Rationale**: Core USP differentiating from chatbots; addresses rural skepticism
**Prototype Implementation**: 3 pre-recorded 30-second Hindi stories with keyword-based matching
**Success Criteria**: Demo shows 1 success story played during skeptical user scenario

### SR-002: India-Specific Technical Constraints
**Requirement**: System MUST work on 2G networks with <50 kbps bandwidth
**Rationale**: 58% rural India lacks smartphones; 2G dominant in target regions
**Prototype Implementation**: Audio compressed to 16kbps; simulated network throttling in demo
**Success Criteria**: Demo call quality remains >80% intelligible at 2G speeds

### SR-003: Process Flow Diagram Deliverable
**Requirement**: Submission MUST include visual end-to-end user journey diagram
**Rationale**: Required hackathon deliverable per judging criteria
**Location**: See design.md Section "User Journey Flow"
**Success Criteria**: Diagram shows 6 stages from discovery to benefit delivery

### SR-004: Privacy-First Architecture (DPDP Compliance)
**Requirement**: System MUST NOT store Aadhaar, bank details, or call recordings without consent
**Rationale**: India DPDP Act 2023 compliance; builds rural trust
**Prototype Implementation**: Only phone hash + language stored; USSD privacy controls
**Success Criteria**: Demo shows data deletion via *123*PRIVACY# code

## Prototype Scope (Hackathon Submission – ~10 Days)

### In-Scope for Hackathon Prototype (~10 Days):
✅ Single-scheme demo: Kisan Credit Card (KCC) application
✅ Multi-language voice conversation (Hindi, English, Kannada, Tamil, Telugu via Bhashini)
✅ Amazon Connect telephony integration (simulated outbound calling for demo)
✅ Basic eligibility matching (50 schemes, PostgreSQL keyword + vector search)
✅ 3 pre-recorded success stories with semantic matching
✅ SMS delivery with document checklist
✅ Privacy dashboard (USSD code simulation)
✅ Amazon RDS PostgreSQL for production-grade data persistence
✅ Optional: Amazon ElastiCache Redis for session caching (performance optimization)
✅ Demo: Complete end-to-end journey for 1 test user across multiple languages

### Out-of-Scope (Future Roadmap):

❌ Advanced Orchestration  
   (Prototype uses simple prompt chaining—eligibility discovery,
   trust-building, and guidance; production may use AWS Step Functions for workflow automation)
❌ External Vector Databases  
   (Prototype uses PostgreSQL pgvector; production may evaluate Amazon OpenSearch for scale)
❌ Voice Memory Network at national scale  
   (Prototype demonstrates the concept using 3 pre-recorded success stories with
   manual semantic matching)
❌ Fully automated proactive discovery engine  
   (Prototype uses manual user selection to simulate outreach and discovery)
❌ Production-grade telephony integration (Amazon Connect / Twilio)  
   (Prototype uses simulated or pre-recorded calls for demo purposes)
❌ Large-scale scheme database (20,000+ schemes)  
   (Prototype includes 50 curated schemes, with a single-scheme end-to-end demo
   focused on Kisan Credit Card – KCC)

# Non-Functional Requirements

## Performance Requirements (Pilot-Scoped)
> **Note**: Production-grade performance targets for 1,000-user pilot deployment

- **Response Time**: <3 seconds per AI turn (optimized Bedrock API configuration)
- **Call Quality**: >90% clarity on 2G networks with adaptive bitrate
- **Availability**: 99.5% uptime during pilot phase
- **Concurrent Users**: 20 simultaneous sessions (pilot scale capacity)
- **Latency**: <1.5 seconds for voice transcription (Amazon Transcribe Streaming)
- **Multi-language Processing**: <2 seconds for Bhashini translation layer
- **API Throughput**: Support 5,000 API calls during peak hours

## Scalability Requirements
- **Horizontal Scaling**: Stateless architecture for easy replication
- **Database**: PostgreSQL with pgvector extension for semantic search
- **API Rate Limits**: Bedrock API calls max 500 during 1-hour demo window (free tier compliance)
- **Storage**: S3 for voice recordings with lifecycle policies
- **Session Management**: In-memory for prototype (Redis for production scale)

## Security Requirements
- **Encryption**: TLS 1.3 for all data in transit
- **Data at Rest**: AES-256 encryption for stored voice recordings
- **Authentication**: API key rotation supported (manual during prototype)
- **PII Handling**: Zero storage of Aadhaar, bank details
- **Access Control**: Role-based access for admin dashboard

## Privacy Requirements
- **Data Minimization**: Collect only phone number (hashed) + language preference
- **Consent Management**: Explicit opt-in for voice recording storage
- **Right to Deletion**: USSD-triggered data deletion within 24 hours
- **Anonymization**: All success stories stripped of PII before storage
- **Retention Policy**: Auto-delete user data after 90 days

## Accessibility Requirements
- **Language Support**: Hindi + English + regional languages (Kannada, Tamil, Telugu) via Bhashini API integration; expandable to 20+ languages in production
- **Voice Clarity**: Text-to-speech optimized for elderly users (slower pace option)
- **Error Handling**: Graceful degradation if speech recognition fails (fallback to DTMF)
- **Network Resilience**: Works on 2G networks with <50 kbps bandwidth

# Technical Constraints

## Hackathon-Specific Constraints
- **Time**: ~10-day development window
- **Team Size**: 2-4 developers
- **Budget**: Free tier AWS services only
- **Infrastructure**: No production telephony (simulated calls)
- **Testing**: Limited to 10-20 test users during demo

## AWS Service Budget Allocation
Prototype phase uses pay-as-you-go AWS services with the following capacity planning:

- **Amazon Bedrock**: Sufficient for 1,000+ test conversations
- **Amazon Transcribe**: Capacity for 500+ minutes of voice processing
- **Amazon Polly**: Support for 100K+ characters of voice synthesis
- **Amazon Connect**: Telephony infrastructure for 100+ test calls
- **Amazon RDS PostgreSQL**: 16-day deployment with automated backups
- **Bhashini API**: Regional language translation for multi-language support

Budget allocated to ensure uninterrupted development and testing during 16-day prototype phase.

## Data Constraints
- **Scheme Database**: Curated 50 schemes from 20,000+ government schemes
- **Training Data**: 500 Q&A pairs for fine-tuning (vs 50,000 in full system)
- **Success Stories**: 3 pre-recorded stories (vs Voice Memory Network at scale)

## Technical Debt Accepted for Prototype
- Hardcoded language models (no dynamic switching)
- Simple keyword-based intent detection (vs full NLU)
- Mock telephony interface (vs real Amazon Connect)
- Single-region deployment (vs multi-region)
- Manual scheme updates (vs real-time government API sync)

# Prototype Cost Estimate (16-Day Hackathon)

## AWS Service Usage
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

# Evaluation & Success Metrics

This section defines how the effectiveness and real-world impact of the Community Voice Bridge system will be evaluated, with a focus on outcome-driven impact economics as emphasized in the evaluation criteria.

## Metric 1: Call Completion Rate
- **Definition**: Percentage of outbound calls that are answered and completed by users.
- **Goal**: >80% call completion rate.
- **Why it matters**: High completion rates indicate accessibility, trust, and effective voice-first engagement, especially for rural and low-literacy users.

## Metric 2: Eligibility Matching Accuracy
- **Definition**: Accuracy of AI-driven scheme eligibility recommendations compared against ground-truth eligibility rules.
- **Goal**: >90% accuracy in scheme eligibility matching.
- **Why it matters**: Accurate eligibility discovery reduces misinformation, prevents false expectations, and builds long-term trust in the system.

## Metric 3: User Trust Score
- **Definition**: Trust level inferred through follow-up sentiment analysis and post-interaction feedback signals (tone, engagement, continuation).
- **Goal**: Positive trust score for >75% of users.
- **Why it matters**: Trust is critical in welfare delivery; higher trust directly correlates with follow-through and benefit realization.

## Metric 4: Outcome Conversion Rate
- **Definition**: Percentage of users who successfully complete at least one welfare application after initial interaction.
- **Goal**: 30-50% conversion from awareness to application initiation in pilot phase.
- **Rationale**: Conservative estimate based on rural phone interaction studies and first-time voice AI adoption patterns. Research shows 30-50% completion rates for voice-first government services in similar demographics.
- **Optimization Path**: Voice Memory Network expected to improve completion to 60%+ as success story library grows and trust compounds.
- **Why it matters**: Measures real impact beyond information delivery, aligning with outcome-based welfare access. Even at 30% completion, significantly outperforms traditional 5-12% call center completion rates.

## Metric 5: Cost Efficiency
- **Definition**: Average cost incurred per successfully guided user.
- **Goal**: ₹15-25 per user during pilot phase (1,000 users); optimizes to ₹3-5 per user at scale (100K+ users through infrastructure optimization and bulk API pricing).
- **Why it matters**: Demonstrates economic feasibility at pilot scale while showing clear path to cost efficiency at production scale through economies of scale.

## Cost Optimization Path
- **Pilot Phase (1K users)**: ₹15-25/user (full-featured, production-grade infrastructure)
- **Scale Phase (10K users)**: ₹8-12/user (optimized API usage, reserved instances)
- **Production Scale (100K+ users)**: ₹3-5/user (bulk pricing, auto-scaling efficiency)

This tiered approach ensures immediate viability while demonstrating clear cost reduction trajectory.

These metrics ensure that Community Voice Bridge is evaluated not just on technical performance, but on measurable social impact, trust, and economic efficiency.

# Scalability & Future Readiness

Community Voice Bridge is designed to transition from a manually simulated prototype to a fully automated, production-grade AWS system.

## Pilot to Production Scaling Path

**Pilot Phase (Current)**
- 1,000 users across 2-3 districts
- 5 languages (Hindi, English, Kannada, Tamil, Telugu)
- 50 curated welfare schemes
- Cost: ₹15-25 per user

**Early Production (6-12 months)**
- 10,000 users across 10 districts  
- 10 languages via expanded Bhashini integration
- 500 welfare schemes with automated ingestion
- Cost: ₹8-12 per user (optimized pricing)

**Full Production (12-24 months)**
- 100K+ users statewide
- 20+ languages
- 5,000+ schemes with real-time government API sync
- Cost: ₹3-5 per user (economies of scale)

This phased approach ensures immediate viability while demonstrating clear path to nationwide scale.

- **Automated Knowledge Ingestion**
  - Scale from a 50-scheme curated dataset to 20,000+ government schemes using a dynamic RAG pipeline powered by **Amazon Bedrock Knowledge Bases**.
  - Enables real-time scheme updates without manual intervention.

- **Production Telephony**
  - Replace simulated calls with **Amazon Connect** integrated with **AWS Lambda** for high-concurrency outbound dialing.
  - Enables real-time call monitoring, sentiment detection, and intelligent call routing.

- **Agentic Orchestration**
  - Evolve from simple prompt chaining to automated workflow orchestration using **AWS Step Functions**.
  - Specialized workflow states handle:
    - Eligibility Discovery
    - Trust Building via Peer Stories
    - Document Collection & Submission Tracking

This roadmap demonstrates a clear and realistic path from hackathon prototype to a scalable, AWS-native production deployment.
