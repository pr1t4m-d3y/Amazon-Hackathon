# Design Document: HealthSathi

## Overview

HealthSathi is a voice-first AI-powered web application built with a React.js frontend and Python FastAPI backend. The system provides hospital navigation, prescription simplification, doctor availability information, and medicine lookup services to patients and attendants in public hospitals.

The architecture follows a client-server model with clear separation between presentation (React), business logic (FastAPI), AI processing (Google Gemini API, EasyOCR), and data persistence (PostgreSQL/SQLite). A critical Safety Layer validates all AI outputs to prevent medical advice, diagnosis, or hallucinated information.

**Key Design Principles:**
- **Safety First**: All AI outputs pass through validation before reaching users
- **Accessibility**: Voice-first design with text fallback for low digital literacy users
- **Privacy**: In-memory processing by default, persistent storage only with consent
- **Simplicity**: Mobile-first responsive UI with clear navigation
- **Bilingual**: Full support for English and Hindi throughout the system

## Architecture

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Client Layer (Browser)                   │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  React.js + Vite + Tailwind CSS                        │ │
│  │  - Voice Interface (Web Speech API)                    │ │
│  │  - Responsive UI Components                            │ │
│  │  - State Management (React Context/Zustand)            │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                            │ HTTPS/REST
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   Application Layer (FastAPI)                │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  API Endpoints                                         │ │
│  │  - Navigation Service                                  │ │
│  │  - Symptom Mapping Service                            │ │
│  │  - Doctor Schedule Service                            │ │
│  │  - Prescription Processing Service                    │ │
│  │  - Medicine Lookup Service                            │ │
│  └────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Safety Layer (Middleware)                            │ │
│  │  - AI Output Validation                               │ │
│  │  - Medical Disclaimer Injection                       │ │
│  │  - Content Filtering                                  │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                            │
                ┌───────────┴───────────┐
                ▼                       ▼
┌──────────────────────────┐  ┌──────────────────────────┐
│   AI/ML Layer            │  │   Data Layer             │
│  ┌────────────────────┐  │  │  ┌────────────────────┐  │
│  │ Google Gemini API  │  │  │  │ PostgreSQL/SQLite  │  │
│  │ (Text Simplify)    │  │  │  │ - Hospital Data    │  │
│  └────────────────────┘  │  │  │ - Doctor Schedules │  │
│  ┌────────────────────┐  │  │  │ - Medicine DB      │  │
│  │ EasyOCR            │  │  │  │ - User Consent     │  │
│  │ (Image to Text)    │  │  │  └────────────────────┘  │
│  └────────────────────┘  │  └──────────────────────────┘
└──────────────────────────┘
```

### Component Responsibilities

**Frontend (React.js):**
- Render mobile-first responsive UI
- Handle voice input/output via Web Speech API
- Manage user interactions and form submissions
- Display results with bilingual support
- Handle image uploads for prescriptions

**Backend (FastAPI):**
- Expose RESTful API endpoints
- Orchestrate business logic and service calls
- Validate and sanitize inputs
- Apply Safety Layer to all AI outputs
- Manage database queries
- Handle file uploads and OCR processing

**AI/ML Services:**
- **Google Gemini API**: Simplify medical text into plain language
- **EasyOCR**: Extract text from prescription images
- Both services are called asynchronously from FastAPI

**Safety Layer:**
- Validate AI outputs for prohibited content (diagnosis, treatment, dosage changes)
- Inject medical disclaimers into all AI responses
- Log and block unsafe outputs
- Implemented as FastAPI middleware/dependency

**Data Layer:**
- Store hospital floor maps, departments, rooms
- Store doctor schedules and availability
- Store medicine database (brand names, generics, active ingredients)
- Store user consent records (if persistent storage is used)

## Components and Interfaces

### Frontend Components

**1. VoiceInterface Component**
```typescript
interface VoiceInterfaceProps {
  onTranscript: (text: string) => void;
  onError: (error: string) => void;
  language: 'en' | 'hi';
}

// Uses Web Speech API
// - SpeechRecognition for STT
// - SpeechSynthesis for TTS
```

**2. NavigationSearch Component**
```typescript
interface NavigationSearchProps {
  onSearch: (query: string) => void;
  results: NavigationResult[];
  loading: boolean;
}

interface NavigationResult {
  departmentName: string;
  floor: number;
  roomNumber: string;
  directions: string;
}
```

**3. PrescriptionUpload Component**
```typescript
interface PrescriptionUploadProps {
  onUpload: (file: File) => void;
  onSimplified: (result: SimplifiedPrescription) => void;
  loading: boolean;
}

interface SimplifiedPrescription {
  originalText: string;
  simplifiedText: string;
  disclaimer: string;
  language: 'en' | 'hi';
}
```

**4. DoctorSchedule Component**
```typescript
interface DoctorScheduleProps {
  doctors: Doctor[];
  onBookToken: (doctorId: string) => void;
}

