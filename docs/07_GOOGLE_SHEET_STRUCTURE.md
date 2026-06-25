# 07 - Google Sheets Structure Setup Guide

## Complete Setup Instructions

### Step 1: Create Base Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Click **+ New** → **Google Sheet**
3. Name it: `SignatureReality-CRM-V10`
4. Share with your team

### Step 2: Create All Required Sheets

#### Core Data Sheets (Required)
1. **Leads** - Main lead database
2. **Users** - User accounts
3. **Inventory** - Property listings
4. **Requirements** - Lead requirements
5. **Activities** - Activity log

#### Transaction Sheets (Required)
6. **SiteVisits** - Site visit records
7. **Negotiations** - Negotiation records
8. **Tokens** - Token receipts
9. **Agreements** - Agreements
10. **Commissions** - Commission records

#### Reference Sheets (Required)
11. **Config** - System configuration
12. **Permissions** - RBAC matrix
13. **AuditLog** - Audit trail
14. **Reports** - Generated reports

#### Support Sheets (Optional)
15. **Templates** - Document templates
16. **Settings** - User preferences
17. **Notifications** - Notification log

### Step 3: Set Up Each Sheet

#### Sheet 1: Leads
```
Columns:
A: LeadID | B: Date | C: Name | D: Mobile | E: Email
F: Source | G: Status | H: Stage | I: Score | J: ScoreCategory
K: Agent | L: Verification | M: VerifiedDate | N: Budget | O: Location
P: PropertyType | Q: Urgency | R: Timeline | S: Notes
T: CreatedBy | U: CreatedDate | V: LastModifiedBy | W: LastModifiedDate

Data Validation:
- Source: WhatsApp, Facebook, Instagram, Housing, MagicBricks, 99acres, Manual
- Status: New, Verified, Assigned, Qualified, Shortlisted, Site Visited, Negotiating, Closed, Invalid
- Stage: [matching status list]
- ScoreCategory: Hot, Warm, Cold
```

#### Sheet 2: Users
```
Columns:
A: UserID | B: Name | C: Email | D: Mobile
E: Role | F: Department | G: Status | H: CreatedDate
I: LastLoginDate | J: Specialization | K: Commission%

Data Validation:
- Role: Agent, Builder, Broker, Manager, Admin
- Status: Active, Inactive, Suspended
- Specialization: Residential, Commercial, Industrial, Land, All
```

#### Sheet 3: Inventory
```
Columns:
A: PropertyID | B: PropertyName | C: Type | D: SubType | E: Builder
F: Owner | G: Area | H: Carpet | I: BHK | J: Price
K: Location | L: Address | M: City | N: Facing | O: Floor
P: Parking | Q: Possession | R: Amenities | S: Status
T: Photos | U: Videos | V: Documents | W: CreatedDate | X: LastModifiedDate

Data Validation:
- Type: Residential, Commercial, Industrial, Land, Agriculture
- Status: Available, Sold, Hold, Exclusive
- Facing: North, South, East, West, NE, NW, SE, SW
```

#### Sheet 4: Requirements
```
Columns:
A: RequirementID | B: LeadID | C: PropertyType | D: BHK
E: MinBudget | F: MaxBudget | G: MinArea | H: MaxArea
I: Locations | J: Facing | K: Parking | L: Amenities
M: Possession | N: Loan | O: Notes | P: CreatedDate

Data Validation:
- PropertyType: Residential, Commercial, Industrial, Land
- Possession: Ready, Future, Any
- Loan: Yes, No, Maybe
```

#### Sheet 5: SiteVisits
```
Columns:
A: VisitID | B: LeadID | C: PropertyID | D: ScheduledDate | E: ScheduledTime
F: CheckInTime | G: CheckOutTime | H: CheckInLocation | I: CheckOutLocation
J: Agent | K: PropertyCondition (1-5) | L: LeadInterest (1-5)
M: Feedback | N: NextStep | O: Status | P: CreatedDate

Data Validation:
- Status: Scheduled, Completed, Cancelled
- PropertyCondition: 1=Poor, 2=Fair, 3=Good, 4=Very Good, 5=Excellent
```

#### Sheet 6: Negotiations
```
Columns:
A: NegotiationID | B: LeadID | C: PropertyID | D: InitialOffer
E: CounterOffer | F: FinalAmount | G: Rounds | H: Status
I: DealProbability (0-100) | J: Notes | K: CreatedDate

Data Validation:
- Status: InProgress, Agreed, Failed
```

#### Sheet 7: Tokens
```
Columns:
A: TokenID | B: LeadID | C: PropertyID | D: Amount
E: ReceiptDate | F: ReceiptNumber | G: ReceivedBy
H: PaymentMethod | I: Documents | J: Status

Data Validation:
- PaymentMethod: Cash, Cheque, Transfer, Other
- Status: Received, Cancelled
```

#### Sheet 8: Commissions
```
Columns:
A: CommissionID | B: LeadID | C: PropertyID | D: SalePrice | E: CommissionRate
F: GrossCommission | G: TDS | H: GST | I: NetCommission
J: Agent1 | K: Agent1Percent | L: Agent1Amount
M: Agent2 | N: Agent2Percent | O: Agent2Amount
P: PaymentStatus | Q: PaymentDate | R: CreatedDate

Data Validation:
- PaymentStatus: Pending, Partial, Paid, Failed
```

#### Sheet 9: Config
```
Columns:
A: Key | B: Value | C: Type | D: Description

Required Config:
- Commission_Rate = 2.5
- Lead_Score_Hot = 80
- Lead_Score_Warm = 50
- Max_Leads_Per_Page = 50
- Session_Timeout = 30
```

#### Sheet 10: Permissions
```
Columns:
A: Role | B: Resource | C: Action | D: Allowed

Key Permissions:
- Admin: All resources, All actions = Yes
- Manager: All resources, Read/Update = Yes
- Agent: Own leads, Update = Yes
- Builder: Own inventory, All = Yes
```

#### Sheet 11: AuditLog
```
Columns:
A: LogID | B: Date | C: Time | D: UserID | E: Action
F: Resource | G: ResourceID | H: OldValue | I: NewValue
J: Status | K: Details

Data Validation:
- Status: Success, Failure
```

### Step 4: Set Up Formatting

**Freeze Header Rows:**
- Select row 1
- View → Freeze → 1 row

**Add Filters:**
- Select header row
- Data → Create a filter

**Format Headers:**
- Bold, background color (light blue)
- Center alignment
- 14px font

### Step 5: Add Sample Data

1. Add 2-3 sample leads
2. Add 2-3 sample properties
3. Add sample users
4. Test all formulas

### Step 6: Set Up Permissions

**For Team Access:**
1. Click Share (top right)
2. Enter team member emails
3. Set as Editor
4. Uncheck "Notify people"

**For RBAC in System:**
- Update Permissions sheet with roles
- Update Users sheet with team members

### Step 7: Connect to Google Apps Script

1. Go to **Extensions** → **Apps Script**
2. Copy the GAS code into editor
3. Deploy as web app
4. Run `setupCRM()` to initialize

---

Next: See [08_BACKEND_ARCHITECTURE.md](08_BACKEND_ARCHITECTURE.md) for backend setup.
