# Design Document: ImpactLedger

## Executive Summary

ImpactLedger is an AI-first cloud-native platform architected on AWS to democratize CSR funding access for rural NGOs in India. The design leverages serverless computing, managed AI services, and edge-optimized architectures to deliver enterprise-grade compliance automation and semantic matchmaking while maintaining accessibility for low-bandwidth, low-literacy users.

This document outlines the technical architecture, justifies technology choices against hackathon criteria (Technical Aptness, Innovation, Business Feasibility), and provides implementation blueprints for a production-ready MVP.

## Design Principles

1. **AI-Native Architecture**: AI is not a feature add-on but the core differentiator—automating legal compliance and enabling semantic discovery
2. **Offline-First Mobile**: Rural connectivity constraints drive architectural decisions for data sync and local processing
3. **Serverless-First Backend**: Cost efficiency and auto-scaling for unpredictable NGO funding cycles
4. **Security by Design**: DPDP Act compliance, data localization, and audit trails built into every layer
5. **Progressive Enhancement**: Graceful degradation from high-bandwidth corporate users to 2G rural users

## Technology Stack Justification

### Why AWS?


1. **Comprehensive AI/ML Portfolio**: AWS Bedrock (Generative AI), Textract (OCR), OpenSearch (Vector Search), Polly/Transcribe (Voice) provide end-to-end AI capabilities without vendor lock-in
2. **Indian Data Residency**: AWS Mumbai (ap-south-1) and Hyderabad (ap-south-2) regions ensure DPDP Act compliance with data localization
3. **Serverless Maturity**: Lambda, API Gateway, Aurora Serverless offer proven cost optimization for variable workloads
4. **Enterprise Security**: AWS KMS, WAF, GuardDuty, and compliance certifications (ISO 27001, SOC 2) meet corporate CSR audit requirements
5. **Cost Efficiency**: Pay-per-use pricing aligns with hackathon constraints and NGO budget realities

### Why AI is the Core Differentiator?

**Problem**: Traditional NGO-CSR platforms are glorified databases with keyword search. They fail because:
- Rural NGOs cannot afford legal consultants to draft Form 10BD (₹15,000-25,000 per form)
- Corporations cannot discover "hidden gem" grassroots NGOs lacking formal documentation
- Manual due diligence takes 60-90 days, causing funding delays

**AI Solution**:
1. **Compliance Automation (Textract + Bedrock)**: Reduces Form 10BD creation from 8 hours to 10 minutes, saving ₹20,000 per NGO annually
2. **Semantic Matchmaking (OpenSearch Vector Engine)**: Discovers NGOs based on impact narratives, not just tags—increasing rural NGO visibility by 3x
3. **Voice-First Interface (Polly/Transcribe)**: Enables vernacular data entry for low-literacy users, expanding addressable market by 5x


**ROI Justification**: For a ₹50 crore CSR budget, reducing due diligence time by 50% saves ₹2-3 crore in consulting fees annually.

### Technology Stack Summary

| Layer | Technology | Justification |
|-------|-----------|---------------|
| **Mobile App** | React Native | Single codebase for Android/iOS; offline-first with AsyncStorage; 60% faster development |
| **Web Dashboard** | React.js + TypeScript | Component reusability with mobile; strong typing for enterprise code quality |
| **API Gateway** | AWS API Gateway | Managed service; built-in throttling, caching, and API key management |
| **Compute** | AWS Lambda (Node.js 20.x) | Serverless cost optimization; auto-scaling; 100ms billing granularity |
| **OCR Engine** | Amazon Textract | 95%+ accuracy on handwritten Hindi/English; pre-trained models; no ML expertise required |
| **Generative AI** | AWS Bedrock (Claude 3 Haiku) | Fast inference (2-3s); low cost ($0.25/1M tokens); structured output for forms |
| **Vector Search** | Amazon OpenSearch (Vector Engine) | Semantic search with k-NN; scales to 10TB+ embeddings; sub-second queries |
| **Voice AI** | Amazon Polly + Transcribe | 5 Indian languages; custom vocabulary; streaming support for low latency |
| **Relational DB** | Amazon Aurora PostgreSQL Serverless v2 | Auto-scaling ACUs; 99.99% SLA; ACID compliance for financial transactions |
| **Object Storage** | Amazon S3 (Intelligent-Tiering) | 99.999999999% durability; automatic cost optimization; lifecycle policies |
| **Caching** | Amazon ElastiCache (Redis) | Sub-millisecond latency; session management; API response caching |
| **CDN** | Amazon CloudFront | Edge caching for mobile app assets; reduces latency for rural 2G/3G users |
| **Authentication** | Amazon Cognito | Managed user pools; MFA; social login; SAML for corporate SSO |
| **Monitoring** | Amazon CloudWatch + X-Ray | Distributed tracing; custom metrics; anomaly detection; log aggregation |
| **CI/CD** | AWS CodePipeline + CodeBuild | Automated testing; blue-green deployments; infrastructure-as-code |



## High-Level Architecture (HLD)

### System Context Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         ImpactLedger Platform                            │
│                                                                           │
│  ┌──────────────┐         ┌──────────────┐         ┌──────────────┐    │
│  │   NGO Mobile │         │  CSR Web     │         │   Admin      │    │
│  │   App        │◄───────►│  Dashboard   │◄───────►│   Portal     │    │
│  │ (React Native)│         │  (React.js)  │         │  (React.js)  │    │
│  └──────────────┘         └──────────────┘         └──────────────┘    │
│         │                         │                         │            │
│         └─────────────────────────┼─────────────────────────┘            │
│                                   │                                      │
│                          ┌────────▼────────┐                            │
│                          │  Amazon         │                            │
│                          │  CloudFront     │                            │
│                          │  (CDN)          │                            │
│                          └────────┬────────┘                            │
│                                   │                                      │
│                          ┌────────▼────────┐                            │
│                          │  API Gateway    │                            │
│                          │  (REST + WSS)   │                            │
│                          └────────┬────────┘                            │
│                                   │                                      │
│         ┌─────────────────────────┼─────────────────────────┐           │
│         │                         │                         │           │
│    ┌────▼─────┐          ┌───────▼──────┐          ┌──────▼──────┐    │
│    │  Lambda  │          │   Lambda     │          │   Lambda    │    │
│    │  (Auth)  │          │  (Business)  │          │   (AI)      │    │
│    └────┬─────┘          └───────┬──────┘          └──────┬──────┘    │
│         │                         │                         │           │
│    ┌────▼─────┐          ┌───────▼──────┐          ┌──────▼──────┐    │
│    │ Cognito  │          │   Aurora     │          │  Bedrock    │    │
│    │          │          │  PostgreSQL  │          │  Textract   │    │
│    └──────────┘          │              │          │  OpenSearch │    │
│                          │  ElastiCache │          │  Polly      │    │
│                          └──────────────┘          └─────────────┘    │
│                                                                         │
│                          ┌──────────────┐                              │
│                          │   Amazon S3  │                              │
│                          │  (Documents) │                              │
│                          └──────────────┘                              │
└─────────────────────────────────────────────────────────────────────────┘
```



### Architecture Flow: NGO Mobile App to Cloud

**Scenario**: NGO user uploads handwritten receipt for compliance automation

1. **Mobile App (Offline-First)**:
   - User captures receipt photo using device camera
   - Image compressed to <2MB using React Native Image Picker
   - Metadata (timestamp, GPS, user_id) attached locally
   - Queued in AsyncStorage if offline; visual indicator shows "Pending Sync"

2. **Connectivity Detection**:
   - NetInfo library monitors network state
   - On connectivity restoration, sync manager triggers upload queue
   - Exponential backoff retry (3 attempts) for failed uploads

3. **Edge Layer (CloudFront + API Gateway)**:
   - CloudFront edge location (Mumbai/Hyderabad) receives request
   - API Gateway validates JWT token (issued by Cognito)
   - Request throttled at 100 req/sec per user to prevent abuse
   - Binary image data forwarded to Lambda

4. **Compute Layer (Lambda)**:
   - `uploadReceiptHandler` Lambda triggered
   - Generates pre-signed S3 URL for direct upload (bypasses Lambda 6MB limit)
   - Returns upload URL to mobile app

5. **Storage Layer (S3)**:
   - Mobile app uploads image directly to S3 using pre-signed URL
   - S3 event notification triggers `processReceiptHandler` Lambda
   - Image stored in `s3://impactledger-receipts/{ngo_id}/{year}/{receipt_id}.jpg`

