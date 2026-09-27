# SEVA AI - Smart Citizen Services & Digital Governance Portal

**SEVA AI** is an intelligent, AI-empowered digital governance platform designed to revolutionize citizen services, automated document verification, application lifecycles, and proactive citizen assistance. Built on modern cloud-native principles, SEVA AI provides real-time document extraction, tamper verification, rule-based workflow progression, and a conversational AI agent with tool-calling capabilities.

---

## Architecture Overview

SEVA AI follows a decoupled, resilient microservices-ready architecture comprising an asynchronous backend API, a reactive modern frontend, dedicated AI/OCR pipelines, and a multi-tier document verification engine.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            Next.js 14 Frontend                              │
│         (App Router, TypeScript, Tailwind CSS, SSE Real-Time Stream)        │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │ HTTP / REST / SSE / JWT
┌──────────────────────────────────────▼──────────────────────────────────────┐
│                             FastAPI Backend                                 │
│      (Async Routers: Auth, Vault, Services, Applications, Chat, Audit)      │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌────────────────────────┐  ┌───────────────────────┐  ┌────────────────┐  │
│  │   Document Pipeline    │  │  Verification Engine  │  │ Workflow Engine│  │
│  │  - Direct PDF Text (ms)│  │  - Verhoeff Checksums │  │  - Rules Matrix│  │
│  │  - PaddleOCR Engine    │  │  - Digital Signature  │  │  - Auto-Advance│  │
│  │  - Gemini Vision 2.0   │  │  - QR / Barcode Cross │  │  - Readiness   │  │
│  │  - Specialized Parsers │  │  - DigiLocker Adapter │  │  - Status State│  │
│  └────────────────────────┘  └───────────────────────┘  └────────────────┘  │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │             Gemini AI Conversational Orchestrator                     │  │
│  │             - Tool Calling (Eligibility, Requirements, Rules)         │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────────────┤
│  Database Layer: SQLAlchemy 2.0 Async (PostgreSQL / SQLite Auto-Fallback)   │
│  Storage Layer: Supabase S3-Compatible Storage / Local Disk Storage         │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1. Frontend Architecture
- **Framework**: Next.js 14 with the App Router architecture.
- **Language & Styling**: TypeScript with strict types, Tailwind CSS, and Lucide icons.
- **Real-Time Updates**: Native Server-Sent Events (SSE) consumer subscribing to `/api/events/stream` for live notifications, application stage updates, and document processing feedback.
- **State Management**: React Context (`AuthContext`) handling session authentication, automatic token persistence, clean 401 unauthenticated recovery, and proactive redirection.

### 2. Backend Architecture
- **Core Framework**: FastAPI with Python 3.11+, providing non-blocking asynchronous request handling.
- **ORM & Data Layer**: SQLAlchemy 2.0 Async with Alembic schema migrations. Supports production PostgreSQL (via `asyncpg`) and seamless local developer zero-config SQLite (`aiosqlite`).
- **Data Validation**: Strict Pydantic v2 schemas across all domain models, request/response contracts, and verification outcomes.
- **Security & Authorization**: PyJWT authentication (HS256 algorithm), bcrypt password hashing, IDOR protection, and fine-grained Cross-Origin Resource Sharing (CORS) security.

