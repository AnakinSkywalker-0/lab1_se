# 1. Requirements Specification — Food Bank Surplus Redistribution Platform

**Actors:** Donor Partner (restaurants / grocery stores), Shelter Coordinator (verified shelters). Supporting: Platform Admin, Push Notification Service, Scheduler.

## 1.1 Functional Requirements (5)

| ID | Type | Description | Priority | Acceptance Criteria | Rationale |
|----|------|-------------|----------|---------------------|-----------|
| **FR-001** | Functional | The system shall allow a Donor Partner to post a surplus food batch with item name, quantity, pickup address, a perishable **expiry countdown timer**, and **dietary tags** (vegan, vegetarian, halal, gluten-free, contains-nuts, etc.). | High | **Pass:** Posted batch appears on the claim board with a live countdown and its tags within 2 s. **Fail:** A batch is accepted without expiry time or with an expiry already in the past. | Core supply-side action. Without expiry data, food safety cannot be enforced. Dietary tags let shelters match the food to the people they serve. |
| **FR-002** | Functional | The system shall allow only **admin-verified** Shelter Coordinators to browse the claim board, with filters for dietary tag, distance, quantity, and time-to-expiry. | High | **Pass:** Verified shelter sees filtered, expiry-sorted results; unverified account gets read-denied (HTTP 403). **Fail:** Unverified user can view pickup addresses or reserve a batch. | Prevents misuse of donor addresses and food. Filtering reduces food wasted because the wrong shelter claims the wrong item. |
| **FR-003** | Functional | The system shall let a verified Shelter Coordinator **reserve a pickup window** for a batch, and shall issue a unique one-time **pickup verification code**. A batch can be held by only one shelter at a time. | High | **Pass:** Shelter reserves batch, window is locked, and the shelter receives a verification code; a second concurrent reservation attempt is rejected. **Fail:** Two shelters hold the same batch, or the window ends after batch expiry. | Prevents double-booking and gives donors proof of who is collecting. This is the central transaction of the platform. |
| **FR-004** | Functional | The system shall **automatically expire** a batch when its countdown reaches zero, remove it from the claim board, cancel any open reservation, and notify the affected parties. | High | **Pass:** Within 60 s of expiry the batch status = EXPIRED, it is hidden from the board, and donor and shelter receive a notice. **Fail:** An expired food batch remains active on the claim board. | Food-safety critical: redistributing expired food is a health and legal risk. |
| **FR-005** | Functional | The system shall allow the Donor Partner to **confirm pickup** by entering or scanning the shelter's verification code, mark the batch COLLECTED, and record it in a donation history with meals-saved totals. | Medium | **Pass:** Valid code marks batch COLLECTED and updates the donor's impact dashboard. **Fail:** An invalid, reused, or expired code is accepted. | Closes the loop, proves handover, and gives donors reporting (CSR / tax records) that encourages continued participation. |

## 1.2 Non-Functional Requirements (2)

| ID | Type | Description | Priority | Acceptance Criteria | Rationale |
|----|------|-------------|----------|---------------------|-----------|
| **NFR-001** | Performance & Scalability | The platform shall dispatch push notifications to all verified shelters within a **5 km radius** within **5 seconds (p95)** of a high-quantity perishable batch being listed, and sustain **1,000 concurrent users** and **100 postings/min** without degradation. | High | **Pass:** Load test (k6 / Locust) at 1,000 concurrent users shows p95 notification latency ≤ 5 s and API p95 ≤ 500 ms. **Fail:** Latency target missed under simulated peak load, or any shelter inside 5 km is not notified. | Perishable food loses value by the minute; fast alerts are what turn a listing into a rescue. Peaks occur at restaurant closing time. |
| **NFR-002** | Security & Reliability | All traffic shall use **TLS 1.2+**; data shall be protected with **role-based access control** (Donor / Shelter / Admin); verification codes shall be single-use, ≥ 8 characters, and expire with the reservation; the platform shall achieve **99.5% monthly availability**. | High | **Pass:** Penetration test (OWASP Top 10) finds no critical or high findings; code replay is rejected; uptime monitoring shows ≥ 99.5%. **Fail:** A shelter can read another shelter's reservation, or a used code is accepted a second time. | Protects shelters' and donors' data, prevents fraudulent pickups, and keeps the service dependable when time-sensitive food is posted. |

## 1.3 Traceability to Use Cases

| Requirement | Use Case(s) |
|---|---|
| FR-001 | Post Surplus Food Batch, Set Expiry Timer & Dietary Tags |
| FR-002 | Browse Claim Board, Verify Shelter Account, Register Account |
| FR-003 | Reserve Pickup Window, Generate Pickup Verification Code, Cancel Reservation |
| FR-004 | Auto-Expire Batch |
| FR-005 | Confirm Pickup via Code, Manage Batch Listing |
| NFR-001 | Send Proximity Push Alert (extends Post Surplus Food Batch) |
| NFR-002 | All use cases (cross-cutting) |