6. **AI Processing Pipeline** (detailed in AI Workflow section):
   - Textract extracts text → Bedrock structures data → Aurora stores result
   - Mobile app polls `/api/receipts/{receipt_id}/status` for completion

7. **Response to Mobile**:
   - Push notification via SNS when processing complete
   - App fetches structured data and displays for user review



### Architecture Flow: CSR Semantic Search

**Scenario**: CSR investor searches "water conservation in drought-prone Maharashtra"

1. **Web Dashboard (React.js)**:
   - User enters natural language query in search bar
   - Debounced API call (500ms) to `/api/search/semantic`

2. **API Gateway + Lambda**:
   - `semanticSearchHandler` Lambda receives query
   - Checks ElastiCache for cached results (TTL: 1 hour)
   - Cache miss triggers AI processing

3. **AI Processing**:
   - Query sent to Bedrock to generate embedding vector (1536 dimensions)
   - Vector sent to OpenSearch k-NN query against NGO profile embeddings
   - OpenSearch returns top 20 matches with similarity scores

4. **Business Logic**:
   - Lambda enriches results with NGO metadata from Aurora
   - Applies filters (verified status, active projects, geographic bounds)
   - Calculates predicted SROI using ML model (stored in S3)

5. **Response**:
   - Results cached in ElastiCache
   - JSON response with NGO profiles, relevance scores, and match explanations
   - Dashboard renders cards with NGO details and "Express Interest" CTA



## Low-Level Design (LLD)

### API Specification

#### Authentication & Authorization

**POST /api/auth/register**
```json
Request:
{
  "user_type": "ngo" | "csr_investor" | "admin",
  "phone": "+91XXXXXXXXXX",
  "email": "user@example.com",
  "organization_name": "string",
  "registration_number": "string" // For NGOs
}

Response:
{
  "user_id": "uuid",
  "verification_required": true,
  "otp_sent": true
}
```

**POST /api/auth/verify-otp**
```json
Request:
{
  "phone": "+91XXXXXXXXXX",
  "otp": "123456"
}

Response:
{
  "access_token": "jwt_token",
  "refresh_token": "jwt_token",
  "expires_in": 3600,
  "user": {
    "user_id": "uuid",
    "user_type": "ngo",
    "profile_complete": false
  }
}
```



#### NGO Profile Management

**POST /api/ngo/profile**
```json
Request:
{
  "organization_name": "string",
  "registration_number": "string",
  "registration_type": "80G" | "12A" | "FCRA",
  "cause_areas": ["education", "health", "environment"],
  "geographic_focus": {
    "state": "Maharashtra",
    "districts": ["Pune", "Satara"],
    "villages": ["Village1", "Village2"]
  },
  "description": "string",
  "impact_narrative": "string", // Unstructured text for semantic indexing
  "contact": {
    "primary_phone": "+91XXXXXXXXXX",
    "email": "ngo@example.org",
    "address": "string"
  }
}

Response:
{
  "ngo_id": "uuid",
  "profile_completeness": 65,
  "verification_status": "pending",
  "next_steps": ["Upload registration certificate", "Add team members"]
}
```

**GET /api/ngo/profile/{ngo_id}**
```json
Response:
{
  "ngo_id": "uuid",
  "organization_name": "string",
  "verification_status": "verified" | "pending" | "rejected",
  "cause_areas": ["education"],
  "geographic_focus": {...},
  "impact_metrics": {
    "beneficiaries_reached": 5000,
    "projects_completed": 12,
    "total_funding_received": 2500000
  },
  "documents": [
    {
      "document_id": "uuid",
      "type": "registration_certificate",
      "url": "s3_presigned_url",
      "verified": true
    }
  ]
}
```



#### Compliance Automation (AI-Powered)

**POST /api/compliance/upload-receipt**
```json
Request (multipart/form-data):
{
  "image": File, // Max 5MB
  "receipt_type": "expense" | "donation" | "salary",
  "project_id": "uuid",
  "metadata": {
    "captured_at": "2024-01-15T10:30:00Z",
    "gps_location": {"lat": 18.5204, "lon": 73.8567}
  }
}

Response:
{
  "receipt_id": "uuid",
  "status": "processing",
  "estimated_completion": "2024-01-15T10:30:05Z",
  "polling_url": "/api/compliance/receipt/{receipt_id}/status"
}
```

**GET /api/compliance/receipt/{receipt_id}/status**
```json
Response:
{
  "receipt_id": "uuid",
  "status": "completed" | "processing" | "failed",
  "extracted_data": {
    "vendor_name": "ABC Suppliers",
    "amount": 15000.00,
    "currency": "INR",
    "date": "2024-01-10",
    "items": [
      {"description": "Water pumps", "quantity": 2, "unit_price": 7500}
    ],
    "confidence_score": 0.92
  },
  "requires_review": false,
  "ai_suggestions": {
    "category": "Equipment Purchase",
    "sdg_mapping": ["SDG 6: Clean Water"]
  }
}
```

**POST /api/compliance/generate-form-10bd**
```json
Request:
{
  "ngo_id": "uuid",
  "financial_year": "2023-24",
  "receipt_ids": ["uuid1", "uuid2", "uuid3"],
  "additional_info": {
    "project_description": "string",
    "beneficiary_count": 500
  }
}

Response:
{
  "form_id": "uuid",
  "status": "generated",
  "download_url": "s3_presigned_url",
  "preview_url": "/api/compliance/form/{form_id}/preview",
  "editable": true,
  "expires_at": "2024-01-15T11:30:00Z"
}
```



#### Semantic Matchmaking

