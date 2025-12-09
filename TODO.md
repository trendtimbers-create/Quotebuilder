# Trend Timbers Quoting System - TODO

## Action Name Changes
- [ ] Rename "Show Document" → "Show Student Upload"
- [ ] Rename "View/Edit Quote" → "Approve Quote"
- [ ] Rename "View Sent Email" → "View Quote"
- [ ] Rename "Send Confirmation" → "Confirm Order"

## Status-Specific Actions
Implement dynamic action buttons based on quote status:

| Status | Available Actions |
|--------|-------------------|
| Pending | Approve Quote, Show Student Upload |
| Quote Sent | View Quote, Confirm Order |
| Order | Show Student Upload, View Picking Slip, Generate Invoice |
| Complete | Show Student Upload, Generate Invoice |

- [ ] Update dashboard to show/hide actions based on status
- [ ] Create "View Picking Slip" action
- [ ] Create "Generate Invoice" action

## Automatic Status Transitions
- [ ] "Approve Quote" action → Status changes to "Quote Sent"
- [ ] "Confirm Order" action → Status changes to "Order"
- [ ] "Generate Invoice" action → Status changes to "Complete"

## Google Sheets Structure
Reorganise "Quote Approvals" spreadsheet into multiple sheets:
- [ ] Rename existing sheet to "Current Orders"
- [ ] Create "Completed Orders" sheet
- [ ] Create "Deleted Orders" sheet
- [ ] Update workflows to write to appropriate sheets based on status

## Dashboard Features
- [ ] Add "Delete Quote" action (moves to Deleted Orders sheet, hides from dashboard)
- [ ] Create default view: Current Orders only
- [ ] Create secondary view: Completed Orders
- [ ] Add view toggle/filter to dashboard

---

## Long-Term Goals
- [ ] Add online payment option to quote email
  - Stripe integration (account exists)
  - PayPal integration (account exists)
  - Decide which to implement first

---

## Completed
_Move items here as they're finished_

