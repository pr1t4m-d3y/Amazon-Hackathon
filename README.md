# HealthSathi 🏥

> **A Voice-First AI Companion for Public Hospital Patients**

**HealthSathi** is an AI-powered web application designed to improve accessibility and clarity in public hospitals. It helps patients navigate facilities, understand medical prescriptions in simple language, and access critical hospital information—all through a voice-first interface.

---

## 📚 Documentation & Architecture

This project follows a strict **Spec-Driven Development** approach. The complete blueprint, including safety protocols and system architecture, is available in the `docs/` directory:

| Document | Description |
| :--- | :--- |
| 📘 [**Requirements Specification**](docs/requirements.md) | Detailed user stories, MVP scope, and **Safety & Ethical Constraints**. |
| 🏗️ [**System Design**](docs/design.md) | Full system architecture, API endpoints (FastAPI), Database Schema, and AI Data Flow. |

---

## 🚀 Key Features (MVP)

* **🏥 Hospital Navigation:** Find departments and rooms with voice-guided directions.
* **💊 Prescription Simplifier:** Upload a photo of a prescription to get a simplified explanation (No dosage advice).
* **⏰ Medicine Reminders:** Manually set reminders for medicine timings and next doctor visits.
* **🥗 Health Context Assistant:** Ask lifestyle/dietary questions (e.g., "Can I eat dates?") and get answers based on public health data with strict disclaimers.
* **👨‍⚕️ Doctor Availability:** Check schedules, room numbers, and mock appointment tokens.
* **🗣️ Voice-First Interface:** Designed for elderly and low-digital-literacy users (English & Hindi support).
* **🛡️ Safety Layer:** A dedicated AI validation layer to prevent medical hallucinations and ensure safety.

## 🛠️ Tech Stack

* **Frontend:** React.js (Vite) + Tailwind CSS (Mobile-First)
* **Backend:** Python (FastAPI)
* **AI/ML:** Google Gemini API (Text Simplification), EasyOCR (Prescription Reading)
* **Database:** PostgreSQL / SQLite (MVP)

## ⚠️ Medical Disclaimer

**HealthSathi is an assistive informational tool only.**
It does not provide medical diagnosis, treatment plans, or dosage recommendations. All AI-generated content is passed through a strict validation layer and includes mandatory medical disclaimers.

---

### 🏃‍♂️ Getting Started

1.  **Clone the repository**
    ```bash
    git clone [https://github.com/pr1t4m-d3y/Amazon-Hackathon.git](https://github.com/pr1t4m-d3y/Amazon-Hackathon.git)
    cd Amazon-Hackathon
    ```

2.  **Review the Specs**
    Check [docs/design.md](docs/design.md) to understand the folder structure and API design before contributing.
