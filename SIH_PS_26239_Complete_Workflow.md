# SIH-PS-26239: AI-Enabled Scholarship & Fellowship Management System
## End-to-End Project Workflow & Architectural Blueprint
**Ministry of Tribal Affairs (MoTA), Government of India**

---

![SIH PS 26239 Workflow Infographic](/C:/Users/Suman/.gemini/antigravity/brain/3c0808e4-b201-4f70-858a-8dc39c717ec1/sih_workflow_infographic.jpg)

---

## 1. Executive Workflow Summary

The system orchestrates a 9-stage unified digital lifecycle:
$$\text{REGISTER} \longrightarrow \text{VERIFY} \longrightarrow \text{APPLY} \longrightarrow \text{AI/OCR PROCESS} \longrightarrow \text{VALIDATE} \longrightarrow \text{SELECT} \longrightarrow \text{SANCTION} \longrightarrow \text{DISBURSE} \longrightarrow \text{MONITOR}$$

It eliminates repetitive physical documentation, automates fraud and eligibility scrutiny via OCR & configurable rules, coordinates a 3-tier verification hierarchy, generates automated merit lists, triggers Direct Benefit Transfer (DBT) via PFMS/NPCI, and supports students 24/7 via the **JAGO Multilingual AI Chatbot**.

---

## 2. Master End-to-End System Flowchart

The following diagram maps the entire macro journey of a student application, showing happy paths, anomaly branches, deficiency re-entry loops, and post-selection operations.

```mermaid
flowchart TD
    subgraph PHASE1["Phase 1: Registration & Profile Creation"]
        A["Student Starts OTR"] --> B["Aadhaar e-KYC Verification"]
        B --> C["Federated Data Ingestion<br/>(DigiLocker, APAAR, UDISE+, PFMS)"]
        C --> D["Golden Student Profile Generated<br/>(Unique Student ID Assigned)"]
    end

    subgraph PHASE2["Phase 2: Scheme Discovery & AI-OCR Processing"]
        D --> E["Select Scheme<br/>(Pre/Post-Matric, Top Class, NFST, NOS)"]
        E --> F["Smart Auto-Fill Form & Document Upload"]
        F --> G["AI-OCR Engine<br/>(LayoutLM / PaddleOCR / Vision API)"]
        G --> H{"OCR Integrity &<br/>Tamper Check"}
        H -- "Manipulated / Low Res" --> H1["Exception Raised: Re-upload Prompt"]
        H1 --> F
        H -- "Valid Extraction" --> I["Standardized Machine-Readable Data"]
    end

    subgraph PHASE3["Phase 3: Rules Engine & 3-Tier Verification"]
        I --> J["Configurable Business Rules Engine (BRE)"]
        J --> K["Tier 1: AI Automated Audit<br/>(Anomaly, Duplicate & Fraud Check)"]
        K --> L{"Deficiency Detected?"}
        L -- "Yes" --> L1["Deficiency Flagged<br/>(SMS / WhatsApp / JAGO Bot Alert)"]
        L1 --> L2["Student Correction & Resubmission"]
        L2 --> J
        L -- "No (Passed Tier 1)" --> M["Tier 2: Educational Institution Verification<br/>(Enrollment, Course Duration, Fees)"]
        M --> N{"Institute Action"}
        N -- "Deficiency / Incomplete" --> L1
        N -- "Rejected" --> REJ["Application Terminated with Audit Log"]
        N -- "Approved" --> O["Tier 3: State Agency / MoTA Authority<br/>(Quota, Reservation, Budget Ceilings)"]
        O --> P{"State/MoTA Action"}
        P -- "Deficiency" --> L1
        P -- "Rejected" --> REJ
        P -- "Approved" --> Q["Fully Verified Application Pool"]
    end

    subgraph PHASE4["Phase 4: Selection & Merit List Generation"]
        Q --> R["Scheme-Specific Scrutiny & Scoring Engine"]
        R --> S{"Scheme Type"}
        S -- "NFST" --> S1["Research Proposal & Guide Assessment"]
        S -- "NOS" --> S2["Top World Univ Rank + GRE/IELTS + Academic Score"]
        S -- "Post/Pre-Matric" --> S3["Family Income Slab + Academic Percentile + PVTG Priority"]
        S1 --> T["Normalized Merit Score"]
        S2 --> T
        S3 --> T
        T --> U["Category-wise Merit Ranking & Committee Sign-off"]
        U --> V["Digital Sanction Order Generation<br/>(Cryptographic Sanction ID)"]
    end

    subgraph PHASE5["Phase 5: DBT Disbursement & Post-Selection"]
        V --> W["Payment Order Pushed to PFMS"]
        W --> X["NPCI Aadhaar Payment Bridge (APBS)"]
        X --> Y["Student Direct Bank Account Credited"]
        Y --> Z["Disbursement Notification & Receipt Issued"]
        Z --> AA["Post-Selection Academic & Renewal Monitoring"]
        AA --> AB["Continuation / Next Academic Year Verification"]
    end

    subgraph CROSS["Cross-Cutting Intelligence Layer"]
        BOT["JAGO Multilingual AI Chatbot<br/>(Voice, Vernacular, Status, Helpdesk)"] -.-> A
        BOT -.-> E
        BOT -.-> L1
        BOT -.-> Z
        ANALYTICS["MoTA Real-time Executive Analytics Dashboard<br/>(Bottlenecks, Spend Tracking, Heatmaps)"] -.-> PHASE3
        ANALYTICS -.-> PHASE4
        ANALYTICS -.-> PHASE5
    end
```

