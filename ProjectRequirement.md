# 🛡️ High-Level System Requirements: LUMINA
**Project Stage:** Pre-Alpha / Patent-Pending  
**Security Tier:** Sovereign Intelligence (Zero-Cloud)

---

## 1. Project Vision
Lumina is a multi-tenant, on-device AI assistant for iPadOS. It provides professional-grade document intelligence, semantic reconciliation, and lifecycle management while maintaining absolute data sovereignty through local-first processing on M-series silicon.

## 2. Multi-Tenant Architecture (The "Sovereign Vault" Logic)
Lumina must maintain a strict "Wall of Separation" between different clients or projects to prevent data leakage and AI cross-contamination.

* **Requirement 2.1: Contextual Isolation.** The application must support a "Global Context Switcher" that mounts/unmounts specific Client Vaults.
* **Requirement 2.2: Directory Siloing.** Each Tenant (Client) must have a dedicated, unique directory within the iPadOS Sandbox.
* **Requirement 2.3: Ephemeral Inference.** When a user switches from "Client A" to "Client B," all active vector embeddings and LLM context windows associated with "Client A" must be purged from volatile memory (RAM).

## 3. Security & Data Encryption
Lumina leverages the Apple Silicon "Secure Enclave" to provide enterprise-grade privacy for small business data.

| Component | Requirement | Technology |
| :--- | :--- | :--- |
| **Data at Rest** | All PDFs and Databases must be encrypted per-vault. | AES-256 (File-level) |
| **Key Management** | Encryption keys must be tied to the device's hardware. | Apple Secure Enclave |
| **Biometrics** | Per-vault "Deep Lock" for sensitive clients. | FaceID / TouchID |
| **Data Egress** | **Zero.** The app must never transmit document data. | Network Sandbox Rules |

## 4. Functional Requirements (The "Delta Engine")

### 4.1 Semantic Comparison
* The system shall perform "Intent-Based" comparison between document versions.
* It must align clauses using high-dimensional vector similarity scores rather than simple character diffing.

### 4.2 On-Device RAG (Retrieval-Augmented Generation)
* The system shall index documents locally using Apple's `NaturalLanguage` and `CoreML` frameworks.
* Users must be able to query their "Vault" via natural language with 100% on-device inference.

### 4.3 Deterministic Source Anchoring
* Every AI-generated insight or summary must be mapped to specific $(x, y)$ coordinates in the source PDF.
* The UI must support a "Verification Link" that instantly scrolls the PDF viewer to the relevant paragraph.

## 5. Technical Stack & Constraints (2026 Standards)

* **Hardware Target:** iPad Pro / iPad Air with M1, M2, M4, or M5 silicon.
* **OS Target:** iPadOS 18.0 or later (Optimized for iPadOS 19 Apple Intelligence SDK).
* **Database:** SwiftData for relational metadata; custom binary stores for vector embeddings.
* **OCR Engine:** VisionKit (On-device spatial text extraction).
* **LLM Engine:** Local execution via `CoreML` / `MLX` / `Apple Intelligence Writing Tools`.

## 6. UI/UX Standard: "Clean Office" 
The interface must adhere to the professional Gold/White aesthetic to ensure high trust and visibility.

* **Primary Palette:** Studio White (#FFFFFF), Lumina Gold Gradient (#A67C00), Charcoal Gray (#333333).
* **Interaction:** Pencil-First design. Support for Apple Pencil annotations on top of the AI-intelligence layer.
* **Layout:** Native iPadOS Sidebar architecture with multi-window support.

---

## 7. Compliance & Audit
* **Audit Trail:** The app shall maintain a local, encrypted log of all "Significant Insights" and "Drafted Responses" to allow users to review AI actions for professional liability purposes.
* **Regulatory Readiness:** Architecture is designed to exceed CCPA and GDPR requirements by never processing data in a third-party cloud.