**POST /api/search/semantic**
```json
Request:
{
  "query": "Find water conservation projects in drought-prone Maharashtra",
  "filters": {
    "cause_areas": ["environment", "water"],
    "states": ["Maharashtra"],
    "min_beneficiaries": 1000,
    "verification_status": "verified"
  },
  "limit": 20,
  "offset": 0
}

Response:
{
  "total_results": 47,
  "results": [
    {
      "ngo_id": "uuid",
      "organization_name": "Water Warriors Foundation",
      "relevance_score": 0.89,
      "match_explanation": "Strong semantic match on water conservation, drought mitigation, and Maharashtra geographic focus",
      "cause_areas": ["environment", "water"],
      "geographic_focus": {"state": "Maharashtra", "districts": ["Satara", "Solapur"]},
      "impact_summary": "Implemented rainwater harvesting in 50 villages, benefiting 15,000 farmers",
      "predicted_sroi": 4.2,
      "sroi_confidence": "high",
      "funding_required": 5000000,
      "profile_url": "/ngo/uuid"
    }
  ],
  "search_metadata": {
    "query_embedding_time_ms": 120,
    "search_time_ms": 450,
    "cached": false
  }
}
```

**POST /api/investor/express-interest**
```json
Request:
{
  "investor_id": "uuid",
  "ngo_id": "uuid",
  "project_id": "uuid",
  "message": "string",
  "funding_amount": 2000000
}

Response:
{
  "interest_id": "uuid",
  "status": "sent",
  "ngo_notified": true,
  "next_steps": "NGO will respond within 48 hours"
}
```



#### Voice Interface

**POST /api/voice/transcribe**
```json
Request (multipart/form-data):
{
  "audio": File, // Max 10MB, formats: mp3, wav, m4a
  "language": "hi-IN" | "mr-IN" | "ta-IN" | "te-IN" | "bn-IN",
  "context": "profile_update" | "project_description" | "impact_report"
}

Response:
{
  "transcription_id": "uuid",
  "text": "हमने पिछले महीने 50 परिवारों को स्वच्छ पानी उपलब्ध कराया",
  "text_english": "We provided clean water to 50 families last month",
  "confidence": 0.91,
  "language_detected": "hi-IN",
  "processing_time_ms": 2800
}
```

**POST /api/voice/synthesize**
```json
Request:
{
  "text": "आपका प्रोफाइल सफलतापूर्वक अपडेट हो गया है",
  "language": "hi-IN",
  "voice_id": "Aditi" // Amazon Polly voice
}

Response:
{
  "audio_url": "s3_presigned_url",
  "duration_seconds": 3.5,
  "format": "mp3"
}
```



#### CSR Dashboard & Analytics

**GET /api/investor/dashboard**
```json
Response:
{
  "investor_id": "uuid",
  "portfolio_summary": {
    "total_funding_committed": 50000000,
    "total_funding_disbursed": 35000000,
    "active_projects": 12,
    "completed_projects": 8,
    "beneficiaries_reached": 125000
  },
  "impact_metrics": {
    "sdg_distribution": {
      "SDG 1": 15000000,
      "SDG 4": 20000000,
      "SDG 6": 15000000
    },
    "geographic_distribution": {
      "Maharashtra": 25000000,
      "Karnataka": 15000000,
      "Tamil Nadu": 10000000
    },
    "average_sroi": 3.8
  },
  "recent_activities": [
    {
      "activity_type": "funding_disbursed",
      "ngo_name": "Education for All",
      "amount": 2000000,
      "timestamp": "2024-01-10T14:30:00Z"
    }
  ],
  "compliance_status": {
    "form_10be_submitted": true,
    "audit_pending": false,
    "next_deadline": "2024-03-31"
  }
}
```

**GET /api/investor/reports/export**
```json
Query Parameters:
?format=pdf|excel&financial_year=2023-24&include_compliance=true

Response:
{
  "report_id": "uuid",
  "download_url": "s3_presigned_url",
  "expires_at": "2024-01-15T12:00:00Z",
  "file_size_bytes": 2457600
}
```



### Database Schema

#### Core Tables

**users**
```sql
CREATE TABLE users (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_type VARCHAR(20) NOT NULL CHECK (user_type IN ('ngo', 'csr_investor', 'admin')),
    phone VARCHAR(15) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE,
    cognito_sub VARCHAR(255) UNIQUE NOT NULL,
    profile_complete BOOLEAN DEFAULT FALSE,
    verification_status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login_at TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE
);

CREATE INDEX idx_users_type ON users(user_type);
CREATE INDEX idx_users_verification ON users(verification_status);
```

**ngos**
```sql
CREATE TABLE ngos (
    ngo_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    organization_name VARCHAR(255) NOT NULL,
    registration_number VARCHAR(100) UNIQUE NOT NULL,
    registration_type VARCHAR(20)[] DEFAULT ARRAY['80G'], -- Array of 80G, 12A, FCRA
    cause_areas VARCHAR(50)[] NOT NULL,
    description TEXT,
    impact_narrative TEXT, -- Unstructured text for semantic search
    embedding VECTOR(1536), -- OpenSearch vector for semantic matching
    geographic_focus JSONB, -- {state, districts, villages}
    contact_info JSONB,
    profile_completeness INTEGER DEFAULT 0,
    verification_status VARCHAR(20) DEFAULT 'pending',
    verified_at TIMESTAMP,
    total_beneficiaries INTEGER DEFAULT 0,
    total_funding_received DECIMAL(15,2) DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_ngos_verification ON ngos(verification_status);
CREATE INDEX idx_ngos_cause_areas ON ngos USING GIN(cause_areas);
CREATE INDEX idx_ngos_geographic ON ngos USING GIN(geographic_focus);
```



**csr_investors**
```sql
CREATE TABLE csr_investors (
    investor_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    company_name VARCHAR(255) NOT NULL,
    cin VARCHAR(21) UNIQUE, -- Corporate Identification Number
    annual_csr_budget DECIMAL(15,2),
    preferred_cause_areas VARCHAR(50)[],
    preferred_states VARCHAR(50)[],
    investment_criteria JSONB, -- {min_beneficiaries, max_funding_per_project, etc.}
    compliance_info JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_investors_cause_areas ON csr_investors USING GIN(preferred_cause_areas);
```

**projects**
```sql
CREATE TABLE projects (
    project_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ngo_id UUID REFERENCES ngos(ngo_id) ON DELETE CASCADE,
    project_name VARCHAR(255) NOT NULL,
    description TEXT,
    cause_area VARCHAR(50) NOT NULL,
    sdg_mapping VARCHAR(20)[], -- Array of SDG codes
    geographic_scope JSONB,
    funding_required DECIMAL(15,2) NOT NULL,
    funding_received DECIMAL(15,2) DEFAULT 0,
    target_beneficiaries INTEGER,
    actual_beneficiaries INTEGER DEFAULT 0,
    start_date DATE,
    end_date DATE,
    status VARCHAR(20) DEFAULT 'draft', -- draft, active, completed, cancelled
    predicted_sroi DECIMAL(5,2),
    sroi_confidence VARCHAR(20), -- low, medium, high
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_projects_ngo ON projects(ngo_id);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_projects_cause ON projects(cause_area);
```



