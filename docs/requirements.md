# Requirements Document

## Introduction

HealthSathi is a voice-first AI-powered web application designed to improve accessibility and clarity in public hospitals. The system assists patients and attendants in navigating hospital facilities, understanding prescriptions and medical reports in simple language, checking doctor availability, and receiving contextual educational health guidance.

The system is strictly assistive and informational. It does not provide diagnosis, treatment, dosage recommendations, or medical decision-making. The target users are patients (often elderly or with low digital literacy), patient attendants, and visitors in public hospitals.

## Glossary

- **HealthSathi_System**: The complete web application including frontend, backend, AI processing, and voice interfaces
- **User**: A patient, patient attendant, or hospital visitor interacting with the system
- **Prescription_Image**: A photograph or scanned image of a medical prescription
- **Medical_Report**: A laboratory test result or diagnostic report document
- **Simplified_Text**: Medical terminology translated into plain language (English or Hindi)
- **Navigation_Query**: A user request to locate a department, room, or facility within the hospital
- **Symptom_Input**: User-provided description of health symptoms for department guidance
- **Doctor_Schedule**: Information about doctor availability including timings, room numbers, and specializations
- **Medicine_Database**: Publicly available drug information including brand names, generic equivalents, and active ingredients
- **Medical_Disclaimer**: A mandatory warning stating the system does not provide medical advice, diagnosis, or treatment
- **Voice_Interface**: Speech-to-text and text-to-speech capabilities for hands-free interaction
- **Safety_Layer**: AI output validation system that prevents hallucinated dosages or treatment recommendations

## Requirements

### Requirement 1: Hospital Navigation System

**User Story:** As a hospital visitor, I want to find departments and rooms using text or voice input, so that I can navigate the hospital facility without getting lost.

#### Acceptance Criteria

1. WHEN a User provides a Navigation_Query, THE HealthSathi_System SHALL return the floor number, department name, and room location
2. WHEN a User requests directions via voice, THE HealthSathi_System SHALL accept speech input and convert it to text for processing
3. WHEN navigation results are displayed, THE HealthSathi_System SHALL provide both text and voice output options
4. WHEN a department name is ambiguous or misspelled, THE HealthSathi_System SHALL suggest the closest matching department names
5. THE HealthSathi_System SHALL maintain a static floor map database with all departments and room numbers

### Requirement 2: Symptom-to-Department Guidance

**User Story:** As a patient with limited medical knowledge, I want to describe my symptoms and receive guidance on which department to visit, so that I can get appropriate care without confusion.

#### Acceptance Criteria

1. WHEN a User provides a Symptom_Input, THE HealthSathi_System SHALL map it to the most relevant hospital department using rule-based logic
2. WHEN displaying department recommendations, THE HealthSathi_System SHALL include the Medical_Disclaimer stating "This is not a diagnosis"
3. WHEN multiple departments are relevant, THE HealthSathi_System SHALL list all applicable departments with brief explanations
4. THE HealthSathi_System SHALL support symptom input in both English and Hindi languages
5. WHEN a symptom cannot be mapped to any department, THE HealthSathi_System SHALL recommend visiting the General Medicine department

### Requirement 3: Doctor Availability Information

**User Story:** As a patient attendant, I want to check doctor schedules and room numbers, so that I can plan visits and avoid unnecessary waiting.

#### Acceptance Criteria

1. WHEN a User searches for a doctor by name or specialization, THE HealthSathi_System SHALL display the Doctor_Schedule including timings and room numbers
2. WHEN a User requests an appointment token, THE HealthSathi_System SHALL generate a mock token number with estimated wait time
3. THE HealthSathi_System SHALL display doctor availability status (Available, Busy, Off-duty) based on current time
4. WHEN no doctors are available in a specialization, THE HealthSathi_System SHALL display the next available time slot
5. THE HealthSathi_System SHALL allow filtering doctors by department, specialization, and availability

### Requirement 4: Prescription and Report Simplification

**User Story:** As a patient with low health literacy, I want to upload my prescription or medical report and receive an explanation in simple language, so that I can understand my medications and test results.

#### Acceptance Criteria

1. WHEN a User uploads a Prescription_Image, THE HealthSathi_System SHALL extract text using OCR technology
2. WHEN medical text is extracted, THE HealthSathi_System SHALL process it through the AI simplification layer to generate Simplified_Text
3. WHEN displaying Simplified_Text, THE HealthSathi_System SHALL append the Medical_Disclaimer to every response
4. THE HealthSathi_System SHALL support simplification output in both English and Hindi languages
5. WHEN the Safety_Layer detects potential hallucinated dosages or treatment advice, THE HealthSathi_System SHALL reject the AI output and display an error message
6. WHEN OCR extraction fails or produces low-confidence results, THE HealthSathi_System SHALL prompt the User to re-upload a clearer image
7. THE HealthSathi_System SHALL simplify technical medical terminology into plain language without altering numerical values from the original document

### Requirement 5: Medicine Information Lookup

**User Story:** As a patient seeking cost-effective alternatives, I want to search for medicine brand names and find generic equivalents, so that I can make informed decisions about my prescriptions.

#### Acceptance Criteria

