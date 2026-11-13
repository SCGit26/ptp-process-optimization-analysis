# Procure-to-Pay (PTP) Process Bottleneck & AP Discrepancy Analysis
**Enterprise Optimization, BPMN 2.0 Process Redesign & System Integration**

---

## 📌 Executive Summary
In legacy Procure-to-Pay (PTP) environments, manual intake of unstructured invoice PDFs via email creates severe operational friction, high exception handling costs, and significant duplicate payment exposure. 

This project delivers an enterprise-grade process optimization analysis for Accounts Payable (AP) operations. By transitioning from manual intake to an **automated portal**, implementing **3-way matching engine logic**, and integrating **ServiceNow exception workflows**, this solution targets:
* **80%+ Touchless Processing Rate** for standard Purchase Order (PO) invoices.
* **75% Reduction in Cycle Time** (from 8 days down to < 48 hours).
* **Mitigation of $150K+ Annual Duplicate Payment Risk** via multi-attribute fuzzy matching.

---

## 📁 Repository Structure
```text
ptp-process-bottleneck-analysis/
├── docs/
│   ├── brd_functional_requirements.md   # Phase 3: BRD, User Stories (BDD), NFRs
│   └── data_integration_mapping.md      # Phase 4: PTP-to-ERP 15-Field Mapping & Payloads
├── diagrams/
│   ├── current_state_workflow.png       # Phase 2: As-Is BPMN 2.0 Process Diagram
│   └── future_state_workflow.png        # Phase 2: To-Be Automated BPMN 2.0 Diagram
└── README.md                            # Project Portfolio Summary