**receipts**
```sql
CREATE TABLE receipts (
    receipt_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ngo_id UUID REFERENCES ngos(ngo_id) ON DELETE CASCADE,
    project_id UUID REFERENCES projects(project_id) ON DELETE SET NULL,
    receipt_type VARCHAR(20) NOT NULL, -- expense, donation, salary
    s3_key VARCHAR(500) NOT NULL,
    s3_bucket VARCHAR(100) NOT NULL,
    file_size_bytes INTEGER,
    mime_type VARCHAR(50),
    processing_status VARCHAR(20) DEFAULT 'pending', -- pending, processing, completed, failed
    extracted_data JSONB, -- Textract output
    ai_confidence DECIMAL(3,2),
    requires_manual_review BOOLEAN DEFAULT FALSE,
    reviewed_by UUID REFERENCES users(user_id),
    reviewed_at TIMESTAMP,
    metadata JSONB, -- GPS, timestamp, device info
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_receipts_ngo ON receipts(ngo_id);
CREATE INDEX idx_receipts_project ON receipts(project_id);
CREATE INDEX idx_receipts_status ON receipts(processing_status);
```

**compliance_forms**
```sql
CREATE TABLE compliance_forms (
    form_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ngo_id UUID REFERENCES ngos(ngo_id) ON DELETE CASCADE,
    form_type VARCHAR(20) NOT NULL, -- 10BD, 10BE, annual_report
    financial_year VARCHAR(10) NOT NULL,
    receipt_ids UUID[], -- Array of receipt references
    generated_data JSONB, -- AI-generated form data
    s3_key VARCHAR(500), -- Final PDF location
    status VARCHAR(20) DEFAULT 'draft', -- draft, submitted, approved, rejected
    submitted_at TIMESTAMP,
    approved_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_forms_ngo ON compliance_forms(ngo_id);
CREATE INDEX idx_forms_type_year ON compliance_forms(form_type, financial_year);
```



**funding_transactions**
```sql
CREATE TABLE funding_transactions (
    transaction_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    investor_id UUID REFERENCES csr_investors(investor_id) ON DELETE CASCADE,
    ngo_id UUID REFERENCES ngos(ngo_id) ON DELETE CASCADE,
    project_id UUID REFERENCES projects(project_id) ON DELETE CASCADE,
    amount DECIMAL(15,2) NOT NULL,
    transaction_type VARCHAR(20) NOT NULL, -- commitment, disbursement, refund
    payment_method VARCHAR(50),
    payment_reference VARCHAR(100),
    status VARCHAR(20) DEFAULT 'pending', -- pending, completed, failed, cancelled
    disbursed_at TIMESTAMP,
    compliance_docs JSONB, -- References to Form 10BD, receipts, etc.
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_transactions_investor ON funding_transactions(investor_id);
CREATE INDEX idx_transactions_ngo ON funding_transactions(ngo_id);
CREATE INDEX idx_transactions_project ON funding_transactions(project_id);
CREATE INDEX idx_transactions_status ON funding_transactions(status);
```

**impact_reports**
```sql
CREATE TABLE impact_reports (
    report_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(project_id) ON DELETE CASCADE,
    ngo_id UUID REFERENCES ngos(ngo_id) ON DELETE CASCADE,
    reporting_period_start DATE NOT NULL,
    reporting_period_end DATE NOT NULL,
    beneficiaries_reached INTEGER,
    funds_utilized DECIMAL(15,2),
    narrative TEXT,
    media_urls JSONB, -- Array of S3 URLs for photos/videos
    sdg_impact JSONB, -- Mapping to SDG indicators
    actual_sroi DECIMAL(5,2),
    verified BOOLEAN DEFAULT FALSE,
    verified_by UUID REFERENCES users(user_id),
    verified_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_reports_project ON impact_reports(project_id);
CREATE INDEX idx_reports_ngo ON impact_reports(ngo_id);
CREATE INDEX idx_reports_period ON impact_reports(reporting_period_start, reporting_period_end);
```



**investor_interests**
```sql
CREATE TABLE investor_interests (
    interest_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    investor_id UUID REFERENCES csr_investors(investor_id) ON DELETE CASCADE,
    ngo_id UUID REFERENCES ngos(ngo_id) ON DELETE CASCADE,
    project_id UUID REFERENCES projects(project_id) ON DELETE CASCADE,
    message TEXT,
    proposed_funding DECIMAL(15,2),
    status VARCHAR(20) DEFAULT 'sent', -- sent, viewed, accepted, rejected, negotiating
    ngo_response TEXT,
    responded_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_interests_investor ON investor_interests(investor_id);
CREATE INDEX idx_interests_ngo ON investor_interests(ngo_id);
CREATE INDEX idx_interests_status ON investor_interests(status);
```

**audit_logs**
```sql
CREATE TABLE audit_logs (
    log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id UUID,
    ip_address INET,
    user_agent TEXT,
    request_data JSONB,
    response_status INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_user ON audit_logs(user_id);
CREATE INDEX idx_audit_action ON audit_logs(action);
CREATE INDEX idx_audit_created ON audit_logs(created_at);
```



## AI Workflow: End-to-End Processing Pipeline

### Workflow 1: Receipt Processing & Compliance Automation

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Receipt Upload to Form 10BD Generation                │
└─────────────────────────────────────────────────────────────────────────┘

Step 1: Image Upload
├─ NGO User captures receipt photo (handwritten Hindi/English)
├─ Mobile app compresses image to <2MB
├─ Uploads to S3 via pre-signed URL
└─ S3 event triggers Lambda: processReceiptHandler

Step 2: OCR Extraction (Amazon Textract)
├─ Lambda invokes Textract DetectDocumentText API
├─ Textract processes image (avg 2-3 seconds)
├─ Returns raw text blocks with confidence scores
└─ Example output:
    {
      "blocks": [
        {"text": "दिनांक: 10/01/2024", "confidence": 0.94},
        {"text": "राशि: ₹15,000", "confidence": 0.89},
        {"text": "विक्रेता: ABC Suppliers", "confidence": 0.92}
      ]
    }

Step 3: Structured Data Extraction (AWS Bedrock - Claude 3 Haiku)
├─ Lambda sends Textract output to Bedrock with prompt:
│   "Extract structured data from this receipt. Return JSON with:
│    vendor_name, amount, currency, date, items[], category"
├─ Bedrock processes in 1-2 seconds (Haiku optimized for speed)
├─ Returns structured JSON:
    {
      "vendor_name": "ABC Suppliers",
      "amount": 15000.00,
      "currency": "INR",
      "date": "2024-01-10",
      "items": [{"description": "Water pumps", "quantity": 2}],
      "category": "Equipment Purchase",
      "confidence": 0.91
    }
└─ If confidence < 0.85, flag for manual review

Step 4: Data Validation & Storage
├─ Lambda validates extracted data (date format, amount range)
├─ Stores in Aurora PostgreSQL receipts table
├─ Updates receipt status to "completed"
└─ Sends push notification to NGO user