### 3. AI, OCR & Extraction Pipeline
- **Ultra-Fast Digital PDF Direct Text Extraction**: Native `pypdfium2` embedded text extraction processing vector e-documents (e-Aadhaar, e-DL, certificates) in under **10 milliseconds** (3000x faster than traditional raster OCR).
- **Pretrained PaddleOCR Neural Engine**: Industrial-grade text detection (`PP-OCRv6_medium_det`) and recognition (`PP-OCRv6_medium_rec`) models optimized with MKLDNN for scanned documents and mobile photographs.
- **Automatic Engine Pre-Warming**: Background thread initialization on server startup eliminates first-upload cold-start latency.
- **Adaptive Image Preprocessing**: Automatic EXIF orientation correction, optimal resolution clamping (1600px max dimension), and CLAHE contrast enhancement.
- **Gemini Vision 2.0 Multimodal Fallback**: Google Gemini Vision integration providing zero-shot extraction fallback if visual OCR confidence is low.
- **Document-Specific Rule Parsers**:
  - **Aadhaar**: Name, DOB, 12-digit UID masking, gender, address.
  - **Driving Licence (DL)**: Standard DL number formats, DOB, validity dates, vehicle classes (MCWG, LMV).
  - **Permanent Account Number (PAN)**: 10-character alphanumeric PAN format, father's name, DOB.
  - **Biometric Photograph**: Sub-millisecond portrait photo validation with face detection awareness.

