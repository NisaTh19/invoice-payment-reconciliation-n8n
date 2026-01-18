# Automated Invoice–Payment Reconciliation (n8n)

This project demonstrates a demo of an automated finance operations workflow built with **n8n**.  
It reconciles invoices against payments using conservative, audit-friendly rules and isolates only risky cases for human review.

The workflow reflects how automation and AI can be safely applied in **finance and accounting contexts**, preserving explainability and internal controls (four-eyes principle).

## Business Problem

Invoice–payment reconciliation is a recurring task in finance teams that is often handled manually.
Accountants must cross-check invoices against bank payments, investigate discrepancies, and prepare exception reports.

This process is:
- time-consuming,
- error-prone,
- difficult to audit,
- and scales poorly as transaction volume grows.

In practice, most invoices match cleanly and require no human intervention.
The real cost comes from identifying and investigating the small subset of **exceptions**:
- unpaid or overdue invoices,
- payments without a corresponding invoice,
- mismatches caused by timing, rounding, or missing references.

This project aims to automate the reconciliation process while ensuring that only genuinely risky cases are escalated for human review.

## Workflow Overview

The workflow is implemented in **n8n** and follows a clear, modular pipeline:

1. Load invoice and payment data (CSV files).
2. Normalize amounts, currencies, and dates.
3. Reconcile invoices against payments using deterministic rules.
4. Classify and isolate exceptions.
5. Generate a concise summary for the accounting team.

Only exceptions require human attention; all clean matches are handled automatically.

![Workflow overview](diagrams/workflow_diagram.png)

## Reconciliation Logic (Deterministic and Audit-Friendly)

The core matching logic is intentionally conservative to remain explainable and audit-friendly.

### Matching rules
An invoice is matched to a payment only if **all** conditions are satisfied:

- **Same client** (`client_id`)
- **Same currency**
- **Amount tolerance:** `|payment_amount - invoice_total| ≤ 0.50`
  - (to allow for small rounding differences or bank fees)
- **Time window:** payment date is within **±120 days** of the invoice date
- **No double counting:** each payment can be used at most once

### Exception classification
If an invoice cannot be matched:

- **OVERDUE_INVOICE**: the due date has passed and the invoice remains unpaid  
- **UNPAID_NOT_DUE**: no matching payment yet, but due date has not passed

After all invoices are processed, any remaining unmatched payments are classified as:

- **ORPHAN_PAYMENT**: payment received without a corresponding invoice match
  - (e.g., missing reference, wrong client ID, or timing mismatch)


