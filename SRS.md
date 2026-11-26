# Software Requirements Specification (SRS)
## For PlantCare AI: Disease Diagnosis & Nearby Plant Medicine Availability Platform

**Standard:** IEEE Std 830-1998 Recommended Practice for Software Requirements Specifications
**Date of Release:** November 21, 2026

---

## Table of Contents
1. [Introduction](#1-introduction)
   - 1.1 Purpose
   - 1.2 Scope
   - 1.3 Definitions, Acronyms, and Abbreviations
   - 1.4 References
   - 1.5 Overview
2. [Overall Description](#2-overall-description)
   - 2.1 Product Perspective
   - 2.2 Product Functions
   - 2.3 User Classes and Characteristics
   - 2.4 Operating Environment
   - 2.5 Design and Implementation Constraints
   - 2.6 Assumptions and Dependencies
3. [Specific Requirements](#3-specific-requirements)
   - 3.1 External Interface Requirements
   - 3.2 Functional Requirements (Structured by Module)
   - 3.3 Non-Functional Requirements
   - 3.4 Database Requirements
4. [Verification and Validation Criteria](#4-verification-and-validation-criteria)
5. [Appendix & Traceability Matrix](#5-appendix--traceability-matrix)

---

## 1. Introduction

### 1.1 Purpose
The purpose of this Software Requirements Specification (SRS) is to establish a rigorous, comprehensive, and definitive technical specification for **PlantCare AI**. This document specifies the functional behavior, architectural constraints, external interface protocols, data schemas, performance metrics, and quality attributes necessary for production deployment and university academic evaluation.

### 1.2 Scope
PlantCare AI is an integrated agricultural intelligence and supply-chain logistics software platform. It enables smallholder farmers, agricultural extension workers, and commercial horticulturists to:
1. Capture or upload high-resolution leaf imagery to identify crop pathologies via deep computer vision.
2. Receive deterministic, agronomy-curated treatment prescriptions adhering to Central Insecticides Board & Registration Committee (CIBRC) regulations.
3. Geospatially locate registered agricultural input retailers within a 5 km to 50 km radius that possess verified, real-time in-stock inventory of the required remedies.
4. Reserve inventory for a 4-hour hold period via cryptographic pickup tokens to guarantee medicine availability during transit.

### 1.3 Definitions, Acronyms, and Abbreviations
- **API:** Application Programming Interface
- **BVA:** Boundary Value Analysis
- **CIBRC:** Central Insecticides Board & Registration Committee
- **CNN:** Convolutional Neural Network
- **DFD:** Data Flow Diagram
- **EP:** Equivalence Partitioning
- **FAO:** Food and Agriculture Organization
- **GIS:** Geographic Information System
- **IPM:** Integrated Pest Management
- **KLOC:** Thousands of Lines of Code
- **MTBF:** Mean Time Between Failures
- **PHI:** Pre-Harvest Interval (safety waiting period before crop harvest)
- **POS:** Point of Sale
- **PWA:** Progressive Web Application
- **SRS:** Software Requirements Specification
- **STLC:** Software Testing Life Cycle
- **VLM:** Vision-Language Model

### 1.4 References
- IEEE Std 830-1998: IEEE Recommended Practice for Software Requirements Specifications.
- ISO/IEC/IEEE 29148:2018: Systems and software engineering — Life cycle processes — Requirements engineering.
- CIBRC Guidelines: Compendium of Approved Agrochemicals and Formulations in India (2025/2026).
- PlantVillage Dataset (Hughes & Salathé, Penn State University).

### 1.5 Overview
The remainder of this document details the functional and operational characteristics of PlantCare AI. Section 2 establishes the product context, user profiles, and architectural boundaries. Section 3 delineates detailed functional and non-functional requirements. Section 4 specifies testing and validation criteria.

---

## 2. Overall Description

### 2.1 Product Perspective
PlantCare AI operates as an autonomous, self-contained multi-tier client-server system that interfaces with mobile camera devices, geospatial positioning hardware (GPS), and agricultural retailer inventory databases.

```
+---------------------------------------------------------------------------------------------------+
|                                        SYSTEM CONTEXT DIAGRAM                                     |
|                                                                                                   |
|  +------------------+         +-------------------------------+         +-----------------------+ |
|  |  Farmer / User   | <=====> |         PLANTCARE AI          | <=====> |  Agro-Dealer Retailer | |
|  |  (Mobile Device) |  HTTPS  |     APPLICATION PLATFORM      |  HTTPS  |  (Inventory & POS)    | |
|  +------------------+         +---------------+---------------+         +-----------------------+ |
|                                               |                                                   |
|                                               v                                                   |
|                                 +---------------------------+                                     |
|                                 | CIBRC Agronomic Database  |                                     |
|                                 | Regulatory Standards      |                                     |
|                                 +---------------------------+                                     |
+---------------------------------------------------------------------------------------------------+
```

### 2.2 Product Functions
- **Foliar Lesion Visual Diagnosis:** Multi-class classification of foliar diseases across solanaceous and cereal crops.
- **Lesion Severity Grading:** Calculation of infected surface area percentage categorized into Healthy, Mild, Moderate, and Severe.
- **Prescription Generation:** Synthesis of dual treatment plans (chemical contact/systemic agents vs. organic bio-controls).
- **Bounding-Box Spatial Pruning:** High-speed geographic filtering of registered retail stores.
- **Haversine Proximity Sorting:** Calculation of driving distance and transit time.
- **Real-Time Stock Querying & Reservation:** Locking of inventory units for verified transit periods.

### 2.3 User Classes and Characteristics
1. **Smallholder Farmer (Primary End-User):**
   - High domain knowledge in farming, moderate digital literacy.
   - Requires simple visual feedback, low-bandwidth resilience, and vernacular support.
2. **Agricultural Extension Worker / Agronomist:**
   - High technical literacy.
   - Uses advanced diagnostics, active ingredient tracking, and epidemiological reporting.
3. **Agro-Dealer / Retail Store Manager:**
   - Commercial input seller.
   - Manages inventory updates, price modifications, and reservation fulfillment.
4. **System Administrator:**
   - Manages CIBRC regulatory database updates, user authentication, and system monitoring.

### 2.4 Operating Environment
- **Server:** Linux Ubuntu 22.04 LTS / macOS, Python 3.10+, PostgreSQL 15 with PostGIS, Redis 7.0.
- **Client:** Modern web browsers (Chrome 100+, Safari 15+, Firefox 95+) or Android 8.0+ / iOS 14+ via Flutter/PWA.
- **Network Constraints:** Functional over 2G/3G low-bandwidth connections (payload size < 300KB compressed).

### 2.5 Design and Implementation Constraints
1. **Regulatory Compliance:** Prescriptions must strictly adhere to CIBRC-approved formulations; unregistered or banned chemicals cannot be recommended.
2. **Latency Constraints:** Total diagnosis-to-store turnaround latency must not exceed 1.0 second on standard broadband or 3.0 seconds on 3G cellular.
3. **Model Weight Footprint:** Edge vision models must not exceed 25MB to ensure compatibility with entry-level smartphones.

---

## 3. Specific Requirements

### 3.1 External Interface Requirements
- **User Interfaces:** Clean responsive mobile-first UI with camera capture button, visual severity thermometer, dual prescription toggle cards, and interactive store map.
- **Hardware Interfaces:** Mobile device camera (minimum 2.0 Megapixels, autofocus), GPS receiver (accuracy within 15 meters).
- **Software Interfaces:** RESTful HTTP/JSON APIs, PostgreSQL connection pool, Redis cache sockets.

### 3.2 Functional Requirements

#### Module 1: Image Acquisition & Preprocessing
- **FR-1.1:** System shall accept image inputs via multipart form file upload (`.jpg`, `.jpeg`, `.png`, `.webp`) or Base64 encoded payload.
- **FR-1.2:** System shall validate that file size does not exceed 10 Megabytes.
- **FR-1.3:** System shall normalize and resize image tensors to $224 \times 224$ pixels across 3 RGB channels with standard ImageNet mean and variance scaling.

#### Module 2: Computer Vision Pathology Classification
- **FR-2.1:** System shall execute inference against 38 crop-disease categories.
- **FR-2.2:** System shall compute softmax confidence probabilities for predicted classes.
- **FR-2.3:** System shall enforce a minimum confidence threshold of 0.70. Images scoring below 0.70 shall trigger an "Uncertain Diagnosis" fallback.
- **FR-2.4:** System shall estimate the percentage of necrotic lesion area to assign a discrete severity grade:
  - Healthy: $0.0\%$
  - Mild: $< 15.0\%$
  - Moderate: $15.0\% \le \text{severity} \le 40.0\%$
  - Severe: $> 40.0\%$

#### Module 3: Expert Agronomic Prescription
- **FR-3.1:** System shall retrieve verified chemical formulations mapped to the diagnosed pathogen.
- **FR-3.2:** System shall retrieve certified biological/organic alternatives.
- **FR-3.3:** Prescriptions shall include active chemical ingredient, concentration, trade brand examples, dosage per acre, water dilution ratios, and safety waiting periods (Pre-Harvest Interval in days).
- **FR-3.4:** System shall disallow synthetic chemical prescriptions when severity is classified as Healthy.

#### Module 4: Hyperlocal Geospatial Store Search
- **FR-4.1:** System shall accept farmer geographic coordinates (latitude, longitude) and target search radius (5.0 km to 100.0 km).
- **FR-4.2:** System shall execute $O(1)$ coordinate bounding-box rejection prior to computing spherical distances.
- **FR-4.3:** System shall compute exact great-circle distance using the numerically stable Haversine formula ($R = 6371.0 \text{ km}$).
- **FR-4.4:** System shall filter results to only display retail stores currently holding active inventory ($\text{stock} > 0$).
- **FR-4.5:** System shall sort results in ascending order of physical distance, breaking ties by unit price.

#### Module 5: Inventory Reservation & Locking
- **FR-5.1:** System shall permit authenticated farmers to reserve up to 20 units of an in-stock medicine.
- **FR-5.2:** System shall atomically decrement the store inventory upon reservation confirmation.
- **FR-5.3:** System shall generate a cryptographically unique reservation ID (`RES-XXXXXXXX`) and an alphanumeric pickup code (`PK-XXXXXX`).
- **FR-5.4:** System shall automatically expire reservations after 4 hours, restoring held quantities to active inventory if unclaimed.

---

### 3.3 Non-Functional Requirements

#### NFR-1: Performance & Latency
- Image preprocessing and model inference shall complete within 250 milliseconds.
- Geospatial inventory search across 5,000 store records shall execute within 150 milliseconds.
- End-to-end API roundtrip latency (excluding network transport) shall not exceed 500 milliseconds (p95).

#### NFR-2: Reliability & Availability
- The system backend shall maintain 99.9% uptime during cropping seasons.
- The system shall implement graceful degradation: if spatial search is unavailable, diagnosis and dosage instructions shall still be delivered immediately.

#### NFR-3: Security & Data Integrity
- All communications shall occur over HTTPS with TLS 1.3 encryption.
- Input data shall be sanitized against SQL injection, cross-site scripting (XSS), and malicious payload buffer overruns.
- PII (Farmer phone numbers) shall be hashed or encrypted at rest using AES-256.

#### NFR-4: Usability & Accessibility
- The mobile interface shall adhere to WCAG 2.1 Level AA accessibility guidelines.
- Touch target sizes for primary mobile actions shall be at least $48 \times 48$ density-independent pixels.

---

## 4. Verification and Validation Criteria

### Testing Selection Methods:
1. **Equivalence Partitioning (EP):** Partition inputs into valid image categories, corrupt byte streams, healthy leaf controls, and out-of-boundary GPS coordinates.
2. **Boundary Value Analysis (BVA):** Explicitly verify severity transitions at $0\%$, $14.9\%$, $15.0\%$, $39.9\%$, $40.0\%$, and radius boundaries at $0 \text{ km}$, $25 \text{ km}$, $100 \text{ km}$.
3. **Race Condition Stress Testing:** Simulate 50 concurrent reservation requests against an inventory of 2 units to verify atomic transaction locks.

### Defined Stop-Testing Criteria:
- **Test Execution:** 100% execution of all functional test cases with 0 failing tests.
- **Code Coverage:** Statement coverage $\ge 90\%$, branch coverage $\ge 85\%$.
- **Defect Thresholds:** 0 open P0 (Blocker) or P1 (Critical) defects; Defect Density $< 0.1$ defects/KLOC.
- **Performance SLA:** Sustained load of 200 concurrent requests with error rate $< 0.01\%$.

---

## 5. Traceability Matrix

| Requirement ID | Module | Implementation File | Verification Test Case |
| :--- | :--- | :--- | :--- |
| **FR-1.1 - 1.3** | Ingestion | `app/api/endpoints.py` | `test_api.py::test_diagnose_api_post` |
| **FR-2.1 - 2.4** | Vision Diagnosis | `app/services/diagnosis_service.py` | `test_diagnosis.py::test_diagnose_tomato_early_blight_default` |
| **FR-3.1 - 3.4** | Agronomic Prescription | `app/services/prescription_service.py` | `test_diagnosis.py::test_diagnose_healthy_crop_severity` |
| **FR-4.1 - 4.5** | Spatial Inventory | `app/services/spatial_inventory_service.py` | `test_spatial_inventory.py::test_nearby_medicines_within_radius` |
| **FR-5.1 - 5.4** | Stock Reservation | `app/services/spatial_inventory_service.py` | `test_spatial_inventory.py::test_create_reservation_success_and_stock_decrement` |

---
**End of Software Requirements Specification**
