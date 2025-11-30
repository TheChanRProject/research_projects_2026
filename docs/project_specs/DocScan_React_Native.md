Technical Specification: DocScan AI – Vision-Powered Document Intelligence Platform

### Resources

- [ChatGPT Thread](https://chatgpt.com/share/692c3446-b664-8008-8b76-f3e9c621ebba)
- [doscan_ai_react_native - GitHub Repo](https://github.com/TheChanRProject/docscan_ai_react_native)

Executive Summary

This document specifies DocScan AI, a cross-platform mobile and backend platform for intelligent document and credit card scanning. The system uses a React Native mobile app to capture images, a FastAPI backend for orchestration, Gemini 3 as a vision language model (VLM), Claude 4.5 for structured output, LangChain for model coordination, MongoDB for persistence, and Azure Blob Storage for secure image storage.

The platform allows users to scan documents or cards, define what entities they care about (e.g., invoice_number, due_date, total_amount), and receive structured JSON outputs that can be inspected, edited, and integrated into downstream systems. It is designed for extensible AI pipelines, robust observability, and future enterprise integrations.

⸻

1. Product Vision and Scope

1.1 Problem Statement

Manual extraction of structured data from documents, invoices, receipts, and card images is:
	•	Error-prone and slow.
	•	Difficult to customize per workflow (different entities for different document types).
	•	Often locked into rigid OCR templates that fail on layout variations.

Organizations and individual power users need a flexible, LLM-native way to convert image-based documents into structured data, with:
	•	Minimal integration friction.
	•	Mobile-first capture.
	•	Custom entity definitions on-the-fly.

1.2 Solution Overview

DocScan AI provides:
	•	A React Native mobile app for capturing document/credit card images and defining extraction schemas (entity names).
	•	A FastAPI backend that:
	•	Stores images in Azure Blob Storage.
	•	Uses LangChain to orchestrate:
	•	Gemini 3 for vision-based understanding of the image.
	•	Claude 4.5 for strongly-typed, schema-conforming structured output.
	•	Persists user, document, and extraction data in MongoDB.
	•	A dashboard and queue view in the app for monitoring and reviewing extraction results.

The system is API-first and can later be extended to web clients, third-party integrations, and workflow engines.

1.3 Target Users
	•	Individual professionals scanning invoices, receipts, IDs, or cards for side projects / personal finance.
	•	Small teams & startups looking to prototype document-intelligence workflows without building full OCR stacks.
	•	Developers & ML engineers who want a mobile capture front-end and a structured JSON backend they can pipe into automation tools and data pipelines.

1.4 Key Differentiators

DocScan AI differentiates itself by:
	•	Allowing users to define entity names directly in the app, as opposed to rigid pre-defined templates.
	•	Leveraging a VLM + LLM pipeline (Gemini 3 + Claude 4.5) for layout-aware understanding and strict schema outputs.
	•	Being mobile-first and cross-platform via React Native.
	•	Using a clean, Plaid-inspired UX theme for clarity, trust, and readability.
	•	Providing an API-ready backend for easy integration into custom systems.

⸻

2. System Architecture

2.1 High-Level Architecture

The system is composed of a mobile client layer, API and AI orchestration layer, and storage layer.

┌─────────────────────────────────────────────────────────────────────────────┐
│                             CLIENT LAYER                                   │
│                                                                             │
│  ┌──────────────────────┐         ┌─────────────────────────────┐          │
│  │  Mobile App          │         │  Future Clients (Web, CLI) │          │
│  │  (React Native, iOS/ │         │  - Web Dashboard           │          │
│  │   Android)           │         │  - API Integrations        │          │
│  └─────────┬────────────┘         └─────────────┬──────────────┘          │
└────────────┼────────────────────────────────────┼──────────────────────────┘
             │                                    │
             └────────────────────────────────────┘
                              HTTPS (REST JSON)
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           API & ORCHESTRATION LAYER                        │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │ FastAPI Backend                                                      │   │
│  │                                                                      │   │
│  │  • Auth & User Management (JWT)                                      │   │
│  │  • Document Upload (multipart/form-data)                             │   │
│  │  • Stats & Dashboard Endpoints                                       │   │
│  │  • Extraction Retrieval & Update                                     │   │
│  │                                                                      │   │
│  │        ┌────────────────────────────┐                                 │   │
│  │        │  LangChain Orchestrator   │                                 │   │
│  │        │                            │                                 │   │
│  │        │  ┌───────────────┐        │   ┌──────────────────────────┐   │   │
│  │        │  │ Gemini 3 VLM  │◀───────┼──▶│ Image + Prompt           │   │   │
│  │        │  └───────────────┘        │   │ (Extraction Candidates)  │   │   │
│  │        │                            │   └──────────────────────────┘   │   │
│  │        │  ┌───────────────┐        │   ┌──────────────────────────┐   │   │
│  │        │  │ Claude 4.5    │◀───────┼──▶│ Structured JSON Output   │   │   │
│  │        │  └───────────────┘        │   └──────────────────────────┘   │   │
│  │        └────────────────────────────┘                                 │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                        ┌────────────┴────────────┐
                        ▼                         ▼
┌────────────────────────────────────────┐  ┌────────────────────────────────┐
│     STORAGE: MongoDB                  │  │   STORAGE: Azure Blob Storage  │
│  • users                              │  │  • Raw document images         │
│  • documents                          │  │  • Thumbnails (optional)       │
│  • extractions                        │  │                                │
└────────────────────────────────────────┘  └────────────────────────────────┘

2.2 Component Descriptions

Mobile App (React Native)
Provides the user interface, including:
	•	Authentication screens (Login / Signup).
	•	Dashboard showing scan statistics and recent documents.
	•	New Scan flow:
	•	Entity definition step.
	•	Camera capture or gallery upload.
	•	Queue view for extraction jobs.
	•	Detail view for structured extraction results.

API Service (FastAPI)
	•	Exposes REST endpoints for:
	•	Authentication and user lifecycle.
	•	Document upload and job creation.
	•	Document listing (queue/history).
	•	Extraction retrieval and updates.
	•	Stats and dashboard metrics.
	•	Handles input validation via Pydantic.
	•	Issues and validates JWT tokens.

LangChain Orchestrator
	•	Coordinates calls to:
	•	Gemini 3 for image understanding.
	•	Claude 4.5 for schema-conforming structured JSON.
	•	Applies prompt templates and dynamic schemas based on user-defined entity lists.
	•	Consolidates raw VLM output and structured result.

MongoDB
	•	Primary data store for:
	•	User accounts.
	•	Document metadata (status, requested entities, blob URLs).
	•	Extraction results (structured output & confidence scores).
	•	Designed for flexible JSON-like storage and rapid iteration.

Azure Blob Storage
	•	Stores raw image files in a dedicated container, e.g., docscan-images.
	•	Accessed by backend for VLM processing; no direct public listing.
	•	Blob naming includes user ID and document type for organization.

⸻

3. Model Orchestration and AI Workflow Design

3.1 VLM + LLM Pipeline

The AI workflow is a two-stage pipeline orchestrated via LangChain.
	1.	Vision Extraction (Gemini 3)
	•	Input:
	•	Image bytes from Azure Blob.
	•	Entity list (e.g., ["invoice_number", "due_date", "total_amount"]).
	•	Prompt:
	•	Instructs Gemini 3 to:
	•	Understand the document layout.
	•	Identify candidate values for each entity.
	•	Provide a JSON response with approximate values and rationales.
	2.	Structured Output Normalization (Claude 4.5)
	•	Input:
	•	Gemini’s raw JSON output.
	•	A dynamically generated Pydantic schema representing requested entities and expected types (e.g., str, float, datetime).
	•	Behavior:
	•	Rewrites Gemini output into strict JSON conforming to the schema.
	•	Resolves ambiguities (e.g., date formats).
	•	Outputs:
	•	structured_output object (entity→value).
	•	Optional confidence_scores.

3.2 Orchestration Patterns

Sequential Pipeline
	•	upload_document → store_blob → call_gemini → call_claude → persist_extraction.
	•	Synchronous in v1 for simplicity; can be refactored to async + background workers.

Retry and Error Handling
	•	Gemini and Claude calls are wrapped in retry logic:
	•	Backoff for transient failures (rate limits, network issues).
	•	Store error messages in documents.error_message if persistent.
	•	If any stage fails:
	•	documents.status → "failed".
	•	extractions record is not created (or flagged with partial error state).

⸻

4. Data Model

4.1 Core Entities (MongoDB)

┌─────────────────────────────────────────────────────────────────────┐
│                               USER                                 │
├─────────────────────────────────────────────────────────────────────┤
│ _id: ObjectId                                                      │
│ email: string (unique)                                             │
│ password_hash: string                                              │
│ created_at: datetime                                               │
│ updated_at: datetime                                               │
│ last_login_at: datetime | null                                     │
└─────────────────────────────────────────────────────────────────────┘
                               1 : N
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│                             DOCUMENT                               │
├─────────────────────────────────────────────────────────────────────┤
│ _id: ObjectId                                                      │
│ user_id: ObjectId (FK → USER)                                      │
│ type: "document" | "credit_card"                                   │
│ status: "queued" | "processing" | "completed" | "failed"           │
│ requested_entities: [string]                                       │
│ azure_blob_url: string                                             │
│ thumbnail_blob_url: string | null                                  │
│ extraction_id: ObjectId | null (FK → EXTRACTION)                   │
│ error_message: string | null                                       │
│ created_at: datetime                                               │
│ updated_at: datetime                                               │
└─────────────────────────────────────────────────────────────────────┘
                                 1 : 1
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────────┐
│                            EXTRACTION                              │
├─────────────────────────────────────────────────────────────────────┤
│ _id: ObjectId                                                      │
│ document_id: ObjectId (FK → DOCUMENT)                              │
│ user_id: ObjectId (FK → USER)                                      │
│ model_pipeline: {                                                  │
│   vision_model: "Gemini 3",                                        │
│   structuring_model: "Claude 4.5",                                 │
│   orchestrator: "LangChain",                                       │
│   version: "v1"                                                    │
│ }                                                                  │
│ requested_entities: [string]                                       │
│ raw_vlm_output: string (JSON string, raw from Gemini)              │
│ structured_output: { [entity: string]: any }                       │
│ confidence_scores: { [entity: string]: number } | null             │
│ created_at: datetime                                               │
│ updated_at: datetime                                               │
└─────────────────────────────────────────────────────────────────────┘

4.2 Indexing Strategy
	•	users:
	•	email – unique index.
	•	documents:
	•	{ user_id: 1, created_at: -1 } – listing & history.
	•	{ status: 1, user_id: 1 } – queues.
	•	extractions:
	•	{ document_id: 1 }.
	•	{ user_id: 1, created_at: -1 }.

⸻

5. API Specification (FastAPI)

5.1 Authentication Endpoints

POST /auth/signup
	•	Request:

{
  "email": "user@example.com",
  "password": "plaintextPassword"
}


	•	Response:

{
  "user_id": "string",
  "access_token": "jwt",
  "token_type": "bearer"
}



POST /auth/login
	•	Request:

{
  "email": "user@example.com",
  "password": "plaintextPassword"
}


	•	Response:

{
  "user_id": "string",
  "access_token": "jwt",
  "token_type": "bearer"
}



5.2 Document & Extraction Endpoints

POST /documents – Create scan job & upload image
	•	Auth: Bearer token.
	•	Request: multipart/form-data
	•	file: image binary (JPEG/PNG).
	•	type: "document" | "credit_card".
	•	entities: JSON array as string, e.g. ["invoice_number","due_date"].
	•	Behavior:
	•	Upload image to Azure Blob.
	•	Create documents record (status "processing").
	•	Invoke AI pipeline (Gemini → Claude) synchronously in v1.
	•	Create extractions record, update documents.status & extraction_id.
	•	Response (v1, synchronous):

{
  "document_id": "string",
  "status": "completed",
  "extraction_id": "string"
}



(For a future async mode, status could be "queued" and extraction done via background jobs.)

⸻

GET /documents – List documents
	•	Auth: Bearer token.
	•	Query params:
	•	status (optional): "queued" | "processing" | "completed" | "failed".
	•	page (default: 1).
	•	page_size (default: 20).
	•	Response:

{
  "items": [
    {
      "document_id": "string",
      "type": "document",
      "status": "completed",
      "created_at": "2025-11-21T12:34:56Z",
      "thumbnail_url": "string or null"
    }
  ],
  "page": 1,
  "page_size": 20,
  "total": 42
}



⸻

GET /documents/{document_id} – Get document metadata
	•	Auth: Bearer token.
	•	Response:

{
  "document_id": "string",
  "type": "document",
  "status": "completed",
  "requested_entities": ["invoice_number","due_date"],
  "azure_blob_url": "string (optionally signed or masked)",
  "thumbnail_url": "string or null",
  "extraction_id": "string or null",
  "error_message": "string or null",
  "created_at": "2025-11-21T12:34:56Z",
  "updated_at": "2025-11-21T12:35:20Z"
}



⸻

GET /documents/{document_id}/extraction – Get structured output
	•	Auth: Bearer token.
	•	Response:

{
  "document_id": "string",
  "structured_output": {
    "invoice_number": "12345-AB",
    "due_date": "2025-01-31",
    "total_amount": 1250.5
  },
  "confidence_scores": {
    "invoice_number": 0.94,
    "due_date": 0.88,
    "total_amount": 0.91
  }
}



⸻

PATCH /documents/{document_id}/extraction – Update extracted fields
	•	Auth: Bearer token.
	•	Request:

{
  "structured_output": {
    "invoice_number": "12345-XYZ",
    "due_date": "2025-02-01"
  }
}


	•	Response:

{
  "document_id": "string",
  "structured_output": {
    "invoice_number": "12345-XYZ",
    "due_date": "2025-02-01",
    "total_amount": 1250.5
  }
}



⸻

5.3 Stats & Dashboard Endpoints

GET /stats/summary
	•	Auth: Bearer token.
	•	Response:

{
  "total_scanned": 150,
  "total_documents": 90,
  "total_credit_cards": 60,
  "completed": 145,
  "failed": 5
}



⸻

6. Security Architecture

6.1 Authentication and Authorization
	•	Auth Mechanism: JWT-based.
	•	Identity:
	•	Basic email/password auth for v1.
	•	Future extension: OAuth2/OIDC for SSO.
	•	Token Handling:
	•	Access tokens with limited TTL (e.g., 1 hour).
	•	Refresh tokens can be added later.

Permission Matrix (v1)

Action	Authenticated User
Create account	✓
Login	✓
Upload document / create job	✓
View own documents & extractions	✓
Modify own extraction	✓
View others’ data	✗

6.2 Data Security
	•	Transport: All client–server communication over HTTPS.
	•	At Rest:
	•	MongoDB configured with disk encryption at infrastructure level (Atlas / host).
	•	Azure Blob Storage uses server-side encryption (SSE).
	•	Sensitive Data Handling:
	•	Avoid logging raw images or full credit card numbers.
	•	Logs only include anonymized metadata (document IDs, user IDs, statuses).

6.3 Secret Management
	•	Environment variables stored in secrets management (e.g., Azure Key Vault or equivalent).
	•	Secrets:
	•	MONGO_URI
	•	JWT_SECRET_KEY
	•	GEMINI_API_KEY
	•	CLAUDE_API_KEY
	•	AZURE_STORAGE_CONNECTION_STRING

⸻

7. Observability and Monitoring

7.1 Metrics

Backend Metrics
	•	Request counts and latencies for:
	•	/auth/*
	•	/documents
	•	/stats/summary
	•	Error rates by endpoint (4xx vs 5xx).
	•	VLM pipeline metrics:
	•	Gemini call latency and failures.
	•	Claude call latency and failures.
	•	Average extraction duration.

Business Metrics
	•	Documents scanned per user per day.
	•	Completion vs failure rates.
	•	Average extraction time from upload to completed status.

7.2 Logging
	•	Structured JSON logs with:
	•	timestamp, level, endpoint, user_id, document_id (if applicable), status_code, latency_ms.
	•	Logs shipped to centralized service (e.g., Azure Log Analytics or ELK stack).

7.3 Tracing (Optional v2+)
	•	OpenTelemetry-based tracing for:
	•	API request span.
	•	Azure Blob upload span.
	•	Gemini and Claude spans inside LangChain pipeline.
	•	Allows debugging slowdowns and failures across the pipeline.

⸻

8. Scalability and Performance

8.1 Scaling Strategy
	•	FastAPI App:
	•	Stateless; can run multiple replicas behind a load balancer.
	•	Gunicorn/Uvicorn workers scaled up based on CPU/memory usage.
	•	MongoDB:
	•	Use MongoDB Atlas or similar with auto-scaling where possible.
	•	Azure Blob:
	•	Scales horizontally by design.

8.2 Performance Targets
	•	API response time (synchronous pipeline):
	•	p50: < 4 seconds.
	•	p95: < 8 seconds.
	•	Upload size:
	•	Recommended single-image upload < 10 MB; larger images compressed client-side.

8.3 Future Async Mode

To improve responsiveness, the pipeline can shift to:
	•	POST /documents returns "queued" quickly.
	•	Background worker processes:
	•	Use a task queue (e.g., Celery + Redis, or Azure Functions) to run the Gemini/Claude pipeline.
	•	Client polls /documents/{id} or receives push notifications (future).

⸻

9. Deployment Architecture

9.1 Environment Layout
	•	Development:
	•	FastAPI running locally (or in Docker).
	•	MongoDB in Docker container.
	•	Azure Blob Storage dev container or emulator.
	•	Staging:
	•	Deployed backend to a single cloud environment (e.g., Azure App Service).
	•	Connected to staging MongoDB cluster and Blob container.
	•	Production:
	•	FastAPI deployed in highly-available configuration (multiple instances).
	•	Managed MongoDB cluster (Atlas).
	•	Production Azure Blob container with appropriate lifecycle policies.

9.2 CI/CD

Pipeline steps:
	1.	Lint & Type Check (mypy / flake8 / ESLint / TS).
	2.	Unit Tests (backend + mobile where applicable).
	3.	Build Docker image (for backend).
	4.	Deploy to staging.
	5.	Smoke tests.
	6.	Promote to production on success.

⸻

10. Technology Stack Recommendation

Layer	Technology	Rationale
Mobile	React Native (TS)	Cross-platform, strong ecosystem
Mobile Nav	React Navigation	De facto standard, flexible
Backend API	FastAPI	Async, typed, fast, great DX
Orchestration	LangChain	Structured AI chains, multi-model support
Vision Model	Gemini 3	Strong multimodal capabilities
Structuring LLM	Claude 4.5	Strong reasoning, JSON-friendly
DB	MongoDB	Schema-flexible, JSON-centric
Object Storage	Azure Blob Storage	Scalable, cheap, direct SDK support
Auth	JWT	Simple, widely supported
Observability	OpenTelemetry (optional) + logs	Standardized tracing & logging


⸻

11. Mobile UI Specifications

11.1 Screens

Login Screen
	•	Elements:
	•	DocScan AI title (Plaid-style navy).
	•	Email, Password fields.
	•	“Login” primary button.
	•	“Create account” text button.
	•	API:
	•	POST /auth/login on submit.

Signup Screen
	•	Similar layout to Login.
	•	API:
	•	POST /auth/signup on submit.

⸻

Dashboard Screen
	•	Sections:
	•	Header: “Dashboard”.
	•	Stats row:
	•	Cards: Total scanned, Documents, Cards.
	•	Recent Scans:
	•	List of last N documents with status chip.
	•	APIs:
	•	GET /stats/summary.
	•	GET /documents?status=completed&page=1&page_size=5.

⸻

New Scan Screen

Two-step UX:
	1.	Entity Definition Step
	•	Segmented control: Document / Credit card.
	•	Dynamic entity list:
	•	Multiple “Entity name” text inputs (e.g., invoice_number, due_date).
	•	“+ Add entity” button.
	•	“Continue to camera” button.
	2.	Camera Step
	•	Full-screen camera preview.
	•	Capture button.
	•	Option for “Upload from gallery”.
	•	On capture:
	•	Show preview.
	•	“Submit for extraction” button → POST /documents.

⸻

Queue Screen
	•	List of documents in various statuses (queued, processing, completed, failed).
	•	Each item:
	•	Type (document/card).
	•	Created date.
	•	Status chip.
	•	Tap item → ScanDetailScreen.

API:
	•	GET /documents?status=<optional>&page=<...>.

⸻

Scan Detail Screen
	•	Header: Scan Detail.
	•	Shows:
	•	Document ID.
	•	Status.
	•	Extracted fields:
	•	Key/value list from structured_output.
	•	Actions:
	•	(v1) “Download JSON / Share” (share sheet).
	•	(v2) “Edit values” → PATCH endpoint.

API:
	•	GET /documents/{id}/extraction.
	•	PATCH /documents/{id}/extraction (future editing).

⸻

12. Implementation Roadmap

Phase 1: Core Backend & Data Model (Weeks 1–3)
	•	Set up FastAPI app skeleton.
	•	Implement MongoDB connection & collections (users, documents, extractions).
	•	Implement auth endpoints (/auth/signup, /auth/login) with password hashing + JWT.
	•	Implement basic /documents listing and /stats/summary using seeded data.

Phase 2: Mobile App Scaffold (Weeks 3–5)
	•	Initialize React Native project with TypeScript.
	•	Implement navigation:
	•	Auth Stack: Login / Signup.
	•	Main Tabs: Dashboard / New Scan / Queue.
	•	Stack: ScanDetail.
	•	Implement Plaid-style theme and base components (TextInputField, StatCard, StatusChip).
	•	Integrate with mock API client for local testing.

Phase 3: AI Pipeline & Storage Integration (Weeks 5–8)
	•	Integrate Azure Blob Storage:
	•	Upload image from FastAPI.
	•	Persist azure_blob_url in documents.
	•	Implement LangChain pipeline:
	•	Gemini 3 vision step.
	•	Claude 4.5 structured output step.
	•	Wire POST /documents to full pipeline.
	•	Implement GET /documents/{id}/extraction.

Phase 4: Mobile–Backend Integration (Weeks 8–10)
	•	Replace mock API in mobile app with real HTTP calls to FastAPI.
	•	Implement image upload via multipart/form-data from NewScanScreen.
	•	Implement queue and detail views using real backend data.
	•	Basic error/UI state handling (loading states, error messages).

Phase 5: Hardening & Observability (Weeks 10–12)
	•	Add structured logging and basic metrics.
	•	Implement retry logic and error handling in AI pipeline.
	•	Add minimal tracing around VLM calls.
	•	Conduct manual load tests and optimize for common cases.

⸻

13. Success Metrics

13.1 Product Metrics
	•	Onboarding: % of users who successfully complete their first scan within 10 minutes of signup.
	•	Usage: Average number of documents scanned per active user per week.
	•	Accuracy proxy: Manual correction rate (how often users modify fields after extraction).

13.2 Technical Metrics
	•	Reliability:
	•	Uptime: ≥ 99.5% for backend.
	•	Job failure rate: < 5% (excluding malformed images).
	•	Performance:
	•	Median extraction time: < 4 seconds.
	•	p95 extraction time: < 8 seconds.
	•	Efficiency:
	•	No major memory leaks or unbounded growth in MongoDB indexes.
	•	Stable latency under expected concurrency.

⸻

14. Risk Assessment and Mitigation

14.1 Technical Risks

Model Output Variability
	•	Risk: Gemini/Claude outputs may be inconsistent or misinterpret documents.
	•	Mitigation:
	•	Carefully designed prompts.
	•	Confidence scores + UI for user corrections.
	•	Optional schema-specific validation rules.

Large Image Sizes
	•	Risk: Very high-resolution images may cause timeouts or slow processing.
	•	Mitigation:
	•	Client-side recommendation to compress images.
	•	Server-side checks and resizing as needed.

Vendor Lock-In
	•	Risk: Hard dependence on Gemini and Claude.
	•	Mitigation:
	•	Abstract model calls behind a LangChain interface.
	•	Design pipeline to swap models with minimal changes.

14.2 Business Risks

Cost of AI Calls
	•	Risk: High per-request cost for VLM + LLM.
	•	Mitigation:
	•	Cache results when appropriate.
	•	Limit maximum free-tier scans.
	•	Introduce billing or rate limiting for heavy users.

Data Sensitivity Concerns
	•	Risk: Users concerned about uploading sensitive documents.
	•	Mitigation:
	•	Transparent privacy policy.
	•	Data retention controls (auto-delete after N days).
	•	Encryption and best-practice security posture.

⸻

15. Appendices

Appendix A: OpenAPI Reference
	•	Full OpenAPI spec auto-generated by FastAPI at:
	•	/openapi.json
	•	/docs (Swagger UI)
	•	/redoc

Appendix B: Prompt Templates (Conceptual)
	•	Gemini Prompt (excerpt):
	•	“You are a vision model. Given this document image and the list of entity names {entities}, extract your best guess for each entity and return a JSON object { 'entities': { '<name>': { 'value': <string>, 'confidence': <0-1>, 'evidence': <short text> } } }.”
	•	Claude Prompt (excerpt):
	•	“Normalize this JSON into the following Pydantic schema. Ensure output is valid JSON and matches the specified field types. Coerce dates to ISO8601 and numbers to floats where appropriate.”

Appendix C: Example MongoDB Document

```json
{
  "_id": { "$oid": "..." },
  "user_id": { "$oid": "..." },
  "type": "document",
  "status": "completed",
  "requested_entities": ["invoice_number", "due_date", "total_amount"],
  "azure_blob_url": "https://...blob.core.windows.net/docscan-images/user123/document/uuid.jpg",
  "thumbnail_blob_url": null,
  "extraction_id": { "$oid": "..." },
  "error_message": null,
  "created_at": "2025-11-21T12:34:56Z",
  "updated_at": "2025-11-21T12:35:20Z"
}
```

⸻

This technical specification provides a comprehensive blueprint for designing, implementing, and scaling the DocScan AI platform. It balances practical mobile-first UX with a robust AI-enabled backend, while leaving room for future enterprise features such as multi-tenancy, advanced security, and workflow integrations.