---

## 3. Swimlane Sequence Diagram: Role-Based Workflow

This interaction flow details the sequence of calls between actors across the verification, audit, and payment stages.

```mermaid
sequenceDiagram
    autonumber
    actor S as Student / Scholar
    participant J as JAGO Chatbot
    participant UI as Portal Web/Mobile
    participant AI as AI-OCR & Anti-Fraud
    participant BRE as Business Rules Engine
    participant INST as Institute Nodal Officer
    participant GOV as MoTA / State Admin
    participant PFMS as PFMS / NPCI DBT Gateway

    Note over S,UI: Phase 1 & 2: Registration & Submission
    S->>UI: Input Aadhaar & Consent
    UI->>AI: Trigger e-KYC & DigiLocker Sync
    AI-->>UI: Return verified identity & caste credentials
    UI->>S: Display pre-filled Golden Profile
    S->>UI: Select Scheme (e.g., NFST / NOS) & upload docs
    UI->>AI: Process uploaded certs & synopsis
    AI-->>BRE: Structured tokens + Confidence Score (>95%)

    Note over BRE,INST: Phase 3: Multi-Tier Verification & Deficiency Loop
    BRE->>AI: Run duplicate check across nationwide DB
    alt Suspicious / Mismatched Data Found
        AI-->>S: Tier 1 Deficiency alert via SMS & WhatsApp
        S->>J: Ask "How do I fix income certificate deficiency?"
        J-->>S: Conversational guidance in native language
        S->>UI: Re-upload corrected document
    else Clear Audit
        AI->>INST: Dispatch to Institute Nodal Queue
    end

    INST->>INST: Verify college admission, roll number & fee slip
    alt Institute detects invalid enrollment
        INST-->>S: Raise Tier 2 Deficiency with comment
    else Approved
        INST->>GOV: Push to State / MoTA Approval Queue
    end

    GOV->>GOV: Check category quota, budget limits & state domicile
    GOV-->>BRE: Mark as Approved for Selection

    Note over GOV,PFMS: Phase 4 & 5: Merit, Sanction & DBT
    BRE->>GOV: Generate Scheme Merit Rank
    GOV->>UI: Issue Digital Sanction Letter with QR Code
    GOV->>PFMS: Push XML Payment Batch to PFMS
    PFMS->>PFMS: NPCI Aadhaar Bridge Account Validation
    PFMS-->>S: Amount credited directly to bank account
    PFMS-->>UI: Payment Status: SUCCESS (UTR Reference)
    UI-->>J: Update timeline for inquiry tracking
```