Step 5: Form 10BD Generation (On-Demand)
├─ NGO user selects multiple receipts for financial year
├─ Clicks "Generate Form 10BD"
├─ Lambda aggregates receipt data from Aurora
├─ Sends to Bedrock with Form 10BD template prompt:
│   "Generate Form 10BD for Indian Income Tax compliance.
│    NGO: {name}, PAN: {pan}, Financial Year: {fy}
│    Receipts: {aggregated_data}
│    Follow official Form 10BD format with all mandatory fields"
├─ Bedrock generates structured form data
├─ Lambda uses PDF generation library (PDFKit) to create PDF
├─ Uploads PDF to S3
└─ Returns download URL to user

Step 6: User Review & Submission
├─ NGO user reviews generated form in mobile app
├─ Can edit fields if needed
├─ Submits for verification
└─ System maintains audit trail (original receipts + AI-generated form)
```



### Workflow 2: Semantic Matchmaking with Vector Search

```
┌─────────────────────────────────────────────────────────────────────────┐
│              CSR Search to NGO Discovery via Semantic Matching           │
└─────────────────────────────────────────────────────────────────────────┘

Step 1: NGO Profile Indexing (Background Process)
├─ When NGO creates/updates profile with impact narrative
├─ Lambda: indexNGOProfileHandler triggered
├─ Sends impact narrative to Bedrock for embedding generation:
│   Input: "We work on water conservation in drought-prone villages of 
│           Maharashtra. Implemented rainwater harvesting in 50 villages..."
│   Output: 1536-dimensional vector [0.023, -0.145, 0.089, ...]
├─ Stores embedding in OpenSearch vector index
└─ Updates ngos.embedding column in Aurora

Step 2: CSR Investor Search Query
├─ CSR user enters: "Find water conservation projects in drought-prone Maharashtra"
├─ Web dashboard sends to API: POST /api/search/semantic
└─ Lambda: semanticSearchHandler receives query

Step 3: Query Embedding Generation
├─ Lambda sends query to Bedrock for embedding
├─ Bedrock returns query vector (1536 dimensions)
└─ Processing time: ~200ms

Step 4: Vector Similarity Search (OpenSearch k-NN)
├─ Lambda queries OpenSearch with k-NN algorithm:
    {
      "query": {
        "knn": {
          "embedding": {
            "vector": [query_vector],
            "k": 20
          }
        }
      }
    }
├─ OpenSearch returns top 20 NGOs with similarity scores
├─ Example: NGO_A (score: 0.89), NGO_B (score: 0.82), ...
└─ Search time: ~300ms for 10,000+ NGO profiles

Step 5: Result Enrichment & Filtering
├─ Lambda fetches full NGO details from Aurora (batch query)
├─ Applies business filters:
│   - verification_status = 'verified'
│   - geographic_focus overlaps with query
│   - active projects exist
├─ Calculates predicted SROI using ML model
└─ Ranks results by combined score (semantic + SROI + recency)

Step 6: Response Caching & Delivery
├─ Lambda caches results in ElastiCache (TTL: 1 hour)
├─ Returns JSON with NGO profiles, relevance scores, match explanations
├─ Total response time: ~800ms (within 2-second SLA)
└─ Dashboard renders results with "Express Interest" buttons
```



### Workflow 3: Voice-First Data Entry

```
┌─────────────────────────────────────────────────────────────────────────┐
│           Vernacular Voice Input to Structured Data Storage              │
└─────────────────────────────────────────────────────────────────────────┘

Step 1: Voice Recording
├─ NGO user taps microphone icon in mobile app
├─ Records audio in regional language (e.g., Marathi)
├─ Audio saved locally as .m4a file (<10MB)
└─ Uploads to S3 via pre-signed URL

Step 2: Speech-to-Text (Amazon Transcribe)
├─ S3 event triggers Lambda: transcribeAudioHandler
├─ Lambda starts Transcribe job with parameters:
│   - LanguageCode: "mr-IN" (Marathi)
│   - VocabularyName: "ngo-domain-terms" (custom vocabulary)
│   - OutputBucketName: "impactledger-transcriptions"
├─ Transcribe processes audio (streaming or batch)
├─ Returns transcription:
│   Original: "आम्ही गेल्या महिन्यात पन्नास कुटुंबांना स्वच्छ पाणी पुरवले"
│   Confidence: 0.92
└─ Processing time: ~2-3 seconds for 30-second audio

Step 3: Translation (Optional - AWS Translate)
├─ If user prefers English output, Lambda calls Translate API
├─ Translates Marathi to English:
│   "We provided clean water to fifty families last month"
└─ Processing time: ~500ms

Step 4: Structured Data Extraction (Bedrock)
├─ Lambda sends transcription to Bedrock with context-aware prompt:
│   "Extract structured data from this NGO impact update.
│    Return JSON with: activity, beneficiary_count, timeframe, location"
├─ Bedrock returns:
    {
      "activity": "Clean water provision",
      "beneficiary_count": 50,
      "beneficiary_type": "families",
      "timeframe": "last month",
      "location": null,
      "confidence": 0.88
    }
└─ Processing time: ~1-2 seconds

Step 5: User Confirmation & Storage
├─ Mobile app displays transcription and extracted data
├─ User confirms or edits
├─ Lambda stores in Aurora (impact_reports or project updates)
└─ Updates NGO profile metrics (total_beneficiaries)

