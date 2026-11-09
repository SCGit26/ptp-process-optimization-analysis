# Data Integration & Field Mapping Matrix
## Project: PTP Automated Invoice Intake to Enterprise ERP Payload

### 1. Overview & Integration Context
* **Source System:** Automated Vendor Portal / OCR Parsing Engine (JSON Inbound Payload)
* **Target System:** Enterprise ERP (SAP S/4HANA / Oracle Financials AP Module)
* **Integration Middleware:** RESTful API / Enterprise Service Bus (ESB)
* **Error Queue:** ServiceNow Incident Management System

---

### 2. PTP-to-ERP Data Mapping Table (15 Enterprise Fields)

| # | Target Field Name (ERP) | Target Table | Data Type | Requirement | Source Payload Field (Inbound) | Transformation / Business Rules | Validation / Error Condition |
|---|---|---|---|---|---|---|---|
| 1 | `VENDOR_TAX_ID` | `LFA1` (Vendor Master) | VARCHAR(11) | Mandatory | `supplier_tax_identifier` | Strip hyphens/spaces. Match against active Vendor Master. | If missing or not found in ERP: Trigger `ERR_VENDOR_UNKNOWN`. |
| 2 | `INVOICE_NUM` | `RBKP` (Header) | VARCHAR(16) | Mandatory | `invoice_header_number` | Trim whitespace; convert to uppercase; strip suffixes (`-REV`, `_v2`). | If `VENDOR_TAX_ID` + normalized `INVOICE_NUM` exists: Flag `ERR_DUPLICATE_INV`. |
| 3 | `INVOICE_DATE` | `RBKP` (Header) | DATE | Mandatory | `document_issue_date` | Convert ISO string (`YYYY-MM-DD`) to ERP standard (`YYYYMMDD`). | If future-dated > 30 days: Trigger `ERR_DATE_OUT_OF_RANGE`. |
| 4 | `GROSS_AMOUNT` | `RBKP` (Header) | DECIMAL(13,2) | Mandatory | `total_amount_due` | Cast to 2-decimal float; enforce positive value. | If $\le 0.00$: Trigger `ERR_INVALID_AMOUNT`. |
| 5 | `CURRENCY_CODE` | `RBKP` (Header) | VARCHAR(3) | Mandatory | `billing_currency` | Standardize to ISO 4217 standard (e.g., USD, EUR, GBP). | If currency code invalid: Trigger `ERR_CURRENCY_MISMATCH`. |
| 6 | `PO_NUMBER` | `EKKO` (PO Header) | VARCHAR(10) | Conditional | `purchase_order_reference` | Left-pad with zeros to match 10-digit ERP PO format. | If PO status != `OPEN`: Trigger `ERR_PO_CLOSED_OR_INVALID`. |
| 7 | `PO_LINE_ITEM_NUM` | `RSEG` (Item) | NUMERIC(5) | Mandatory | `line_item_details[].po_line` | Direct map to line items array element. | If line number does not exist on PO: Trigger `ERR_LINE_NOT_FOUND`. |
| 8 | `ITEM_QUANTITY` | `RSEG` (Item) | DECIMAL(13,3) | Mandatory | `line_item_details[].qty` | Parse quantity; evaluate against Goods Receipt (`EKBE`). | If Invoice Qty > Goods Receipt Qty: Trigger `ERR_QTY_OVERAGE`. |
| 9 | `ITEM_UNIT_PRICE` | `RSEG` (Item) | DECIMAL(11,2) | Mandatory | `line_item_details[].unit_rate` | Direct map; calculate total line price (`qty` * `unit_rate`). | If Unit Price variance > 2% vs PO line price: Trigger `ERR_PRICE_MISMATCH`. |
| 10 | `TAX_AMOUNT` | `RSEG` (Item) | DECIMAL(13,2) | Optional | `tax_summary.calculated_tax` | Verify tax line rate calculation against state/country rules. | If tax sum != header total breakdown: Trigger `ERR_TAX_CALC_DISCREPANCY`. |
| 11 | `PAYMENT_TERMS` | `LFB1` (Vendor Company) | VARCHAR(4) | Mandatory | `agreed_payment_terms` | If null, default to ERP Vendor Master default terms (e.g., `NT30`). | If mismatched with PO terms: Flag for AP Specialist review. |
| 12 | `VENDOR_BANK_ACC` | `LFBK` (Bank Data) | VARCHAR(18) | Mandatory | `remit_to_account_num` | Decrypt incoming payload token; cross-check against ERP bank record. | If bank account differs from registered Vendor Master: Trigger `ERR_BANK_MISMATCH`. |
| 13 | `RECEIPT_DOC_NUM` | `EKBE` (History) | VARCHAR(10) | Conditional | `goods_receipt_id` | Match against GR document linked to specified `PO_NUMBER`. | If GR document missing for 3-way match PO: Trigger `ERR_GR_PENDING`. |
| 14 | `PAYMENT_BLOCK_FLAG`| `RBKP` (Header) | CHAR(1) | Automatic | N/A (Internal Logic) | Set to `'X'` (Blocked) if any validation error triggers; else set to `' '` (Clear). | Internal ERP control status indicator. |
| 15 | `EXCEPTION_TICKET_ID`| `RBKP` (Header) | VARCHAR(20) | Conditional | N/A (ServiceNow Integration)| Populated with ServiceNow ticket ID (e.g., `INC0982341`) if `PAYMENT_BLOCK_FLAG` = `'X'`. | Synchronizes ERP resolution state with ServiceNow workflow. |


### 3. Integration Payload Examples

#### Inbound Portal JSON Payload (Source Data)
```json
{
  "supplier_tax_identifier": "12-3456789",
  "invoice_header_number": "INV-88492-REV",
  "document_issue_date": "2026-11-01",
  "total_amount_due": 10500.00,
  "billing_currency": "USD",
  "purchase_order_reference": "4500012345",
  "line_item_details": [
    {
      "po_line": 1,
      "qty": 100,
      "unit_rate": 105.00
    }
  ]
}


{
  "Header": {
    "VENDOR_TAX_ID": "123456789",
    "INVOICE_NUM": "INV88492",
    "INVOICE_DATE": "20261101",
    "GROSS_AMOUNT": 10500.00,
    "CURRENCY_CODE": "USD",
    "PO_NUMBER": "004500012345",
    "PAYMENT_BLOCK_FLAG": " "
  },
  "Items": [
    {
      "PO_LINE_ITEM_NUM": 1,
      "ITEM_QUANTITY": 100.000,
      "ITEM_UNIT_PRICE": 105.00
    }
  ]
}