---

## 4. Application Lifecycle State Machine

Each application advances through deterministic states with clear transition triggers and rollback capabilities:

```mermaid
stateDiagram-v2
    [*] --> DRAFT: Student registers OTR
    DRAFT --> PROFILE_VERIFIED: Aadhaar eKYC & DigiLocker success
    PROFILE_VERIFIED --> SUBMITTED: Scheme chosen & documents submitted
    
    SUBMITTED --> TIER1_AI_AUDIT: System queues application
    TIER1_AI_AUDIT --> DEFICIENCY_TIER1: Fraud / Document mismatch flagged
    DEFICIENCY_TIER1 --> SUBMITTED: Student resubmits corrected doc
    
    TIER1_AI_AUDIT --> TIER2_INSTITUTE_PENDING: AI confidence > 90%
    TIER2_INSTITUTE_PENDING --> DEFICIENCY_TIER2: Academic / Fee discrepancy
    DEFICIENCY_TIER2 --> TIER2_INSTITUTE_PENDING: Student clarifies via portal
    TIER2_INSTITUTE_PENDING --> REJECTED: Ineligible / Fake student
    
    TIER2_INSTITUTE_PENDING --> TIER3_GOV_PENDING: Institute approves
    TIER3_GOV_PENDING --> DEFICIENCY_TIER3: Domicile / Category quota query
    DEFICIENCY_TIER3 --> TIER3_GOV_PENDING: Student resubmits
    TIER3_GOV_PENDING --> REJECTED: Over budget / Exceeded attempts
    
    TIER3_GOV_PENDING --> VERIFIED_POOL: State & MoTA final sign-off
    VERIFIED_POOL --> MERIT_RANKED: Automated scoring applied
    MERIT_RANKED --> SANCTIONED: Digital Sanction Order issued
    SANCTIONED --> PAYMENT_IN_PROGRESS: PFMS batch pushed
    PAYMENT_IN_PROGRESS --> PAYMENT_FAILED: IFSC / Aadhaar unlink error
    PAYMENT_FAILED --> PAYMENT_IN_PROGRESS: Account corrected
    PAYMENT_IN_PROGRESS --> DISBURSED: NPCI confirms credit
    
    DISBURSED --> POST_MONITORING: Active scholarship monitoring
    POST_MONITORING --> [*]: Course completion / Degree awarded
```

---

## 5. Detailed Phase-by-Phase Technical Blueprint

### Phase 1: One-Time Registration (OTR) & Golden Profile Creation
* **Primary Objective**: Eliminate redundant paperwork by creating a persistent, verified digital identity.
* **Integrations**:
  1. **Aadhaar e-KYC**: Real-time demographic verification (UIDAI OTP/Biometric).
  2. **DigiLocker**: Automated retrieval of Caste (ST/PVTG) certificates, Class 10/12 marksheets, and Domicile certificates directly from state issuers.
  3. **APAAR / ABC (Academic Bank of Credits)**: Automated academic history fetch.
  4. **UDISE+**: School verification for Pre-Matric candidates.
  5. **PFMS / NPCI APBS**: Bank account validation checking whether account is active and seeded with Aadhaar.
* **Output**: **Unique Student ID** (e.g., `MOTA-2026-ST-092817`) with an immutable verified credential store.

---