Step 6: Voice Feedback (Amazon Polly)
├─ System generates confirmation message in Marathi
├─ Lambda calls Polly SynthesizeSpeech API:
│   Input: "आपला अहवाल यशस्वीरित्या जतन झाला आहे"
│   VoiceId: "Aditi" (Indian female voice)
├─ Polly returns audio stream
├─ Mobile app plays confirmation
└─ Processing time: ~1 second
```



## Feasibility Analysis & Constraints

### Handling Intermittent Connectivity (Critical for Rural India)

**Challenge**: 2G/3G networks with frequent outages, high latency (500ms-2s), and low bandwidth (50-100 Kbps).

**Solution Architecture**:

1. **Offline-First Mobile App Design**
   - Local SQLite database for data persistence
   - React Native AsyncStorage for user preferences
   - Queue-based sync manager with exponential backoff
   - Differential sync (only changed data, not full profiles)

2. **Data Sync Strategy**
   ```
   Priority Queue:
   P0 (Critical): Authentication tokens, user profile
   P1 (High): Receipt uploads, compliance forms
   P2 (Medium): Impact reports, project updates
   P3 (Low): Media files (photos/videos)
   
   Sync Triggers:
   - Automatic: On connectivity restoration
   - Manual: User-initiated "Sync Now" button
   - Scheduled: Every 6 hours if connected
   ```

3. **Bandwidth Optimization**
   - Image compression: 5MB → 500KB (90% reduction) using React Native Image Resizer
   - Progressive image loading: Thumbnail first, full resolution on demand
   - API response compression: gzip (70% size reduction)
   - Lazy loading: Fetch only visible data, paginate results

4. **Conflict Resolution**
   - Last-write-wins for profile updates (with timestamp comparison)
   - Server-side merge for receipt uploads (append-only)
   - User prompt for conflicting edits (rare case)

5. **Offline Capabilities**
   - View cached NGO profiles and project details
   - Capture receipts and queue for upload
   - Record voice notes locally
   - Draft compliance forms (submit when online)

6. **Network Resilience**
   - Retry logic: 3 attempts with exponential backoff (1s, 2s, 4s)
   - Timeout configuration: 30s for API calls, 60s for file uploads
   - Fallback to SMS for critical notifications (OTP, funding alerts)



### Cost Optimization for Hackathon & MVP

**Estimated Monthly AWS Costs (1,000 NGOs, 50 CSR Investors)**:

| Service | Usage | Cost (USD) |
|---------|-------|------------|
| Lambda | 5M requests, 512MB, 3s avg | $25 |
| API Gateway | 5M requests | $17.50 |
| Aurora Serverless v2 | 2 ACUs avg, 50GB storage | $87 |
| S3 | 500GB storage, 10K uploads | $12 |
| Textract | 10K pages/month | $15 |
| Bedrock (Claude Haiku) | 50M input tokens, 5M output | $62.50 |
| OpenSearch | t3.small.search (2 nodes) | $60 |
| Transcribe | 1,000 hours audio | $24 |
| Polly | 5M characters | $20 |
| CloudFront | 100GB transfer | $8.50 |
| ElastiCache | cache.t3.micro | $12 |
| Cognito | 10K MAU | $27.50 |
| **Total** | | **~$371/month** |

**Cost Optimization Strategies**:
1. Use Lambda free tier (1M requests/month)
2. Aurora Serverless v2 scales to 0.5 ACU during low usage
3. S3 Intelligent-Tiering moves old receipts to Glacier (90% savings)
4. CloudFront caching reduces origin requests by 80%
5. Bedrock Haiku is 10x cheaper than Sonnet for structured tasks
6. OpenSearch t3.small sufficient for MVP (upgrade to r6g for production)



### Security & Compliance Architecture

**DPDP Act Compliance**:
1. **Data Localization**: All data stored in AWS ap-south-1 (Mumbai) region
2. **Consent Management**: Explicit opt-in for data collection, stored in users.consent_data JSONB
3. **Right to Erasure**: Automated data deletion workflow within 30 days
4. **Data Minimization**: Collect only essential fields, anonymize beneficiary PII
5. **Audit Trails**: 7-year retention in audit_logs table

**Security Controls**:
1. **Authentication**: AWS Cognito with MFA for CSR investors, OTP for NGOs
2. **Authorization**: IAM roles with least privilege, Lambda execution roles scoped per function
3. **Encryption**:
   - At rest: S3 SSE-KMS, Aurora encryption, EBS encryption
   - In transit: TLS 1.3, API Gateway with custom domain + ACM certificate
4. **Network Security**:
   - Lambda in VPC with private subnets
   - Aurora in isolated subnet, no public access
   - Security groups: Allow only necessary ports (443 for HTTPS)
5. **API Security**:
   - Rate limiting: 100 req/min per user, 1000 req/min per IP
   - Input validation: JSON schema validation, SQL injection prevention
   - AWS WAF: Block common exploits (OWASP Top 10)
6. **Secrets Management**: AWS Secrets Manager for API keys, database credentials
7. **Monitoring**: GuardDuty for threat detection, CloudTrail for API auditing



### Scalability & Performance

**Horizontal Scaling Strategy**:
1. **Compute**: Lambda auto-scales to 1,000 concurrent executions (soft limit, can increase)
2. **Database**: Aurora Serverless v2 scales from 0.5 to 128 ACUs automatically
3. **Search**: OpenSearch scales horizontally by adding data nodes
4. **Caching**: ElastiCache Redis cluster mode for distributed caching

**Performance Optimizations**:
1. **API Response Time**:
   - Target: p95 < 500ms, p99 < 1s
   - ElastiCache for frequently accessed data (NGO profiles, search results)
   - Database connection pooling (RDS Proxy)
   - Async processing for non-critical tasks (email notifications)

2. **AI Processing Time**:
   - Textract: 2-3s for standard receipts (AWS SLA)
   - Bedrock Haiku: 1-2s for structured extraction (optimized for speed)
   - OpenSearch k-NN: <500ms for 10K vectors (with HNSW algorithm)
   - Total pipeline: <5s for receipt processing (meets requirement)

3. **Database Query Optimization**:
   - Indexes on foreign keys, status fields, timestamps
   - Materialized views for dashboard aggregations
   - Read replicas for analytics queries (separate from transactional load)

4. **CDN & Edge Optimization**:
   - CloudFront caches static assets (React app, images) at edge locations
   - API Gateway caching for GET endpoints (TTL: 5 minutes)
   - Lambda@Edge for request routing and header manipulation



## Future Roadmap & Business Viability

### Phase 1: MVP & Pilot Launch (Months 1-3)

**Scope**: Core features for hackathon demonstration and initial pilot

**Features**:
- NGO registration and profile creation
- Receipt upload and OCR processing
- Basic Form 10BD generation
- CSR investor onboarding
- Semantic search (English only)
- Mobile app (Android only)
- Web dashboard for CSR investors

**Target**: 50 NGOs, 5 CSR investors in 3 districts (Pune, Satara, Solapur)

**Success Metrics**:
- 80% reduction in Form 10BD creation time
- 90% OCR accuracy on Hindi/English receipts
- 85% user satisfaction (SUS score)
- 10 successful NGO-CSR matches

**Technical Milestones**:
- Deploy serverless backend on AWS
- Integrate Textract + Bedrock pipeline
- Implement offline-first mobile app
- Set up CI/CD pipeline



### Phase 2: Scale & Advanced Features (Months 4-6)

**Scope**: Expand to 10 districts, add advanced AI features

**New Features**:
1. **Voice Interface**: Hindi, Marathi, Tamil support via Polly/Transcribe
2. **Predictive SROI**: ML model trained on historical funding data
3. **Multi-language Support**: 5 regional languages for mobile app
4. **iOS App**: React Native iOS build
5. **Advanced Analytics**: CSR dashboard with impact visualizations
6. **Automated Compliance Reminders**: Deadline tracking and notifications
7. **Document Verification**: Integration with government APIs (MCA, Income Tax)

**Target**: 500 NGOs, 25 CSR investors across Maharashtra

**Success Metrics**:
- ₹5 crore CSR funding facilitated
- 50,000 beneficiaries reached through funded projects
- 90% voice transcription accuracy
- 70% SROI prediction accuracy

**Technical Enhancements**:
- OpenSearch vector index optimization (HNSW algorithm)
- Aurora read replicas for analytics
- CloudWatch custom metrics and alarms
- Automated testing (80% code coverage)



### Phase 3: Blockchain & Government Integration (Months 7-12)

**Scope**: Enterprise-grade transparency and regulatory integration

**New Features**:
1. **Blockchain for Fund Transparency**:
   - Amazon Managed Blockchain (Hyperledger Fabric)
   - Immutable ledger for all funding transactions
   - Smart contracts for milestone-based disbursements
   - Public transparency dashboard (anonymized data)

2. **Government API Integration**:
   - MCA (Ministry of Corporate Affairs) for CSR reporting
   - Income Tax Department for Form 10BD/10BE submission
   - NGO Darpan for NGO verification
   - NITI Aayog for SDG impact mapping

3. **Advanced AI Features**:
   - Fraud detection using anomaly detection (SageMaker)
   - Impact prediction using time-series forecasting
   - Automated due diligence reports
   - Multilingual chatbot for NGO support (Lex)

4. **Marketplace Features**:
   - NGO rating and review system
   - CSR project marketplace (browse and fund)
   - Peer-to-peer NGO collaboration
   - Impact investing for HNIs

**Target**: 5,000 NGOs, 200 CSR investors, pan-India coverage

**Success Metrics**:
- ₹50 crore CSR funding facilitated
- 500,000 beneficiaries reached
- 95% compliance form acceptance rate
- Break-even on platform operations

**Technical Architecture**:
- Multi-region deployment (Mumbai + Hyderabad)
- Disaster recovery with RTO < 4 hours
- 99.99% uptime SLA
- SOC 2 Type II compliance



### Business Model & Revenue Streams

**Freemium Model for NGOs**:
- Free tier: Basic profile, 10 receipt uploads/month, 1 Form 10BD/year
- Pro tier (₹999/month): Unlimited uploads, advanced analytics, priority support
- Enterprise tier (₹4,999/month): Multi-user accounts, API access, white-label reports

**Subscription Model for CSR Investors**:
- Starter (₹9,999/month): Search 100 NGOs, 5 active projects, basic dashboard
- Professional (₹29,999/month): Unlimited search, 20 projects, advanced analytics, API access
- Enterprise (₹99,999/month): Custom integrations, dedicated account manager, compliance automation

**Transaction Fees**:
- 1% platform fee on successful funding transactions (capped at ₹50,000 per transaction)
- Example: ₹1 crore funding → ₹1 lakh platform fee

**Value-Added Services**:
- Compliance consulting: ₹15,000 per Form 10BD review
- Impact assessment reports: ₹50,000 per project
- NGO capacity building workshops: ₹25,000 per session
- Custom integrations for corporates: ₹2-5 lakhs per project

**Revenue Projections (Year 1)**:
- NGO subscriptions: 100 Pro users × ₹999 × 12 = ₹11.99 lakhs
- CSR subscriptions: 20 Professional × ₹29,999 × 12 = ₹71.99 lakhs
- Transaction fees: ₹10 crore funding × 1% = ₹10 lakhs
- Services: ₹20 lakhs
- **Total: ₹1.14 crores**

**Break-even Analysis**:
- Fixed costs: ₹40 lakhs/year (AWS, salaries, operations)
- Variable costs: ₹30 lakhs/year (support, marketing)
- Break-even: Month 18 at current growth rate



## Deployment Architecture

### Infrastructure as Code (IaC)

**AWS CDK (TypeScript)**:
```typescript
// Example CDK stack structure
class ImpactLedgerStack extends Stack {
  constructor(scope: Construct, id: string) {
    // VPC with public/private subnets
    const vpc = new ec2.Vpc(this, 'ImpactLedgerVPC', {
      maxAzs: 2,
      natGateways: 1
    });

    // Aurora Serverless v2 cluster
    const dbCluster = new rds.DatabaseCluster(this, 'AuroraCluster', {
      engine: rds.DatabaseClusterEngine.auroraPostgres({
        version: rds.AuroraPostgresEngineVersion.VER_15_3
      }),
      serverlessV2MinCapacity: 0.5,
      serverlessV2MaxCapacity: 16,
      vpc,
      vpcSubnets: { subnetType: ec2.SubnetType.PRIVATE_ISOLATED }
    });

    // Lambda functions
    const apiLambda = new lambda.Function(this, 'ApiHandler', {
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('dist/api'),
      environment: {
        DB_HOST: dbCluster.clusterEndpoint.hostname,
        BEDROCK_REGION: 'us-east-1'
      },
      vpc
    });

    // API Gateway
    const api = new apigateway.RestApi(this, 'ImpactLedgerAPI', {
      deployOptions: {
        throttlingRateLimit: 1000,
        throttlingBurstLimit: 2000
      }
    });

    // S3 buckets
    const receiptsBucket = new s3.Bucket(this, 'ReceiptsBucket', {
      encryption: s3.BucketEncryption.KMS,
      lifecycleRules: [{
        transitions: [{
          storageClass: s3.StorageClass.INTELLIGENT_TIERING,
          transitionAfter: Duration.days(30)
        }]
      }]
    });

    // OpenSearch domain
    const searchDomain = new opensearch.Domain(this, 'SearchDomain', {
      version: opensearch.EngineVersion.OPENSEARCH_2_11,
      capacity: {
        dataNodes: 2,
        dataNodeInstanceType: 't3.small.search'
      },
      vpc
    });
  }
}
```



### CI/CD Pipeline

**AWS CodePipeline Stages**:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          CI/CD Pipeline                                  │
└─────────────────────────────────────────────────────────────────────────┘

Stage 1: Source
├─ GitHub repository (main branch)
├─ Webhook triggers on push/PR merge
└─ CodePipeline fetches latest code

Stage 2: Build (CodeBuild)
├─ Install dependencies (npm install)
├─ Run linting (ESLint)
├─ Run unit tests (Jest)
├─ Build Lambda functions (TypeScript → JavaScript)
├─ Build React Native app (Android APK)
├─ Build React.js dashboard (production bundle)
└─ Upload artifacts to S3

Stage 3: Test (CodeBuild)
├─ Integration tests against staging environment
├─ API contract tests (Postman/Newman)
├─ Security scanning (OWASP Dependency Check)
├─ Performance tests (Artillery.io)
└─ Generate test reports

Stage 4: Deploy to Staging
├─ CDK deploy to staging account
├─ Database migrations (Flyway)
├─ Lambda function updates
├─ API Gateway deployment
├─ CloudFront cache invalidation
└─ Smoke tests

Stage 5: Manual Approval
├─ Slack notification to team
├─ Review staging environment
└─ Approve/Reject deployment

Stage 6: Deploy to Production
├─ Blue-green deployment strategy
├─ CDK deploy to production account
├─ Gradual traffic shift (10% → 50% → 100%)
├─ CloudWatch alarms monitoring
└─ Automatic rollback on errors

Stage 7: Post-Deployment
├─ Synthetic monitoring (CloudWatch Synthetics)
├─ Performance metrics collection
├─ Slack notification: Deployment successful
└─ Update deployment dashboard
```



