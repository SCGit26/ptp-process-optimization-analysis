# Business Requirements Document (BRD)
## Project: PTP Process Optimization & Automated Invoice Exception Handling

### 1. Executive Summary & Business Case
* **Project Context:** The current Procure-to-Pay (PTP) intake process relies on unstructured email attachments, manual data entry, and memory-based duplicate checks. This creates severe operational bottlenecks and financial leakage.
* **Business Objective:** Implement an automated intake portal, automated 3-way matching (Invoice vs. PO vs. Goods Receipt), system-driven duplicate detection, and a ServiceNow exception routing engine.
* **Target Success Metrics:**
  * Achieve **80%+ Touchless Processing Rate** for standard Purchase Order (PO) invoices.
  * Reduce overall invoice processing cycle time by **75%** (from 8 days down to under 2 days).
  * Eliminate **$150K+ in annual duplicate payment exposure** through automated Tax ID and invoice metadata matching.


### 2. Functional Requirements

#### 2.1 Intake & Data Extraction
* **FR-01 (Mandatory Validation):** The system shall mandate entry of `Vendor Tax ID`, `PO Number`, `Invoice Number`, and `Invoice Gross Amount` at the point of portal submission.
* **FR-02 (File Attachment Parsing):** For legacy email submissions, the system shall leverage OCR/data extraction to parse PDF metadata into structured JSON payloads.

#### 2.2 Validation & 3-Way Matching Engine
* **FR-03 (Automated 3-Way Match):** The ERP engine shall cross-reference the incoming `Invoice Line Items`, `PO Line Items`, and `Goods Receipt (GR) Quantities`.
* **FR-04 (Tolerance Checks):** The system shall auto-approve invoices where unit price variance is 2% or less of the contracted PO line price.

#### 2.3 Risk & Duplicate Detection
* **FR-05 (Multi-Attribute Duplicate Flag):** The system shall flag a submission as a "Potential Duplicate" if it shares an exact `Vendor Tax ID` + `Invoice Gross Amount` and matches the `Invoice Number` (ignoring case, spaces, and special character suffixes like `-REV` or `_v2`).

#### 2.4 Exception Management & Workflow
* **FR-06 (ServiceNow Incident Creation):** When a 3-way match or duplicate check fails, the system shall automatically generate a ServiceNow exception ticket populated with the failure code (e.g., `ERR_PRICE_MISMATCH` or `ERR_POSSIBLE_DUPLICATE`).


### 3. Agile User Stories & Acceptance Criteria

#### User Story 1: Automated Duplicate Flagging
**As an** Accounts Payable Specialist,  
**I want** the ERP engine to automatically flag invoices matching historical Tax IDs, amounts, and fuzzy invoice numbers within a 90-day window,  
**So that** I can prevent duplicate payments before funds are disbursed.

* **Acceptance Criteria (Given-When-Then):**
  * **Given** an invoice is submitted with Tax ID `12-3456789`, Amount `$5,000.00`, and Invoice Number `INV-1002-REV`,
  * **When** an invoice already exists in the Vendor History DB with Tax ID `12-3456789`, Amount `$5,000.00`, and Invoice Number `INV1002`,
  * **Then** the system must hold the payment, set the invoice status to `PENDING_AUDIT`, and generate a ServiceNow task assigned to the AP Risk Queue with risk confidence score `HIGH`.

---

#### User Story 2: Touchless 3-Way Match Approval
**As a** Finance Manager,  
**I want** invoices that match the PO and Goods Receipt within tolerance to be auto-approved,  
**So that** my team only spends time reviewing exceptions.

* **Acceptance Criteria (Given-When-Then):**
  * **Given** an incoming invoice tied to `PO-88392`,
  * **When** `Invoice Quantity` less than or equal to `Goods Receipt Received Qty` AND `Invoice Unit Price` equals `PO Unit Price`,
  * **Then** the system will set invoice status to `APPROVED_FOR_PAYMENT` without triggering human manual review.

---

#### User Story 3: Exception Route Assignment in ServiceNow
**As an** AP Exception Handler,  
**I want** price mismatch exceptions automatically routed to the Requisitioner rather than general AP queues,  
**So that** resolution delays are minimized.

* **Acceptance Criteria (Given-When-Then):**
  * **Given** an invoice fails matching due to `ERR_PRICE_MISMATCH`,
  * **When** the ServiceNow ticket is created,
  * **Then** the ticket `Assignment Group` must default to the PO Requisitioner's Department Manager, and set the SLA resolution timer to `48 Hours`.


### 4. Non-Functional Requirements (NFRs)
* **Performance:** The ERP validation engine must process incoming portal payloads and return validation status within less than 10 seconds.
* **Auditability:** All manual overrides performed by AP Specialists on duplicate flags must log `User ID`, `Timestamp`, and `Justification Reason` in an unalterable audit log.
* **Security & Compliance:** Vendor financial data (bank details and Tax IDs) must be encrypted.