### Phase 2: Smart Application & AI-OCR Document Processing
* **Supported Schemes**:
  * *Pre-Matric Scholarship for ST Students* (Classes IX & X)
  * *Post-Matric Scholarship for ST Students* (Class XI to Post-Graduation)
  * *Top Class Education Scheme for ST Students* (Premier institutes like IITs, IIMs, NITs)
  * *National Fellowship for Higher Education of ST Students (NFST)* (M.Phil / Ph.D. scholars)
  * *National Overseas Scholarship for ST Candidates (NOS)* (Master's / Ph.D. abroad)
* **Smart Auto-Fill**:
  When a student chooses a scheme, $80\%$ of fields are pre-populated from the Golden Profile.
* **AI-OCR Pipeline**:
  ```
  Uploaded PDF/Image 
       ↓ 
  Image Pre-processing (Deskew, Denoise, Contrast Enhancement)
       ↓ 
  OCR Engine (PaddleOCR / Tesseract 5 / TrOCR)
       ↓ 
  Layout & Entity Extraction (LayoutLMv3 / Donut / NER)
       ↓ 
  Confidence Scoring & Tamper Check (EXIF analysis, ELA, Digital Signature)
       ↓ 
  Structured JSON Tokenization
  ```
* **Validation Rules**:
  * Name phonetic match (Levenshtein distance $\le 2$ or Jaro-Winkler score $\ge 0.92$).
  * ST/PVTG certificate issuing authority validation via state repository hashes.
  * Institution accreditation validation via AISHE / NIRF code database.

---

### Phase 3: Configurable Rules Engine & 3-Tier Verification
* **Business Rules Engine (BRE)**:
  Implemented using a declarative decision engine (e.g., JSON Rules Engine / Drools / Celery workflow):
  * **Income Ceiling**: e.g., $\le ₹2.5\text{ Lakhs/year}$ for Post-Matric; $\le ₹6.0\text{ Lakhs}$ for Top Class; $\le ₹8.0\text{ Lakhs}$ for NOS.
  * **Academic Eligibility**: Minimum percentage criteria or qualifying exam percentiles.
  * **Scheme Exclusivity**: Checking against NSP to prevent double-dipping across ministries.
* **The 3-Tier Verification Hierarchy**:
  1. **Tier 1 – AI Audit**: Immediate instant check for missing fields, expired certificates, fuzzy name mismatches, duplicate documents across any sibling/parent records, and synthetic identity markers.
  2. **Tier 2 – Educational Institution (INO Queue)**: College Nodal Officer verifies admission register number, physical attendance/enrollment, course duration, and actual tuition/hostel fee structure.
  3. **Tier 3 – State Agency / MoTA Officer (SNO Queue)**: Final statutory review verifying state tribal quotas, special PVTG reservations, and financial sanction ceilings.
* **Deficiency & Resubmission Feedback Loop**:
  * If any tier flags a missing or ambiguous document, status becomes `DEFICIENCY_RAISED`.
  * Multi-channel alert triggered: SMS, WhatsApp alert, and banner notification in student dashboard.
  * The student sees exactly which field/page is defective with highlighted instructions.
  * Resubmitted docs bypass initial steps and return directly to the tier that requested the correction.

---

### Phase 4: Selection & Merit List Generation
Different schemes utilize specific mathematical selection criteria:

| Scheme | Evaluation Criteria | Selection Logic |
| :--- | :--- | :--- |
| **NFST (Fellowship)** | Research proposal quality, guide credentials, UGC-NET/GATE score | Weighted Index = $0.4(\text{Proposal}) + 0.3(\text{NET Score}) + 0.3(\text{PG Marks})$ |
| **NOS (Overseas)** | Foreign University QS Rank, GRE/IELTS band, Academic merit | Tiered by QS World Top 500 Ranking + Category Quota |
| **Top Class** | Admission in notified premier institute (IIT/IIM/AIIMS) | 100% course fee + living allowance up to annual ceiling |
| **Post-Matric / Pre-Matric** | Need-cum-merit: Income slab + Previous year grade + PVTG priority | Sorted by lowest family income first, breaking ties by academic score |

* **Digital Sanction Process**:
  * Selection Committee approves digitally through e-Sign / Digital Signature Certificate (DSC).
  * System generates a tamper-evident **Digital Sanction Order** with a verifiable QR code, allocated amount, academic duration, and unique Sanction ID.

---

### Phase 5: DBT Payment Processing & Post-Selection Monitoring
* **Disbursement Architecture**:
  $$\text{Sanction Order} \xrightarrow{\text{Batch XML/API}} \text{PFMS} \xrightarrow{\text{Aadhaar Bridge}} \text{NPCI} \xrightarrow{\text{APBS Direct Credit}} \text{Student Bank Account}$$
* **Payment Lifecycle States**:
  * `SANCTIONED` $\rightarrow$ `PAYMENT_INITIATED` $\rightarrow$ `PFMS_VALIDATED` $\rightarrow$ `APBS_PROCESSING` $\rightarrow$ `DISBURSED (UTR)`
  * *Failure Recovery*: If payment bounces due to Aadhaar de-linking or inactive KYC, an automatic notification alerts the student with actionable bank rectification steps.
* **Post-Selection & Renewal Monitoring**:
  * Bi-annual academic progress tracker (semester marksheet upload via DigiLocker).
  * Institution attendance verification token.
  * Fellowship progress reports uploaded by research guide for subsequent tranche releases.

---

## 6. Cross-Cutting Systems

### A. JAGO Multilingual AI Chatbot
* **Accessibility**: Supports Hindi, English, Santhali, Gondi, Odia, Bengali, Telugu, Marathi, and other regional languages.
* **Modalities**: Text, Voice-to-Text, and Text-to-Speech (utilizing Bhashini AI engine).
* **Capabilities**:
  * Scheme recommendation based on student qualification and tribe.
  * Real-time application tracking ("Where is my application stuck?").
  * Step-by-step guidance on resolving specific deficiency notifications.
  * Automated payment receipt lookup and grievance logging.

### B. Anti-Fraud & Anomaly Detection System
* **Document Hash Matching**: SHA-256 hash checks on uploaded PDFs to detect reused certificates across multiple profiles.
* **Metadata & ELA Forensics**: Error Level Analysis (ELA) to detect Photoshop edits on income figures or dates.
* **Cross-Scheme Duplicate Prevention**: Real-time cross-referencing with NSP (National Scholarship Portal) and state scholarship databases to prevent multiple simultaneous claims.
* **Geographical & Institution Anomaly Detector**: Flags high concentrations of applications from non-existent or un-accredited private coaching centers claiming institutional fees.

### C. MoTA Executive Analytics Dashboard
* **Macro KPIs**: Total applications, verification velocity, average turnaround time (TAT) per tier, total disbursed funds, pending deficiency clearance rate.
* **Drilldown Capabilities**:
  * State-wise and District-wise tribal coverage heatmaps.
  * PVTG (Particularly Vulnerable Tribal Groups) saturation metrics.
  * Institution-level bottleneck detection (identifying colleges lagging in Tier-2 approvals).

---

## 7. Recommended Technology Stack for Hackathon Prototype

```
┌────────────────────────────────────────────────────────┐
│                        FRONTEND                        │
│   React.js / Next.js 14 + Tailwind CSS + PWA Support   │
│       Accessible (WCAG 2.1 AA) + Multilingual UI       │
└──────────────────────────┬─────────────────────────────┘
                           │ REST / GraphQL APIs
┌──────────────────────────▼─────────────────────────────┐
│                    APPLICATION BACKEND                 │
│      Python (FastAPI) / Node.js (NestJS) + Redis       │
│      JWT Auth + Role-Based Access Control (RBAC)       │
└──────┬───────────────────┬───────────────────┬─────────┘
       │                   │                   │
┌──────▼──────┐     ┌──────▼──────┐     ┌──────▼──────┐
│  AI / OCR   │     │RULES ENGINE │     │ JAGO BOT    │
│  PaddleOCR  │     │ Python Rule │     │ LangChain   │
│  OpenCV     │     │ Engine /    │     │ Bhashini    │
│  HuggingFace│     │ Celery Flow │     │ RAG VectorDB│
└─────────────┘     └─────────────┘     └─────────────┘
       │                   │                   │
┌──────▼───────────────────▼───────────────────▼─────────┐
│                    DATABASE & STORAGE                  │
│   PostgreSQL (Transactional) + MinIO / S3 (Encrypted)  │
│         Milvus / ChromaDB (Vector Search for RAG)      │
└────────────────────────────────────────────────────────┘
```