### Monitoring & Observability

**CloudWatch Dashboards**:
1. **Application Metrics**:
   - API request count, latency (p50, p95, p99)
   - Lambda invocations, errors, duration
   - Database connections, query performance
   - Cache hit/miss ratio

2. **Business Metrics**:
   - NGO registrations per day
   - Receipt uploads per hour
   - CSR searches per day
   - Funding transactions per week

3. **AI Metrics**:
   - Textract processing time
   - Bedrock token usage and cost
   - OpenSearch query latency
   - Voice transcription accuracy

**Alarms**:
- API error rate > 1% → PagerDuty alert
- Lambda duration > 10s → Slack notification
- Database CPU > 80% → Auto-scaling trigger
- S3 upload failures > 5% → Email alert

**Distributed Tracing (X-Ray)**:
- End-to-end request tracing
- Identify bottlenecks in AI pipeline
- Service map visualization
- Anomaly detection

**Log Aggregation**:
- CloudWatch Logs for all Lambda functions
- Structured JSON logging
- Log retention: 30 days (hot), 7 years (cold in S3)
- Log Insights queries for debugging



## Risk Mitigation Strategies

### Technical Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| AI hallucination in Form 10BD | High | Medium | Human review for confidence < 85%, audit trail, legal disclaimer |
| OpenSearch scaling issues | Medium | Low | Start with t3.small, monitor query latency, upgrade to r6g if needed |
| Lambda cold starts | Medium | Medium | Provisioned concurrency for critical functions, keep functions warm |
| Data loss | High | Low | Automated backups every 6 hours, point-in-time recovery, multi-AZ deployment |
| Security breach | High | Low | WAF, GuardDuty, penetration testing, bug bounty program |
| Vendor lock-in (AWS) | Medium | Low | Use open standards (PostgreSQL, OpenSearch), containerize Lambdas |

