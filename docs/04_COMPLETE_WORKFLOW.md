# 04 - Complete Workflow with All Stages

## Complete Lead-to-Deal Workflow

This document provides the exhaustive workflow from lead acquisition through deal closure.

### Stage Flow Diagram

```
② LEAD SOURCE
   ┃
   └─── Manual Entry
   └─── WhatsApp Integration
   └─── Facebook Lead Ads
   └─── Instagram DMs
   └─── Housing.com API
   └─── MagicBricks API
   └─── 99acres API
        │
        ↓
③ DUPLICATE CHECK
   ┃
   ├─ Phone Match
   ├─ Email Match
   ├─ Name + Phone Fuzzy Match
   └─ Phone + Location Fuzzy Match
        │
        ├── Match Found → Merge/Mark Duplicate
        └── No Match → Proceed
        │
        ↓
④ LEAD CREATION
   ┃
   ├─ Generate Lead ID
   ├─ Create Lead Record
   ├─ Set Initial Status: "New"
   └─ Create Timeline Entry
        │
        ↓
⑤ LEAD VERIFICATION
   ┃
   ├─ Verify Phone Number
   ├─ Confirm Lead Authenticity
   ├─ Check Budget Realism
   └─ Update Verification Status
        │
        ├── Verified → Proceed
        ├── Not Verified → Archive
        └── Call Later → Set Reminder
        │
        ↓
⑥ LEAD ASSIGNMENT
   ┃
   ├─ Auto Assignment (by specialization & workload)
   └─ Manual Assignment (by manager)
        │
        ↓
⑦ REQUIREMENT CAPTURE
   ┃
   ├─ Property Type
   ├─ Budget Range
   ├─ Location Preference
   ├─ Area/BHK
   ├─ Possession Timeline
   ├─ Lead Urgency
   └─ Additional Notes
        │
        ↓
⑧ LEAD SCORING
   ┃
   ├─ Budget Match (0-25 pts)
   ├─ Urgency (0-25 pts)
   ├─ Credibility (0-25 pts)
   ├─ Engagement (0-25 pts)
   └─ Total: 0-100 points
        │
        ├── 80-100: Hot
        ├── 50-79: Warm
        └── 0-49: Cold
        │
        ↓
⑨ VERIFICATION COMPLETE
   ┃
   └─ Set Status: "Verified"
        │
        ↓
⑩ MATCHING ENGINE
   ┃
   ├─ Hard Filters (Budget, Type, Location)
   ├─ Soft Filters (Preference-based)
   ├─ AI Scoring
   └─ Generate Top 5-10 Matches
        │
        ↓
⑪ SHORTLIST CREATION
   ┃
   ├─ Select 3-10 Properties
   ├─ Add Agent Notes
   ├─ Send to Lead
   └─ Track Send Status
        │
        ↓
⑫ LEAD RESPONSE
   ┃
   ├─ Lead Selects Properties
   ├─ Lead Expresses Interest
   └─ Lead Provides Feedback
        │
        ↓
⑬ SITE VISIT SCHEDULING
   ┃
   ├─ Confirm Date & Time
   ├─ Send Reminder
   ├─ Prepare Site Visit Package
   └─ Set GPS Tracking
        │
        ↓
⑭ SITE VISIT EXECUTION
   ┃
   ├─ Check-in with GPS
   ├─ Property Tour
   ├─ Lead Feedback
   ├─ Photos/Videos
   ├─ Check-out with GPS
   └─ Update Lead Status: "Site Visited"
        │
        ↓
⑮ CLIENT DISCUSSION
   ┃
   ├─ Discuss Property Details
   ├─ Address Concerns
   ├─ Discuss Price Range
   └─ Gauge Interest Level
        │
        ↓
⑯ PROPERTY COMPARISON
   ┃
   ├─ Compare Multiple Properties
   ├─ Highlight Differences
   ├─ Collect Lead Feedback
   └─ Identify Top Choice
        │
        ↓
⑰ NEGOTIATION INITIATION
   ┃
   ├─ Lead Makes Initial Offer
   ├─ Property Owner Responds
   ├─ Track All Offers
   └─ Set Status: "Negotiating"
        │
        ↓
⑱ COUNTER-OFFER ROUNDS
   ┃
   ├─ Owner Counter-Offer
   ├─ Lead Response
   ├─ Multiple Rounds
   ├─ Calculate Deal Probability
   └─ Log All Discussions
        │
        ↓
⑲ AGREEMENT REACHED
   ┃
   ├─ Both Parties Agree
   ├─ Final Amount
   ├─ Terms & Conditions
   └─ Set Status: "Agreed"
        │
        ↓
⑳ TOKEN RECEIPT
   ┃
   ├─ Generate Receipt
   ├─ Record Amount
   ├─ Record Date & Payer
   ├─ Store Documents
   └─ Create Token Record
        │
        ↓
⑴ AGREEMENT DOCUMENTATION
   ┃
   ├─ Buyer Signature
   ├─ Seller Signature
   ├─ Witness Signatures
   ├─ Store Agreement Copy
   └─ Set Status: "Token Received"
        │
        ↓
⑵ REGISTRY APPLICATION
   ┃
   ├─ Prepare Registry Documents
   ├─ Submit Application
   ├─ Track Application Status
   └─ Set Status: "Registry Processing"
        │
        ↓
⑶ REGISTRY COMPLETION
   ┃
   ├─ Registry Approval
   ├─ Registry Certificate Received
   ├─ Verify Certificate
   └─ Set Status: "Registry Complete"
        │
        ↓
⑷ COMMISSION CALCULATION
   ┃
   ├─ Calculate Gross Commission
   ├─ Deduct TDS (Tax)
   ├─ Calculate GST
   ├─ Calculate Net Commission
   ├─ Split Among Agents
   └─ Create Commission Record
        │
        ↓
⑸ PAYMENT PROCESSING
   ┃
   ├─ Generate Payment Voucher
   ├─ Process Payment
   ├─ Update Payment Status: "Paid"
   └─ Record Payment Date
        │
        ↓
⑹ DEAL CLOSED
   ┃
   ├─ Set Status: "Closed"
   ├─ Mark Completion Date
   ├─ Record Success Metrics
   ├─ Generate Success Report
   └─ Archive Lead
        │
        ↓
⑺ REPORTING & ANALYTICS
   ┃
   ├─ Generate Lead Source Report
   ├─ Generate Revenue Report
   ├─ Generate Agent Performance
   ├─ Generate Commission Report
   └─ Generate Monthly Summary
        │
        ↓
⑻ AUTOMATION & ARCHIVE
   ┃
   ├─ Archive Historical Data
   ├─ Maintain Audit Trail
   ├─ Backup System Data
   └─ Cleanup Old Records
```

## Detailed Stage Information

### Stage 1: Lead Source
**Duration**: Real-time
**Actors**: Lead Source Systems
**Outputs**: Raw Lead Data

### Stage 2-20: [See 02_BUSINESS_BLUEPRINT.md for detailed stage information]

### Stage 21-23: Post-Deal Activities

**Reporting**: Generate all analytics
**Automation**: Trigger post-deal workflows
**Archive**: Store historical data

---

Next: See [05_STATE_MACHINE.md](05_STATE_MACHINE.md) for state definitions.