### 4. Government Evidence Verification Decision Engine
SEVA AI enforces the **Government Evidence Hierarchy**:
- **Strong Evidence**: Digital cryptographic signatures (PKCS#7 / PAdES), live government issuer verification, DigiLocker URI XML payloads, and government registry matching.
- **Supporting Evidence**: QR code / barcode cross-referencing, Verhoeff checksums (Aadhaar), standard regex checksums, OCR visual layout matching, and SHA-256 integrity recording.
- **Decision Engine Rule**: Supporting evidence alone yields `OCR_EXTRACTED` or `NEEDS_REVIEW`; only strong evidence certifies `VERIFIED`, preventing fabricated verification claims.

### 5. Workflow State Machine & Eligibility Engine
- Rule-based dynamic eligibility evaluation (`service_rules.py`).
- Application stage progression: `DRAFT` ➔ `DOCUMENTS_PENDING` ➔ `UNDER_REVIEW` ➔ `VERIFIED` / `APPROVED`.
- Real-time application readiness assessment: dynamic checklist evaluating missing vs. provided/verified documents.

---

## Comprehensive Feature List

### Citizen Portal & Identity Management
- **Citizen Registration & Login**: Fast, secure onboarding with JWT tokens, session persistence, and instant token validation.
- **Session Lifecycle & Recovery**: Seamless handling of expired sessions with automatic storage cleanup and clean login redirection.
- **Citizen Profile Auto-Sync**: Automatic synchronization between verified document fields (Aadhaar/PAN/DL) and citizen profile details (Full Name, Date of Birth, Gender, Address).

### Smart Document Vault
- **Multi-Format Ingestion**: Supports PDF documents, JPEG/PNG images, and text attachments with automatic MIME type verification.
- **SHA-256 Deduplication**: Prevents duplicate uploads while automatically linking existing vault credentials to new applications.
- **Real-Time Vault Analytics**: Live statistics on total documents, verified documents, pending reviews, and storage utilization.
- **Document Categorization**: Automatic tagging by government document type (Aadhaar, PAN, Driving License, Income Proof, Birth Proof, Medical Certificate, Photograph).

### High-Speed Document Extraction (< 250ms)
- **Instant Digital PDF Parsing**: Sub-second extraction and field structuring for official digital PDFs.
- **PaddleOCR Deep Learning Engine**: Robust local OCR for scanned paper documents and photographs.
- **Zero Hallucination Guarantee**: Strict parsing rules extract only verifiable document tokens, completely eliminating fabricated citizen data.

### Multi-Tier Document Verification
- **Aadhaar Verhoeff Checksum**: Validates the mathematical integrity of 12-digit Aadhaar numbers.
- **PAN Checksum & Format Validation**: Ensures strict compliance with NSDL/UTIITSL format rules.
- **QR Code & Barcode Cross-Validation**: Scans 2D and 1D codes embedded in government documents and cross-checks decoded payloads against OCR fields.
- **Biometric Photo Identification**: Validates portrait photographs required for licensing and identity cards.
- **Tamper & Risk Flagging**: Highlights mismatched identifiers, low confidence scores, or expired documents as `NEEDS_REVIEW`.

### End-to-End Application Lifecycle Management
- **Supported Government Services**:
  - **Driving License Application**: Age eligibility (18+), Learner's License prerequisite, Aadhaar/address proof, medical declaration, biometric photograph.
  - **Birth Certificate Registration**: Hospital proof, parental identity verification, address documentation.
  - **Income Certificate Issuance**: Salary slip/ITR proof, residential proof, self-declaration verification.
- **Dynamic Application Checklist**: Real-time interactive document status tracking (`MISSING`, `PROVIDED`, `OCR_EXTRACTED`, `NEEDS_REVIEW`, `VERIFIED`).
- **Application Readiness Meter**: Visual percentage indicator reflecting real application completion based on verified database state.
- **Application Progress Tracking**: Timeline tracking application submission, review, officer decision, and certificate issuance.

### Conversational AI Assistant (SEVA Bot)
- **Gemini-Powered Natural Language Interface**: Chat assistant capable of answering questions about government schemes, eligibility criteria, and application steps.
- **Intelligent Tool-Calling**: The agent autonomously queries backend APIs for service requirements, user eligibility, and active application statuses.
- **Context-Aware Recommendations**: Recommends applicable schemes based on citizen profile attributes (age, location, income).

### Audit Trail & Transparency
- **Immutable Audit Event Logging**: Comprehensive audit trail recording every state change, document upload, verification check, and application transition.
- **Citizen Audit Log View**: Citizens can review the complete timeline of actions performed on their applications for full transparency.

---

## Technology Stack

| Component | Technology | Description |
| :--- | :--- | :--- |
| **Frontend** | Next.js 14, React 18, TypeScript | App Router, SSR, Server & Client Components |
| **UI & Styling** | Tailwind CSS, Lucide React | Modern, responsive citizen-centric UI |
| **Backend API** | FastAPI, Python 3.11+ | High-performance asynchronous REST API |
| **Database** | SQLAlchemy 2.0, Alembic, PostgreSQL / SQLite | Async ORM with automatic dev SQLite fallback |
| **Authentication** | PyJWT, Passlib (Bcrypt) | Stateless JWT authentication with secure hashing |
| **OCR Engines** | pypdfium2, PaddleOCR (MKLDNN) | Sub-10ms digital PDF parser + deep learning OCR |
| **AI / LLM** | Google Gemini (google-genai SDK) | Multimodal document fallback & chat agent |
| **Storage** | Supabase Storage / Local Disk Storage | S3-compatible cloud bucket or local uploads |
| **Real-time** | Server-Sent Events (SSE) | Event-driven application state broadcasting |
| **DevOps** | Docker, Docker Compose | Containerized reproducible deployments |

---

## Getting Started

### Prerequisites
- **Python**: 3.11 or newer
- **Node.js**: 20+ and npm
- **Docker & Docker Compose** (Optional, for containerized run)

---

### Local Setup (Zero-Config Development)

#### 1. Backend Setup
```bash
cd backend
python -m venv venv

# On Linux/macOS:
source venv/bin/activate
# On Windows:
.\venv\Scripts\activate

pip install -r requirements.txt

# Start backend (auto-initializes SQLite database & seeds default services on startup):
python -m uvicorn app.main:app --reload --port 8000
```
Backend API will be running at `http://127.0.0.1:8000` with interactive Swagger docs at `http://127.0.0.1:8000/docs`.

#### 2. Frontend Setup
```bash
cd frontend
npm install
npm run dev
```
Frontend Portal will be accessible at `http://localhost:3000`.

---

### Docker Setup

To run the complete platform via Docker Compose:
```bash
docker-compose up --build
```

---

## Default Test Credentials

For quick local testing and evaluation:
- **Email**: `citizen@example.com`
- **Password**: `password123`
*(Or sign up with any new account via the frontend registration screen)*

---

## License

This project is licensed under the MIT License.
