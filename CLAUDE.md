# Trend Timbers - Student Quote System

## Project Overview

Automated quoting system for **Trend Timbers**, a timber supply company in Australia. The system processes student timber cutting lists from schools, generates quotes, handles approvals, and manages orders through to invoicing.

**Owner:** Matt (Technical Manager)  
**Business:** Trend Timbers - timber supplier  
**Primary Users:** Matt's boss (non-technical), school students submitting quotes

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CUSTOMER-FACING                                  │
├─────────────────────────────────────────────────────────────────────┤
│  trend-timbers-quote-builder-1-3.html                               │
│  - React-based wizard form                                          │
│  - Excel/CSV upload OR manual entry                                 │
│  - Live timber pricing from Google Sheets                           │
│  - Width nesting, length splitting calculations                     │
│  - 15% waste allowance auto-calculated                              │
└──────────────────────┬──────────────────────────────────────────────┘
                       │ POST to webhook
                       ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    N8N WORKFLOWS (8 total)                          │
├─────────────────────────────────────────────────────────────────────┤
│  01 - Excel Upload Handler     (processes submissions)              │
│  02 - Approval Handler         (boss approves/edits quotes)         │
│  03 - Get All Quotes           (dashboard data feed)                │
│  04 - Generate Picking Slip    (PDF generation)                     │
│  05 - Send Confirmation Email  (order confirmed to customer)        │
│  06 - Update Quote Status      (status changes from dashboard)      │
│  07 - Generate Invoice         (PDF invoice creation/email)         │
│  08 - Work Order               (printable work order for warehouse) │
└──────────────────────┬──────────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    DATA STORAGE                                     │
├─────────────────────────────────────────────────────────────────────┤
│  Google Sheets: "Quote Approvals"                                   │
│  Document ID: 1so-VLU9J8yCwIVoYcu1cV9hoQlAnCgHqRJmXBwkq1bA         │
│  Current Sheet: Sheet1 (gid=0)                                      │
└──────────────────────┬──────────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    ADMIN DASHBOARD                                  │
├─────────────────────────────────────────────────────────────────────┤
│  trend-timbers-dashboard-v8-FINAL.html                              │
│  - Quote management interface                                       │
│  - Status tracking (PENDING → QUOTE SENT → ORDER → COMPLETE)        │
│  - Action buttons per quote                                         │
│  - Invoice generation                                               │
│  - Email previews                                                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## N8N Workflows

All workflows hosted at: `https://ttimbers.app.n8n.cloud/webhook/`

### 01 - Excel Upload Handler
**Webhook:** `POST /1ab34010-47ac-4006-948b-ced4f7e5e774`  
**Purpose:** Receives student quote submissions from the quote builder form
- Generates unique QuoteID (format: TT-XXXX)
- Stores quote data in Google Sheets
- Creates approval link for boss
- Sends notification email to trendtimbers@gmail.com
- Generates picking slip HTML

### 02 - Approval Handler
**Webhook:** `GET/POST /approve-quote`  
**Purpose:** Allows boss to review and approve quotes
- GET request: Shows approval form with quote details
- POST request: Processes approval, updates pricing, sends quote to customer
- Supports delivery cost addition and custom notes
- Editable component pricing before sending

### 03 - Get All Quotes
**Webhook:** `GET /get-quotes`  
**Auth:** Header Auth (`X-API-Key: EbonyMaple`)  
**Purpose:** Returns all quotes for dashboard display
- Returns: QuoteID, student info, components, totals, status, EmailHTML, PickingSlipHTML

### 04 - Generate Fresh Picking Slip PDF
**Webhook:** `POST /generate-picking-slip`  
**Purpose:** Generates PDF picking slip for warehouse
- Uses PDFShift API for PDF generation
- Landscape A4 format
- Contains cutting list with component IDs (A, B, C...)

### 05 - Send Confirmation Email
**Webhook:** `POST /send-confirmation`  
**Auth:** Header Auth  
**Purpose:** Sends order confirmation to customer when status changes to ORDER

### 06 - Update Quote Status
**Webhook:** `POST /update-status`  
**Purpose:** Updates quote status and notes in Google Sheets
- Payload: `{ quoteId, status, notes }`

### 07 - Generate Invoice
**Webhook:** `POST /generate-invoice`  
**Auth:** Header Auth  
**Purpose:** Generates and optionally emails PDF invoice
- Supports download OR email to customer
- Invoice numbering system