1. WHEN a User searches for a medicine brand name, THE HealthSathi_System SHALL retrieve information from the Medicine_Database including generic name and active ingredients
2. WHEN displaying medicine information, THE HealthSathi_System SHALL include the Medical_Disclaimer stating this is informational only and not substitution advice
3. THE HealthSathi_System SHALL display multiple generic equivalents when available for a brand name medicine
4. WHEN a medicine is not found in the database, THE HealthSathi_System SHALL display a message indicating no information is available
5. THE HealthSathi_System SHALL show medicine information including typical uses and common side effects from publicly available sources

### Requirement 6: Voice Interface

**User Story:** As an elderly patient with limited digital literacy, I want to interact with the system using voice commands, so that I can access information without typing.

#### Acceptance Criteria

1. WHEN a User activates voice input, THE HealthSathi_System SHALL capture speech and convert it to text using speech-to-text technology
2. WHEN the Voice_Interface produces text output, THE HealthSathi_System SHALL provide an option to read it aloud using text-to-speech
3. WHEN voice recognition fails or produces low-confidence results, THE HealthSathi_System SHALL fall back to text input mode
4. THE HealthSathi_System SHALL support voice input in both English and Hindi languages
5. WHEN reading simplified reports aloud, THE HealthSathi_System SHALL use clear pronunciation and appropriate pacing for comprehension

### Requirement 7: Data Privacy and Security

**User Story:** As a patient concerned about privacy, I want my health data to be processed securely without unauthorized storage, so that my medical information remains confidential.

#### Acceptance Criteria

1. WHEN a User uploads a Prescription_Image or Medical_Report, THE HealthSathi_System SHALL process it in memory without persistent storage unless explicit consent is provided
2. WHEN explicit consent is provided, THE HealthSathi_System SHALL store health data with encryption and user-specific access controls
3. THE HealthSathi_System SHALL not share User health data with third parties without explicit consent
4. WHEN a User session ends, THE HealthSathi_System SHALL clear all temporary health data from memory
5. THE HealthSathi_System SHALL comply with applicable data protection regulations for health information

### Requirement 8: Safety Constraints and Medical Disclaimers

**User Story:** As a healthcare administrator, I want the system to never provide medical advice or diagnosis, so that we avoid legal liability and ensure patient safety.

#### Acceptance Criteria

1. THE HealthSathi_System SHALL never output diagnosis, treatment plans, or dosage change recommendations
2. WHEN displaying any AI-generated health information, THE HealthSathi_System SHALL append the Medical_Disclaimer
3. THE HealthSathi_System SHALL implement the Safety_Layer to validate all AI outputs before displaying to Users
4. WHEN the Safety_Layer detects medical advice or diagnostic content, THE HealthSathi_System SHALL block the output and log the incident
5. THE HealthSathi_System SHALL display a prominent disclaimer on the home page stating the system is for informational purposes only

### Requirement 9: Mobile-First Responsive Interface

**User Story:** As a patient using a mobile phone in the hospital, I want the interface to work smoothly on my device, so that I can access information on the go.

#### Acceptance Criteria

1. WHEN a User accesses the system on a mobile device, THE HealthSathi_System SHALL display a responsive interface optimized for small screens
2. THE HealthSathi_System SHALL support touch gestures for navigation and interaction on mobile devices
3. WHEN network connectivity is poor, THE HealthSathi_System SHALL display loading indicators and graceful error messages
4. THE HealthSathi_System SHALL maintain readability with appropriate font sizes and contrast ratios on mobile screens
5. WHEN a User rotates their device, THE HealthSathi_System SHALL adapt the layout to portrait or landscape orientation

### Requirement 10: Multi-Language Support

**User Story:** As a patient who speaks Hindi, I want to interact with the system in my preferred language, so that I can understand information clearly.

#### Acceptance Criteria

1. THE HealthSathi_System SHALL support user interface text in both English and Hindi languages
2. WHEN a User selects a language preference, THE HealthSathi_System SHALL persist the choice across sessions
3. THE HealthSathi_System SHALL provide AI-generated Simplified_Text in the User's selected language
4. WHEN voice input is used, THE HealthSathi_System SHALL detect and process the spoken language (English or Hindi)
5. THE HealthSathi_System SHALL display the Medical_Disclaimer in the User's selected language

### Requirement 11: System Performance and Reliability

**User Story:** As a hospital administrator, I want the system to respond quickly and reliably, so that patients receive timely assistance.

#### Acceptance Criteria

1. WHEN a User submits a Navigation_Query, THE HealthSathi_System SHALL return results within 2 seconds
2. WHEN processing a Prescription_Image with OCR, THE HealthSathi_System SHALL complete extraction within 10 seconds
3. WHEN the AI simplification service is unavailable, THE HealthSathi_System SHALL display an error message and suggest trying again later
4. THE HealthSathi_System SHALL maintain 99% uptime during hospital operating hours
5. WHEN system load is high, THE HealthSathi_System SHALL queue requests and display estimated wait times to Users

### Requirement 12: Error Handling and User Feedback

**User Story:** As a user encountering an error, I want to receive clear feedback about what went wrong, so that I can take corrective action.

#### Acceptance Criteria

1. WHEN an error occurs during processing, THE HealthSathi_System SHALL display a user-friendly error message in the selected language
2. WHEN OCR extraction produces unclear results, THE HealthSathi_System SHALL prompt the User with specific guidance to improve image quality
3. WHEN voice input is not understood, THE HealthSathi_System SHALL ask the User to repeat or rephrase their query
4. THE HealthSathi_System SHALL log all errors with timestamps and context for debugging purposes
5. WHEN a critical system component fails, THE HealthSathi_System SHALL display a maintenance message and provide alternative contact information