### Business Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Low NGO adoption | High | Medium | Freemium model, vernacular support, offline-first design, field training |
| CSR investor skepticism | High | Medium | Pilot with 5 corporates, case studies, third-party audits, compliance guarantees |
| Regulatory changes | Medium | Medium | Legal advisory board, automated compliance updates, government partnerships |
| Competition | Medium | High | AI differentiation, network effects, first-mover advantage in rural segment |
| Funding constraints | High | Low | Phased rollout, cost optimization, revenue from Phase 1, CSR grants |

### Operational Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Support scalability | Medium | High | Chatbot for common queries, knowledge base, tiered support (free vs paid) |
| Data quality issues | Medium | Medium | Validation rules, manual review queue, user feedback loop |
| Language accuracy | Medium | Medium | Custom vocabularies, user corrections, continuous model improvement |
| Infrastructure costs | Medium | Low | Serverless auto-scaling, cost monitoring, budget alerts, reserved capacity |



## Hackathon Judging Criteria Alignment

### 1. Technical Aptness (30%)

**Strengths**:
- **Modern Architecture**: Serverless, microservices, event-driven design
- **AWS Best Practices**: Well-architected framework (security, reliability, performance, cost optimization)
- **Scalability**: Auto-scaling at every layer (Lambda, Aurora, OpenSearch)
- **AI Integration**: Multi-service AI pipeline (Textract → Bedrock → OpenSearch)
- **Infrastructure as Code**: CDK for reproducible deployments
- **CI/CD**: Automated testing and deployment pipeline

**Demonstration Points**:
- Live demo of receipt upload → Form 10BD generation in <10 seconds
- Semantic search returning relevant NGOs in <2 seconds
- Voice interface transcribing Marathi audio in real-time
- Architecture diagram showing AWS service integration
- Code walkthrough of AI processing pipeline

### 2. Innovation (25%)

**Unique Differentiators**:
- **AI-First Compliance**: First platform to automate Form 10BD using Generative AI
- **Semantic Discovery**: Vector search for unstructured NGO narratives (not just keyword matching)
- **Voice-First for Rural Users**: Vernacular voice interface for low-literacy users
- **Offline-First Mobile**: Designed for 2G/3G networks with intermittent connectivity
- **Predictive Impact**: ML-based SROI prediction before funding

**Innovation Narrative**:
"ImpactLedger doesn't just digitize existing processes—it reimagines CSR funding using AI. We're not building a database; we're building an intelligent matchmaker that understands impact narratives and automates legal compliance."



### 3. Business Feasibility (25%)

**Market Opportunity**:
- ₹25,000 crore annual CSR spending in India (Companies Act 2013 mandate)
- 3.3 million registered NGOs, but only 10% receive CSR funding
- Rural NGOs spend ₹20,000-30,000 per year on compliance consultants
- Corporates spend 60-90 days on due diligence per NGO

**Revenue Model**:
- Freemium for NGOs (₹999-4,999/month for premium)
- Subscription for CSR investors (₹9,999-99,999/month)
- 1% transaction fee on successful funding
- Value-added services (consulting, reports)

**Go-to-Market**:
- Phase 1: Pilot with 5 corporates in Maharashtra (Months 1-3)
- Phase 2: Expand to 10 districts, 500 NGOs (Months 4-6)
- Phase 3: Pan-India, government partnerships (Months 7-12)

**Competitive Advantage**:
- AI differentiation (compliance automation, semantic search)
- Rural focus (offline-first, vernacular support)
- Network effects (more NGOs → more CSR investors → more NGOs)
- First-mover advantage in AI-powered CSR space

**Financial Projections**:
- Year 1: ₹1.14 crores revenue, ₹70 lakhs costs → ₹44 lakhs profit
- Year 2: ₹5 crores revenue (5,000 NGOs, 200 investors)
- Break-even: Month 18



### 4. Social Impact (20%)

**Problem Solved**:
- **For NGOs**: Reduce compliance burden by 80%, increase funding access by 3x
- **For Corporates**: Reduce due diligence time by 50%, discover high-impact grassroots projects
- **For Beneficiaries**: Faster funding → faster impact (water, education, health)

**Impact Metrics**:
- Year 1: ₹10 crore CSR funding facilitated → 100,000 beneficiaries reached
- Year 2: ₹50 crore funding → 500,000 beneficiaries
- Year 3: ₹200 crore funding → 2 million beneficiaries

**SDG Alignment**:
- SDG 1 (No Poverty): Enable funding for poverty alleviation projects
- SDG 4 (Quality Education): Support education NGOs
- SDG 6 (Clean Water): Fund water conservation projects
- SDG 17 (Partnerships): Bridge NGO-corporate gap

**Social Innovation**:
- Democratize access to CSR funding for rural NGOs
- Reduce information asymmetry between NGOs and corporates
- Increase transparency and accountability in CSR spending
- Empower low-literacy users through voice interface

## Conclusion

ImpactLedger represents a paradigm shift in CSR funding—from manual, opaque processes to AI-powered, transparent matchmaking. By leveraging AWS's comprehensive AI/ML portfolio, we deliver enterprise-grade compliance automation and semantic discovery while maintaining accessibility for rural, low-bandwidth users.

The architecture is designed for hackathon demonstration (MVP in 3 months) while being production-ready for scale (5,000 NGOs, 200 CSR investors). Our phased roadmap balances technical innovation with business viability, ensuring sustainable growth and measurable social impact.

**Key Takeaways**:
1. **AI is the differentiator**: Not a feature, but the core value proposition
2. **Offline-first design**: Critical for rural India's connectivity constraints
3. **Serverless architecture**: Cost-efficient, auto-scaling, production-ready
4. **Business model**: Freemium + subscriptions + transaction fees = sustainable revenue
5. **Social impact**: ₹10 crore funding facilitated in Year 1, 100,000 beneficiaries reached