### 08 - Work Order
**Webhook:** `GET /work-order?id=TT-XXXX`  
**Purpose:** Generates printable work order for warehouse staff
- Shows customer details, cutting list, delivery method
- One-page format optimised for printing

---

## Quote Statuses (Current)

| Status | Description |
|--------|-------------|
| `PENDING` | New quote, awaiting boss review |
| `QUOTE SENT` | Quote approved and emailed to customer |
| `FOLLOW UP` | Customer hasn't responded, needs follow-up |
| `ORDER` | Customer confirmed, order in progress |
| `COMPLETE` | Order fulfilled |

---

## Dashboard Actions (Current)

| Action | Description | Availability |
|--------|-------------|--------------|
| View Document | Opens original uploaded Excel file | If OriginalFile exists |
| View/Edit Quote | Opens approval form to edit pricing | If approvalLink exists |
| View Sent Email | Shows the email that was sent to customer | If EmailHTML exists |
| View Picking Slip | Shows cutting list for warehouse | If PickingSlipHTML exists |
| Send Confirmation | Sends order confirmation email | Always |
| Generate Invoice | Opens invoice modal | Always |
| Work Order | Opens printable work order | Only for ORDER/COMPLETE status |

---

## Google Sheets Structure

**Document:** Quote Approvals  
**Sheet:** Sheet1

| Column | Description |
|--------|-------------|
| QuoteID | Unique identifier (TT-XXXX) |
| Timestamp | Submission date/time |
| StudentName | Customer name |
| StudentEmail | Customer email |
| StudentMobile | Customer phone |
| School | School name |
| Teacher | Teacher name |
| DeliveryOption | "pickup" or "delivery" |
| ComponentsJSON | JSON array of timber components |
| TotalsJSON | JSON object with subtotal, gst, total |
| EmailHTML | Full HTML of sent quote email |
| PickingSlipHTML | Full HTML of picking slip |
| Status | Current quote status |
| DeliveryCost | Added delivery cost (if any) |
| Notes | Internal notes |
| OriginalFile | Google Drive link to uploaded file |
| SentDate | Date quote was sent to customer |
| approvalLink | URL to approval form |

---

## Brand Guidelines

### Colors
- **Primary Green:** `#055902`
- **Light Green:** `#067a04`
- **Dark Green:** `#044701`
- **Gold/Beige:** `#c5aa7f`
- **Light Gold:** `#d4c4a8`
- **Brown:** `#412e28`
- **Cream Background:** `#f8f9fa`

### Typography
- Font: System fonts (-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto)
- Clean, professional appearance
- No emojis in customer-facing emails (emojis OK in dashboard)

### Logo
- Embedded as base64 in emails and forms
- White version on coloured backgrounds

---

## API Authentication

Dashboard uses header-based auth for all API calls:
```javascript
headers: { 'X-API-Key': 'EbonyMaple' }
```

Matching n8n credential: "Header Auth account 3"

---

## Company Details

```javascript
const COMPANY = {
    name: 'Trend Timbers',
    phone: '02 4577 5277',
    email: 'info@trendtimbers.com.au',
    abn: '69 124 544 412',
    bank: {
        name: 'Westpac',
        accountName: 'RB AJ Clark PTY LTD',
        bsb: '032274',
        accountNumber: '249387'
    }
};
```

---

## Key Files

| File | Purpose |
|------|---------|
| `trend-timbers-quote-builder-1-3.html` | Customer-facing quote form |
| `trend-timbers-dashboard-v8-FINAL.html` | Admin dashboard |
| `01-08 workflow JSON files` | n8n workflow exports |
| `Quote_Approvals` Google Sheet | Central data storage |

---

## Known Issues / Considerations

1. **OAuth Token Expiration:** Google Sheets OAuth tokens can expire causing silent failures. Monitor for "401" errors.
2. **Boss is Non-Technical:** All interfaces need to be "idiot-proof" with clear labels and minimal options.
3. **Large Data in Sheets:** EmailHTML and PickingSlipHTML columns contain full HTML - can make sheet slow.

---

## TODO - See TODO.md

Current priorities:
1. Rename action buttons for clarity
2. Implement status-specific actions
3. Automatic status transitions
4. Multi-sheet structure (Current Orders / Completed / Deleted)
5. Dashboard view filtering
6. Long-term: Stripe/PayPal payment integration

---

## Development Notes

- All workflows use n8n Cloud instance
- PDFShift API used for PDF generation
- Forms hosted as static HTML (can be deployed anywhere)
- Dashboard fetches data on page load via authenticated webhook