interface Doctor {
  id: string;
  name: string;
  specialization: string;
  roomNumber: string;
  schedule: TimeSlot[];
  availability: 'Available' | 'Busy' | 'Off-duty';
}

interface TimeSlot {
  day: string;
  startTime: string;
  endTime: string;
}
```

**5. MedicineLookup Component**
```typescript
interface MedicineLookupProps {
  onSearch: (medicineName: string) => void;
  results: MedicineInfo | null;
}

interface MedicineInfo {
  brandName: string;
  genericName: string;
  activeIngredients: string[];
  genericEquivalents: string[];
  commonUses: string;
  sideEffects: string;
  disclaimer: string;
}
```

### Backend API Endpoints

**Base URL:** `/api/v1`

**1. Navigation Endpoints**

```python
POST /navigation/search
Request:
{
  "query": "cardiology department",
  "language": "en"
}

Response:
{
  "results": [
    {
      "department_name": "Cardiology",
      "floor": 3,
      "room_number": "301-A",
      "directions": "Take elevator to 3rd floor, turn right"
    }
  ],
  "suggestions": []  # If query is ambiguous
}
```

**2. Symptom Mapping Endpoints**

```python
POST /symptoms/map
Request:
{
  "symptoms": "chest pain and shortness of breath",
  "language": "en"
}

Response:
{
  "departments": [
    {
      "name": "Cardiology",
      "reason": "Chest pain may indicate heart-related issues",
      "floor": 3,
      "room_number": "301-A"
    },
    {
      "name": "Emergency",
      "reason": "Immediate attention recommended",
      "floor": 1,
      "room_number": "ER-1"
    }
  ],
  "disclaimer": "This is not a diagnosis. Please consult a doctor."
}
```

**3. Doctor Schedule Endpoints**

```python
GET /doctors/search?name={name}&specialization={spec}&department={dept}
Response:
{
  "doctors": [
    {
      "id": "doc_123",
      "name": "Dr. Sharma",
      "specialization": "Cardiologist",
      "department": "Cardiology",
      "room_number": "301-B",
      "schedule": [
        {
          "day": "Monday",
          "start_time": "09:00",
          "end_time": "13:00"
        }
      ],
      "availability": "Available"
    }
  ]
}

POST /doctors/book-token
Request:
{
  "doctor_id": "doc_123",
  "patient_name": "John Doe"  # Optional for MVP
}

Response:
{
  "token_number": "A-042",
  "estimated_wait_minutes": 30,
  "doctor_name": "Dr. Sharma",
  "room_number": "301-B"
}
```

**4. Prescription Processing Endpoints**

```python
POST /prescriptions/upload
Content-Type: multipart/form-data
Request:
{
  "image": <file>,
  "language": "en",
  "consent_to_store": false
}

Response:
{
  "prescription_id": "rx_789",
  "extracted_text": "Tab. Metformin 500mg...",
  "ocr_confidence": 0.92,
  "processing_status": "success"
}

POST /prescriptions/simplify
Request:
{
  "prescription_id": "rx_789",
  "text": "Tab. Metformin 500mg BD PC",
  "language": "en"
}

