# Security Threat Model: WordVision

This document provides a comprehensive threat model for the WordVision application, employing the STRIDE methodology and addressing AI-specific risks.

---

## 1. Data Movement Map

The following diagram visualizes the flow of data between the user, the frontend, the FastAPI backend, and the various cloud and AI services used by the application.

    graph TD

    User[User] -->|Login/Auth Request| Cognito(AWS Cognito)
    User -->|Upload Book| Frontend("Frontend - Expo React Native")
    User -->|Select Text for Highlight/Visualize| Frontend
    User -->|View Highlights/Images| Frontend
    User -->|Regenerate/Edit Prompt| Frontend

    Frontend -->|User Credentials| Cognito
    Cognito -->|Authentication Token| Frontend
    Frontend -->|"ePub File Upload"| Backend("Backend - FastAPI")
    Frontend -->|Highlight Text Data| Backend
    Frontend -->|"Visualize Request (Text)"| Backend
    Frontend -->|Image Regeneration Prompt| Backend
    Frontend -->|Delete Image Request| Backend

    Backend -->|Store User Credentials| Cognito
    Backend -->|"Store ePub Files"| S3(AWS S3)
    Backend -->|Store Generated Images| S3
    Backend -->|"Store Metadata, Annotations"| Mongo(MongoDB)
    Backend -->|"Send Text for Image Generation"| HF(Hugging Face Spaces)
    Backend -->|Retrieve Generated Image URL| HF
    Backend -->|Retrieve ePub File Path| S3
    Backend -->|Retrieve Generated Image Path| S3
    Backend -->|Store/Retrieve User Data| Mongo

    HF -->|Image Generation Request| HF
    HF -->|Generated Image Data| Backend

    Mongo -->|User Metadata| Backend
    S3 -->|"ePubs & Generated Images"| Backend
---

## 2. STRIDE THREAT ANALYSIS

### **Spoofing**
*   **S1: Unauthorized Access via Stolen Credentials** (High)
    *   **Component:** AWS Cognito, Backend
    *   **Vector:** Phishing, brute-force, or compromised tokens.
*   **S2: Malicious User Impersonation** (Medium)
    *   **Component:** Frontend, Backend, MongoDB
    *   **Vector:** Exploiting weak session management or authorization logic.

### **Tampering**
*   **T1: Modification of Uploaded ePub Files** (High)
    *   **Component:** AWS S3, Backend
    *   **Vector:** Replacing legitimate ePub files with malicious XSS payloads.
*   **T2: Tampering with Generated Images** (Medium)
    *   **Component:** AWS S3, Backend
    *   **Vector:** Replacing legitimate images with harmful or misleading content.

### **Repudiation**
*   **R1: Inability to Trace Image Generation Requests** (Medium)
    *   **Component:** Backend, Hugging Face
    *   **Vector:** Lack of detailed logging on who initiated specific AI requests.

### **Information Disclosure**
*   **I1: Exposure of User Authentication Credentials** (High)
    *   **Component:** AWS Cognito
    *   **Vector:** Weak password policies or insecure credential storage.
*   **I2: Unauthorized Access to Files and Images** (High)
    *   **Component:** AWS S3, Backend
    *   **Vector:** Insecure S3 bucket policies or weak API access control.

### **Denial of Service**
*   **D1: Resource Exhaustion on Hugging Face Spaces** (High)
    *   **Component:** Hugging Face, Backend
    *   **Vector:** Overwhelming the AI service with excessive image requests.

### **Elevation of Privilege**
*   **E1: Bypassing Authorization for Sensitive Data** (High)
    *   **Component:** Backend
    *   **Vector:** Exploiting API vulnerabilities to access data belonging to other users.

---

## 3. AI-SPECIFIC RISKS

*   **Prompt Injection (PI1)**: Users injecting harmful instructions into text to bypass AI content filters. (High Severity)
*   **Insecure Output Handling (IOH1)**: AI generating images containing malicious content or steganographic payloads. (High Severity)
*   **Data Leakage (DL2)**: Inadvertent disclosure of user PII within AI-generated outputs. (Medium Severity)

---

## 4. REMEDIATION PLAN

1.  **Critical Priority**: Implement robust **Input Validation & Sanitization** for all ePub uploads and AI prompts.
2.  **High Priority**: Enforce **Strong Authentication (MFA)** via AWS Cognito and lock down **S3 Bucket Policies**.
3.  **Medium Priority**: Establish comprehensive **Logging & Monitoring** for all backend and AI API interactions.

---

## Conclusion

This threat model identifies critical security risks inherent in the **WordVision** architecture. By implementing this prioritized remediation plan, the development team can ensure a secure and resilient experience for users while leveraging the power of AI-driven features.