Response:
{
  "original_text": "Tab. Metformin 500mg BD PC",
  "simplified_text": "Metformin tablet, 500 milligrams. Take twice daily after meals.",
  "disclaimer": "This is for informational purposes only. Follow your doctor's instructions. Do not change dosage without consulting your doctor.",
  "language": "en",
  "safety_check_passed": true
}
```

**5. Medicine Lookup Endpoints**

```python
GET /medicines/search?name={medicine_name}&language={lang}
Response:
{
  "brand_name": "Crocin",
  "generic_name": "Paracetamol",
  "active_ingredients": ["Paracetamol 500mg"],
  "generic_equivalents": ["Dolo 650", "Calpol", "Paracip"],
  "common_uses": "Pain relief, fever reduction",
  "side_effects": "Rare: nausea, allergic reactions",
  "disclaimer": "This is informational only. Consult your doctor before substituting medicines.",
  "language": "en"
}
```

**6. Health Check Endpoint**

```python
GET /health
Response:
{
  "status": "healthy",
  "services": {
    "database": "connected",
    "gemini_api": "available",
    "ocr_service": "available"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

### Safety Layer Interface

```python
class SafetyLayer:
    """
    Validates AI outputs to prevent medical advice, diagnosis, or hallucinations.
    """
    
    def validate_output(self, text: str, context: str) -> SafetyResult:
        """
        Args:
            text: AI-generated text to validate
            context: Context type ('prescription', 'symptom', 'medicine')
        
        Returns:
            SafetyResult with validation status and filtered text
        """
        pass
    
    def inject_disclaimer(self, text: str, context: str, language: str) -> str:
        """
        Appends appropriate medical disclaimer to text.
        """
        pass
    
    def detect_prohibited_content(self, text: str) -> List[str]:
        """
        Detects prohibited patterns:
        - Diagnosis statements ("You have...", "This indicates...")
        - Treatment recommendations ("You should take...", "Increase dosage...")
        - Dosage changes ("Take more/less...", "Stop taking...")
        
        Returns:
            List of detected violations
        """
        pass

class SafetyResult:
    passed: bool
    filtered_text: str
    violations: List[str]
    confidence: float
```

## Data Models

### Database Schema (PostgreSQL/SQLite)

**1. Departments Table**
```sql
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    name_hindi VARCHAR(100),
    floor INTEGER NOT NULL,
    room_number VARCHAR(20) NOT NULL,
    description TEXT,
    keywords TEXT[]  -- For search matching
);
```

**2. Doctors Table**
```sql
CREATE TABLE doctors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    specialization VARCHAR(100) NOT NULL,
    department_id INTEGER REFERENCES departments(id),
    room_number VARCHAR(20) NOT NULL,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**3. Doctor Schedules Table**
```sql
CREATE TABLE doctor_schedules (
    id SERIAL PRIMARY KEY,
    doctor_id UUID REFERENCES doctors(id),
    day_of_week INTEGER NOT NULL,  -- 0=Monday, 6=Sunday
    start_time TIME NOT NULL,
    end_time TIME NOT NULL,
    is_active BOOLEAN DEFAULT TRUE
);
```

**4. Medicines Table**
```sql
CREATE TABLE medicines (
    id SERIAL PRIMARY KEY,
    brand_name VARCHAR(200) NOT NULL,
    generic_name VARCHAR(200) NOT NULL,
    active_ingredients TEXT[] NOT NULL,
    common_uses TEXT,
    side_effects TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(brand_name, generic_name)
);
```

**5. Medicine Equivalents Table**
```sql
CREATE TABLE medicine_equivalents (
    id SERIAL PRIMARY KEY,
    medicine_id INTEGER REFERENCES medicines(id),
    equivalent_brand_name VARCHAR(200) NOT NULL,
    equivalent_medicine_id INTEGER REFERENCES medicines(id)
);
```

**6. Symptom Mappings Table**
```sql
CREATE TABLE symptom_mappings (
    id SERIAL PRIMARY KEY,
    symptom_keywords TEXT[] NOT NULL,
    department_id INTEGER REFERENCES departments(id),
    priority INTEGER DEFAULT 1,  -- Higher = more urgent
    reason TEXT NOT NULL,
    reason_hindi TEXT
);
```

**7. User Consent Table** (Optional - only if user opts in)
```sql
CREATE TABLE user_consents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id VARCHAR(100) UNIQUE NOT NULL,
    consent_given BOOLEAN NOT NULL,
    consent_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    data_retention_days INTEGER DEFAULT 30
);
```

**8. Prescription Records Table** (Optional - only with consent)
```sql
CREATE TABLE prescription_records (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id VARCHAR(100) REFERENCES user_consents(session_id),
    original_text TEXT,
    simplified_text TEXT,
    language VARCHAR(2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP  -- Auto-delete after retention period
);
```

**9. Appointment Tokens Table**
```sql
CREATE TABLE appointment_tokens (
    id SERIAL PRIMARY KEY,
    token_number VARCHAR(10) NOT NULL,
    doctor_id UUID REFERENCES doctors(id),
    patient_name VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'waiting',  -- waiting, called, completed
    UNIQUE(doctor_id, token_number, DATE(created_at))
);
```

### Python Data Models (Pydantic)

```python
from pydantic import BaseModel, Field
from typing import List, Optional
from datetime import time, datetime
from enum import Enum

class Language(str, Enum):
    EN = "en"
    HI = "hi"

class AvailabilityStatus(str, Enum):
    AVAILABLE = "Available"
    BUSY = "Busy"
    OFF_DUTY = "Off-duty"

# Navigation Models
class NavigationQuery(BaseModel):
    query: str = Field(..., min_length=1, max_length=200)
    language: Language = Language.EN

class NavigationResult(BaseModel):
    department_name: str
    floor: int
    room_number: str
    directions: str
    suggestions: Optional[List[str]] = []

# Symptom Models
class SymptomQuery(BaseModel):
    symptoms: str = Field(..., min_length=1, max_length=500)
    language: Language = Language.EN

class DepartmentRecommendation(BaseModel):
    name: str
    reason: str
    floor: int
    room_number: str

class SymptomResponse(BaseModel):
    departments: List[DepartmentRecommendation]
    disclaimer: str

# Doctor Models
class TimeSlot(BaseModel):
    day: str
    start_time: str
    end_time: str

class Doctor(BaseModel):
    id: str
    name: str
    specialization: str
    department: str
    room_number: str
    schedule: List[TimeSlot]
    availability: AvailabilityStatus

class TokenBooking(BaseModel):
    doctor_id: str
    patient_name: Optional[str] = None

class TokenResponse(BaseModel):
    token_number: str
    estimated_wait_minutes: int
    doctor_name: str
    room_number: str

# Prescription Models
class PrescriptionUpload(BaseModel):
    language: Language = Language.EN
    consent_to_store: bool = False

class OCRResult(BaseModel):
    prescription_id: str
    extracted_text: str
    ocr_confidence: float
    processing_status: str

class SimplificationRequest(BaseModel):
    prescription_id: Optional[str] = None
    text: str = Field(..., min_length=1)
    language: Language = Language.EN

class SimplificationResponse(BaseModel):
    original_text: str
    simplified_text: str
    disclaimer: str
    language: Language
    safety_check_passed: bool

# Medicine Models
class MedicineInfo(BaseModel):
    brand_name: str
    generic_name: str
    active_ingredients: List[str]
    generic_equivalents: List[str]
    common_uses: str
    side_effects: str
    disclaimer: str
    language: Language

# Safety Models
class SafetyResult(BaseModel):
    passed: bool
    filtered_text: str
    violations: List[str]
    confidence: float
```

## Data Flow: Prescription Analysis Feature

This section details the complete data flow for the prescription simplification feature, which is the core AI-powered component of HealthSathi.

### Step-by-Step Flow

**1. User Uploads Prescription Image (Frontend)**
```
User Action: Clicks "Upload Prescription" → Selects image file
↓
React Component: PrescriptionUpload
- Validates file type (jpg, png, pdf)
- Validates file size (< 10MB)
- Shows preview to user
- Displays language selector (EN/HI)
- Displays consent checkbox
↓
API Call: POST /api/v1/prescriptions/upload
- Content-Type: multipart/form-data
- Body: {image: File, language: "en", consent_to_store: false}
```

**2. Backend Receives Upload (FastAPI)**
```
FastAPI Endpoint: upload_prescription()
↓
Input Validation:
- Check file format
- Check file size
- Sanitize filename
- Generate unique prescription_id
↓
Temporary Storage:
- Save to /tmp/prescriptions/{prescription_id}.jpg
- Store in memory if consent_to_store = false
```

**3. OCR Processing (EasyOCR)**
```
OCR Service Call:
- Load image from temporary storage
- Initialize EasyOCR reader with language ['en', 'hi']
- Extract text with confidence scores
↓
EasyOCR Output:
[
  (['Tab.', 'Metformin', '500mg'], 0.95),
  (['BD', 'PC'], 0.88),
  (['Dr.', 'Sharma'], 0.92)
]
↓
Text Aggregation:
- Combine text fragments
- Filter low-confidence results (< 0.7)
- Format into readable string
↓
Result: "Tab. Metformin 500mg BD PC\nDr. Sharma"
```

**4. Return OCR Result to Frontend**
```
Response: OCRResult
{
  "prescription_id": "rx_abc123",
  "extracted_text": "Tab. Metformin 500mg BD PC",
  "ocr_confidence": 0.92,
  "processing_status": "success"
}
↓
Frontend displays extracted text
User confirms or edits text
User clicks "Simplify"
```

**5. Simplification Request (Frontend → Backend)**
```
API Call: POST /api/v1/prescriptions/simplify
Body: {
  "prescription_id": "rx_abc123",
  "text": "Tab. Metformin 500mg BD PC",
  "language": "en"
}
```

**6. AI Simplification (Google Gemini API)**
```
FastAPI Endpoint: simplify_prescription()
↓
Prepare Gemini Prompt:
"""
You are a medical text simplifier for patients with low health literacy.
Simplify the following prescription text into plain {language} language.

Rules:
1. Explain medical abbreviations (BD = twice daily, PC = after meals)
2. Keep all numerical values EXACTLY as written
3. Use simple words
4. Do NOT provide dosage advice
5. Do NOT diagnose conditions
6. Do NOT recommend treatment changes

Prescription: {text}

Output format:
Medicine name, dosage. Instructions in simple language.
"""
↓
Call Gemini API:
- Model: gemini-pro
- Temperature: 0.3 (low for consistency)
- Max tokens: 500
↓
Gemini Response:
"Metformin tablet, 500 milligrams. Take twice daily after meals."
```

**7. Safety Layer Validation**
```
Safety Layer: validate_output()
↓
Check for Prohibited Patterns:
- Regex: "you (have|are|should)" → Diagnosis/advice
- Regex: "(increase|decrease|stop) (taking|dosage)" → Dosage change
- Regex: "this (treats|cures|prevents)" → Treatment claim
- Check: Numerical values match original (500mg → 500mg ✓)
↓
Violations Detected: []
Safety Check: PASSED
↓
Inject Disclaimer:
"This is for informational purposes only. Follow your doctor's 
instructions. Do not change dosage without consulting your doctor."
```

**8. Store Result (If Consent Given)**
```
IF consent_to_store == true:
  INSERT INTO prescription_records (
    session_id,
    original_text,
    simplified_text,
    language,
    expires_at
  ) VALUES (
    {session_id},
    "Tab. Metformin 500mg BD PC",
    "Metformin tablet, 500 milligrams...",
    "en",
    NOW() + INTERVAL '30 days'
  )
ELSE:
  # Process in memory only, no database write
```

**9. Return Simplified Result**
```
Response: SimplificationResponse
{
  "original_text": "Tab. Metformin 500mg BD PC",
  "simplified_text": "Metformin tablet, 500 milligrams. Take twice daily after meals.",
  "disclaimer": "This is for informational purposes only...",
  "language": "en",
  "safety_check_passed": true
}
↓
Frontend displays result with disclaimer
Offers text-to-speech option
```

**10. Cleanup**
```
Delete temporary file: /tmp/prescriptions/rx_abc123.jpg
Clear in-memory data if no consent
Log transaction (without PII) for monitoring
```

### Error Handling in Data Flow

**OCR Failure:**
```
IF ocr_confidence < 0.7:
  Response: {
    "processing_status": "low_confidence",
    "message": "Image quality too low. Please upload a clearer image.",
    "suggestions": [
      "Ensure good lighting",
      "Avoid shadows and glare",
      "Keep camera steady"
    ]
  }
```

**Gemini API Failure:**
```
IF gemini_api_error:
  Response: {
    "processing_status": "ai_unavailable",
    "message": "Simplification service temporarily unavailable. Please try again.",
    "fallback": "Original text: {extracted_text}"
  }
```

**Safety Layer Rejection:**
```
IF safety_check_passed == false:
  Log violation with context
  Response: {
    "processing_status": "safety_check_failed",
    "message": "Unable to simplify this text safely. Please consult your doctor directly.",
    "violations": ["detected_diagnosis_language"]
  }
```

### Performance Considerations

- **OCR Processing**: ~5-8 seconds for typical prescription image
- **Gemini API Call**: ~2-3 seconds for text simplification
- **Safety Validation**: ~100-200ms for pattern matching
- **Total End-to-End**: ~8-12 seconds from upload to simplified result

### Security Considerations

- **File Upload**: Validate MIME types, scan for malicious content
- **Temporary Storage**: Auto-delete after processing (max 1 hour retention)
- **API Keys**: Store Gemini API key in environment variables, never in code
- **Rate Limiting**: Max 10 uploads per user per hour to prevent abuse
- **Data Encryption**: Encrypt prescription_records table at rest if consent given


## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Navigation Properties

**Property 1: Navigation responses contain required location fields**

*For any* valid navigation query, the system response should contain department name, floor number, and room number fields.

**Validates: Requirements 1.1**

**Property 2: Misspelled departments produce suggestions**

*For any* department name with character substitutions, deletions, or additions (edit distance ≤ 2), the system should return at least one department suggestion.

**Validates: Requirements 1.4**

**Property 3: Department database integrity**

*For all* departments in the database, each record should have non-null values for name, floor, and room_number fields.

**Validates: Requirements 1.5**

### Symptom Mapping Properties

**Property 4: Symptom inputs map to departments**

*For any* valid symptom input string, the system should return at least one department recommendation.

**Validates: Requirements 2.1**

**Property 5: Multiple relevant departments are all returned**

*For any* symptom input that matches multiple department mappings in the database, all matching departments should appear in the response.

**Validates: Requirements 2.3**

### Doctor Schedule Properties

**Property 6: Doctor search results contain schedule information**

*For any* doctor search query that returns results, each doctor record should contain schedule (timings) and room_number fields.

**Validates: Requirements 3.1**

**Property 7: Token generation produces unique identifiers**

*For any* two token booking requests for the same doctor on the same day, the generated token numbers should be distinct.

**Validates: Requirements 3.2**

**Property 8: Doctor availability matches schedule**

*For any* doctor with a defined schedule, the availability status at a given time should be "Available" if current time falls within a schedule slot, "Off-duty" if outside all slots, and "Busy" if within a slot but currently occupied.

**Validates: Requirements 3.3**

**Property 9: Doctor filters reduce result set**

*For any* doctor search with filters applied (department, specialization, availability), the filtered result set should be a subset of the unfiltered results.

**Validates: Requirements 3.5**

### Prescription Simplification Properties

**Property 10: OCR extraction produces text output**

*For any* valid prescription image upload, the OCR process should return extracted text with a confidence score.

**Validates: Requirements 4.1**

**Property 11: Extracted text produces simplified output**

*For any* extracted medical text that passes safety validation, the AI simplification should return non-empty simplified text.

**Validates: Requirements 4.2**

**Property 12: Numerical values are preserved in simplification**

*For any* prescription text containing numerical values (dosages, quantities), the simplified text should contain the exact same numerical values.

**Validates: Requirements 4.7**

**Property 13: Safety layer blocks prohibited content**

*For any* AI-generated text containing patterns matching diagnosis statements ("you have", "this indicates"), treatment recommendations ("you should take", "increase dosage"), or dosage changes ("take more", "stop taking"), the safety layer should reject the output and return safety_check_passed = false.

**Validates: Requirements 4.5, 8.1, 8.4**

**Property 14: Safety layer validates all AI outputs**

*For any* AI-generated content (prescription simplification, symptom guidance), the response should include a safety_check_passed field indicating validation occurred.

**Validates: Requirements 8.3**

### Medicine Lookup Properties

**Property 15: Medicine search results contain required fields**

*For any* medicine search that returns a result, the medicine record should contain generic_name and active_ingredients fields.

**Validates: Requirements 5.1**

**Property 16: Generic equivalents are complete**

*For any* medicine with equivalents in the medicine_equivalents table, all equivalent brand names should appear in the response.

**Validates: Requirements 5.3**

**Property 17: Medicine records include usage and side effects**

*For all* medicines in the database, each record should have non-empty common_uses and side_effects fields.

**Validates: Requirements 5.5**

### Medical Disclaimer Properties

**Property 18: Medical disclaimers are present in all health content**

*For any* response containing AI-generated health information (simplified prescriptions, symptom mappings, medicine information), the response should contain a disclaimer field with text matching the pattern "informational purposes only" or "not a diagnosis".

**Validates: Requirements 2.2, 4.3, 5.2, 8.2**

**Property 19: Disclaimers match user language**

*For any* response with a medical disclaimer, if the user's selected language is Hindi, the disclaimer should contain Hindi text; if English, it should contain English text.

**Validates: Requirements 10.5**

### Voice Interface Properties

**Property 20: Voice input produces text transcription**

*For any* voice input activation with valid audio, the speech-to-text process should return a text transcription string.

**Validates: Requirements 6.1**

**Property 21: Text responses include TTS capability**

*For any* text response from the system, the response object should include a flag or method indicating text-to-speech is available.

**Validates: Requirements 6.2**

### Data Privacy Properties

**Property 22: No persistent storage without consent**

*For any* prescription upload where consent_to_store = false, no record should be created in the prescription_records table.

**Validates: Requirements 7.1**

**Property 23: Consent enables encrypted storage**

*For any* prescription upload where consent_to_store = true, a record should be created in the prescription_records table with an expiration timestamp.

**Validates: Requirements 7.2**

**Property 24: Session end clears temporary data**

*For any* user session, when the session ends, all temporary prescription images and in-memory data associated with that session_id should be deleted.

**Validates: Requirements 7.4**

### Multi-Language Properties

**Property 25: UI strings have translations**

*For all* user-facing text strings in the application, translations should exist for both English ("en") and Hindi ("hi") language codes.

**Validates: Requirements 10.1**

**Property 26: Language preference persists**

*For any* user who selects a language preference, if they return in a new session with the same session identifier, the language preference should be restored.

**Validates: Requirements 10.2**

**Property 27: Simplified text matches requested language**

*For any* prescription simplification request with language parameter set to "hi", the simplified_text should contain Hindi characters; if set to "en", it should contain English text.

**Validates: Requirements 10.3**

### Performance Properties

**Property 28: Navigation queries respond within time limit**

*For any* navigation query, the response time from request to response should be less than 2000 milliseconds.

**Validates: Requirements 11.1**

**Property 29: OCR processing completes within time limit**

*For any* prescription image upload, the OCR extraction time should be less than 10000 milliseconds.

**Validates: Requirements 11.2**

**Property 30: High load triggers request queuing**

*For any* system state with concurrent requests exceeding capacity threshold, new requests should receive a queued status with estimated wait time.

**Validates: Requirements 11.5**

### Error Handling Properties

**Property 31: Error messages match user language**

*For any* error that occurs during processing, the error message should be in the user's selected language (English or Hindi).

**Validates: Requirements 12.1**

**Property 32: Errors produce log entries**

*For any* error that occurs in the system, a log entry should be created with timestamp, error type, and context information.

**Validates: Requirements 12.4**

## Error Handling

### Error Categories

**1. User Input Errors**
- Invalid file formats for prescription uploads
- Empty or malformed queries
- Unsupported languages

**Handling:**
- Return 400 Bad Request with descriptive error message
- Provide specific guidance on correct format
- Log error with user context (no PII)

**2. OCR Processing Errors**
- Low confidence extraction (< 0.7)
- Image quality too poor
- Unsupported image format

**Handling:**
- Return 422 Unprocessable Entity
- Provide specific guidance: "Please upload a clearer image with good lighting"
- Suggest retaking photo with tips
- Log OCR confidence scores

**3. AI Service Errors**
- Gemini API unavailable
- API rate limit exceeded
- API timeout

**Handling:**
- Return 503 Service Unavailable
- Display: "Simplification service temporarily unavailable. Please try again in a few minutes."
- Implement exponential backoff retry (3 attempts)
- Fall back to displaying original extracted text
- Log API errors with timestamps

**4. Safety Layer Violations**
- Prohibited content detected in AI output
- Numerical value mismatch
- Medical advice patterns found

**Handling:**
- Block output from reaching user
- Return 422 Unprocessable Entity
- Display: "Unable to simplify this text safely. Please consult your doctor directly."
- Log violation details (without PII) for review
- Alert system administrators for repeated violations

**5. Database Errors**
- Connection failures
- Query timeouts
- Data integrity violations

**Handling:**
- Return 500 Internal Server Error for unexpected failures
- Return 404 Not Found for missing records
- Display user-friendly message: "We're experiencing technical difficulties. Please try again."
- Log full error stack trace
- Implement database connection pooling and retry logic

**6. Voice Interface Errors**
- Speech recognition failure
- Low confidence transcription
- Unsupported browser

**Handling:**
- Automatically fall back to text input
- Display: "Voice input not available. Please type your query."
- Provide visual indicator of fallback mode
- Log browser compatibility issues

### Error Response Format

All API errors follow this consistent format:

```json
{
  "error": {
    "code": "OCR_LOW_CONFIDENCE",
    "message": "Image quality too low. Please upload a clearer image.",
    "details": {
      "confidence": 0.65,
      "threshold": 0.70
    },
    "suggestions": [
      "Ensure good lighting",
      "Avoid shadows and glare",
      "Keep camera steady"
    ],
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

### Retry Logic

**OCR Processing:**
- Automatic retry: 2 attempts with 1-second delay
- User-initiated retry: Unlimited with rate limiting (max 10 per hour)

**AI Simplification:**
- Automatic retry: 3 attempts with exponential backoff (1s, 2s, 4s)
- Fallback: Display original text if all retries fail

**Database Queries:**
- Automatic retry: 3 attempts with 500ms delay
- Connection pool: Maintain 5-10 active connections

### Logging Strategy

**Log Levels:**
- **ERROR**: System failures, safety violations, API errors
- **WARN**: Low confidence OCR, retry attempts, rate limiting
- **INFO**: Successful requests, performance metrics
- **DEBUG**: Detailed request/response data (development only)

**Log Format:**
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "service": "prescription_service",
  "error_code": "SAFETY_VIOLATION",
  "message": "Prohibited content detected",
  "context": {
    "prescription_id": "rx_abc123",
    "violation_type": "diagnosis_language",
    "user_session": "session_xyz"
  }
}
```

**Privacy in Logs:**
- Never log prescription text or medical content
- Never log user names or identifiable information
- Log only session IDs and request IDs for tracing
- Rotate logs daily, retain for 30 days

## Testing Strategy

### Dual Testing Approach

HealthSathi requires both **unit tests** and **property-based tests** for comprehensive coverage. These approaches are complementary:

- **Unit tests** verify specific examples, edge cases, and integration points
- **Property-based tests** verify universal properties across all inputs

### Unit Testing

**Focus Areas:**
- Specific examples demonstrating correct behavior
- Edge cases (empty inputs, boundary values, special characters)
- Error conditions (API failures, invalid data, timeouts)
- Integration points between components (API endpoints, database queries)

**Example Unit Tests:**
- Test that searching for "Cardiology" returns the Cardiology department
- Test that uploading a non-image file returns 400 error
- Test that unmappable symptom "xyz123" recommends General Medicine
- Test that home page contains the medical disclaimer
- Test that medicine "Crocin" returns "Paracetamol" as generic name

**Unit Test Framework:**
- **Frontend**: Vitest + React Testing Library
- **Backend**: pytest + pytest-asyncio
- **Coverage Target**: 80% code coverage

### Property-Based Testing

**Focus Areas:**
- Universal properties that hold for all inputs
- Comprehensive input coverage through randomization
- Invariants that must be maintained
- Round-trip properties (serialization, parsing)

**Configuration:**
- **Minimum 100 iterations per property test** (due to randomization)
- **Library**: Hypothesis (Python) for backend, fast-check (TypeScript) for frontend
- **Tag Format**: Each test must include a comment:
  ```python
  # Feature: health-sathi, Property 12: Numerical values are preserved in simplification
  ```

**Example Property Tests:**

```python
# Feature: health-sathi, Property 12: Numerical values are preserved in simplification
@given(prescription_text=st.text(min_size=10))
def test_numerical_preservation(prescription_text):
    """For any prescription text containing numbers, 
    simplified text should contain the same numbers."""
    numbers_in_original = extract_numbers(prescription_text)
    simplified = simplify_prescription(prescription_text)
    numbers_in_simplified = extract_numbers(simplified)
    assert numbers_in_original == numbers_in_simplified

# Feature: health-sathi, Property 18: Medical disclaimers are present
@given(symptom=st.text(min_size=5, max_size=100))
def test_disclaimer_presence(symptom):
    """For any symptom input, response should contain disclaimer."""
    response = map_symptom_to_department(symptom)
    assert "disclaimer" in response
    assert "not a diagnosis" in response["disclaimer"].lower()

# Feature: health-sathi, Property 7: Token generation produces unique identifiers
@given(doctor_id=st.uuids(), num_requests=st.integers(min_value=2, max_value=10))
def test_unique_tokens(doctor_id, num_requests):
    """For any doctor, multiple token requests should produce unique tokens."""
    tokens = [book_token(doctor_id) for _ in range(num_requests)]
    token_numbers = [t["token_number"] for t in tokens]
    assert len(token_numbers) == len(set(token_numbers))  # All unique
```

**Property Test Coverage:**
- All 32 correctness properties defined in this document
- Each property maps to at least one property-based test
- Tests run with minimum 100 iterations to ensure statistical coverage

### Integration Testing

**API Integration Tests:**
- Test complete request/response cycles for all endpoints
- Test authentication and authorization flows
- Test rate limiting and request queuing
- Test error handling across service boundaries

**External Service Integration:**
- Mock Gemini API responses for consistent testing
- Mock EasyOCR for faster test execution
- Test actual API integration in staging environment

**Database Integration:**
- Test with actual PostgreSQL/SQLite database
- Test transaction rollback on errors
- Test concurrent access and locking

### End-to-End Testing

**User Flows:**
- Complete prescription upload → OCR → simplification flow
- Complete symptom input → department recommendation flow
- Complete doctor search → token booking flow
- Voice input → text processing → voice output flow

**Tools:**
- Playwright for browser automation
- Test in Chrome, Firefox, Safari
- Test on mobile viewports (375px, 768px, 1024px)

### Performance Testing

**Load Testing:**
- Simulate 100 concurrent users
- Test OCR processing under load
- Test database query performance
- Measure API response times

**Tools:**
- Locust for load testing
- pytest-benchmark for Python performance tests

**Targets:**
- Navigation queries: < 2 seconds (Property 28)
- OCR processing: < 10 seconds (Property 29)
- API endpoints: < 500ms for non-AI operations

### Security Testing

**Areas:**
- File upload validation (malicious files, oversized files)
- SQL injection prevention (parameterized queries)
- XSS prevention (input sanitization)
- API rate limiting (max 10 uploads per hour per user)
- Data encryption (prescription records at rest)

**Tools:**
- OWASP ZAP for vulnerability scanning
- Bandit for Python security linting
- npm audit for frontend dependencies

### Test Organization

**Backend Tests:**
```
tests/
├── unit/
│   ├── test_navigation_service.py
│   ├── test_symptom_mapping.py
│   ├── test_prescription_service.py
│   ├── test_safety_layer.py
│   └── test_medicine_lookup.py
├── property/
│   ├── test_navigation_properties.py
│   ├── test_prescription_properties.py
│   ├── test_safety_properties.py
│   └── test_disclaimer_properties.py
├── integration/
│   ├── test_api_endpoints.py
│   ├── test_database_operations.py
│   └── test_external_services.py
└── e2e/
    └── test_user_flows.py
```

**Frontend Tests:**
```
src/
├── components/
│   ├── VoiceInterface.test.tsx
│   ├── PrescriptionUpload.test.tsx
│   └── NavigationSearch.test.tsx
└── __tests__/
    ├── properties/
    │   └── ui-properties.test.ts
    └── integration/
        └── user-flows.test.ts
```

### Continuous Integration

**CI Pipeline:**
1. Lint code (ESLint, Pylint)
2. Run unit tests (fast feedback)
3. Run property tests (100 iterations each)
4. Run integration tests
5. Check code coverage (minimum 80%)
6. Run security scans
7. Build Docker images
8. Deploy to staging

**Test Execution Time:**
- Unit tests: ~2 minutes
- Property tests: ~5 minutes
- Integration tests: ~3 minutes
- E2E tests: ~10 minutes
- **Total**: ~20 minutes

### Test Data Management

**Fixtures:**
- Sample prescription images (clear, blurry, handwritten)
- Sample medical text (various languages, formats)
- Sample hospital data (departments, doctors, schedules)
- Sample medicine database (brand names, generics)

**Data Generation:**
- Use Hypothesis strategies for property tests
- Use Faker for realistic test data
- Use factory_boy for database fixtures

### Monitoring and Observability

**Metrics to Track:**
- API response times (p50, p95, p99)
- OCR success rate and confidence scores
- Safety layer violation rate
- Error rates by category
- User session duration

**Tools:**
- Prometheus for metrics collection
- Grafana for visualization
- Sentry for error tracking
- CloudWatch for AWS infrastructure (if deployed)

### Testing Constraints

**What NOT to Test:**
- Third-party library internals (Web Speech API, EasyOCR)
- Browser-specific rendering (rely on CSS framework)
- Network latency (mock external calls)
- Actual medical accuracy (rely on Gemini API and safety layer)

**Testing Philosophy:**
- Test behavior, not implementation
- Test at the appropriate level (unit vs integration)
- Avoid brittle tests (don't test exact text, test patterns)
- Maintain fast test execution (mock slow operations)
- Keep tests independent (no shared state)
