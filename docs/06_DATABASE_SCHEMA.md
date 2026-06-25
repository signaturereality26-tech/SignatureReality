# 06 - Database Schema

## Google Sheets Structure

### Core Data Sheets

#### LEADS Sheet
Main lead records table

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| LeadID | String | Yes | Unique identifier (auto-generated) |
| Date | Date | Yes | Lead creation date |
| Name | String | Yes | Lead name |
| Mobile | String | Yes | Lead mobile number |
| Email | String | No | Lead email |
| Source | String | Yes | Lead source (WhatsApp/Facebook/etc) |
| Status | String | Yes | Lead status (New/Verified/Assigned/etc) |
| Stage | String | Yes | Current workflow stage |
| Score | Number | Yes | Lead score (0-100) |
| ScoreCategory | String | Yes | Hot/Warm/Cold |
| Agent | String | No | Assigned agent name |
| Verification | String | Yes | Verification status |
| VerifiedDate | Date | No | Verification date |
| Budget | Number | No | Lead budget |
| Location | String | No | Preferred location |
| PropertyType | String | No | Preferred property type |
| Urgency | String | No | High/Medium/Low |
| Timeline | String | No | Possession timeline |
| Notes | String | No | Additional notes |
| CreatedBy | String | Yes | Creator user ID |
| CreatedDate | Date | Yes | Creation timestamp |
| LastModifiedBy | String | No | Last modifier user ID |
| LastModifiedDate | Date | No | Last modification timestamp |

#### REQUIREMENTS Sheet
Lead property requirements

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| RequirementID | String | Yes | Unique identifier |
| LeadID | String | Yes | Link to lead |
| PropertyType | String | Yes | Residential/Commercial/etc |
| BHK | Number | No | Number of bedrooms |
| MinBudget | Number | No | Minimum budget |
| MaxBudget | Number | No | Maximum budget |
| MinArea | Number | No | Minimum area (sq ft) |
| MaxArea | Number | No | Maximum area (sq ft) |
| Locations | String | No | Comma-separated locations |
| Facing | String | No | North/South/East/West |
| Parking | String | No | Yes/No/Type |
| Amenities | String | No | Comma-separated amenities |
| Possession | String | No | Ready/Future/Any |
| Loan | String | No | Yes/No |
| Notes | String | No | Additional requirements |
| CreatedDate | Date | Yes | Creation date |

#### INVENTORY Sheet
Property inventory

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| PropertyID | String | Yes | Unique property ID |
| PropertyName | String | Yes | Property name/title |
| Type | String | Yes | Residential/Commercial/etc |
| SubType | String | No | Apartment/Villa/etc |
| Builder | String | No | Builder name (if applicable) |
| Owner | String | No | Owner name |
| Area | Number | No | Total area (sq ft) |
| Carpet | Number | No | Carpet area (sq ft) |
| BHK | Number | No | Number of bedrooms |
| Price | Number | Yes | Property price |
| Location | String | Yes | Property location |
| Address | String | No | Full address |
| City | String | Yes | City |
| Facing | String | No | Facing direction |
| Floor | String | No | Floor number |
| Parking | String | No | Parking details |
| Possession | String | No | Possession status |
| Amenities | String | No | Amenities list |
| Status | String | Yes | Available/Sold/Hold/Exclusive |
| Photos | String | No | Google Drive link to photos |
| Videos | String | No | Google Drive link to videos |
| Documents | String | No | Google Drive link to docs |
| CreatedDate | Date | Yes | Creation date |
| LastModifiedDate | Date | No | Last modified date |

#### USERS Sheet
User accounts and roles

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| UserID | String | Yes | Unique user ID |
| Name | String | Yes | User full name |
| Email | String | Yes | User email |
| Mobile | String | No | User mobile |
| Role | String | Yes | Agent/Builder/Broker/Manager/Admin |
| Department | String | No | Department name |
| Status | String | Yes | Active/Inactive/Suspended |
| CreatedDate | Date | Yes | Account creation date |
| LastLoginDate | Date | No | Last login date |
| Specialization | String | No | Residential/Commercial/etc |
| Commission% | Number | No | Commission percentage |

### Transaction Sheets

#### SITE_VISITS Sheet
Site visit records

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| VisitID | String | Yes | Unique visit ID |
| LeadID | String | Yes | Link to lead |
| PropertyID | String | Yes | Link to property |
| ScheduledDate | Date | Yes | Scheduled date |
| ScheduledTime | Time | No | Scheduled time |
| CheckInTime | Time | No | Actual check-in time |
| CheckOutTime | Time | No | Actual check-out time |
| CheckInLocation | String | No | GPS check-in location |
| CheckOutLocation | String | No | GPS check-out location |
| Agent | String | Yes | Agent name |
| PropertyCondition | Number | No | 1-5 rating |
| LeadInterest | Number | No | 1-5 rating |
| Feedback | String | No | Visit feedback |
| NextStep | String | No | Follow-up action |
| Status | String | Yes | Scheduled/Completed/Cancelled |
| CreatedDate | Date | Yes | Creation date |

#### NEGOTIATIONS Sheet
Negotiation records

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| NegotiationID | String | Yes | Unique negotiation ID |
| LeadID | String | Yes | Link to lead |
| PropertyID | String | Yes | Link to property |
| InitialOffer | Number | No | Lead's initial offer |
| CounterOffer | Number | No | Owner's counter |
| FinalAmount | Number | No | Final agreed amount |
| Rounds | Number | No | Number of negotiation rounds |
| Status | String | Yes | InProgress/Agreed/Failed |
| DealProbability | Number | No | 0-100 probability |
| Notes | String | No | Negotiation notes |
| CreatedDate | Date | Yes | Creation date |

#### TOKENS Sheet
Token receipt records

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| TokenID | String | Yes | Unique token ID |
| LeadID | String | Yes | Link to lead |
| PropertyID | String | Yes | Link to property |
| Amount | Number | Yes | Token amount |
| ReceiptDate | Date | Yes | Receipt date |
| ReceiptNumber | String | No | Receipt number |
| ReceivedBy | String | Yes | Agent name |
| PaymentMethod | String | Yes | Cash/Cheque/Transfer |
| Documents | String | No | Google Drive document link |
| Status | String | Yes | Received/Cancelled |

#### COMMISSIONS Sheet
Commission records

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| CommissionID | String | Yes | Unique commission ID |
| LeadID | String | Yes | Link to lead |
| PropertyID | String | Yes | Link to property |
| SalePrice | Number | Yes | Final sale price |
| CommissionRate | Number | Yes | Commission rate % |
| GrossCommission | Number | Yes | Gross commission |
| TDS | Number | No | TDS deduction |
| GST | Number | No | GST amount |
| NetCommission | Number | Yes | Net commission |
| Agent1 | String | No | Primary agent |
| Agent1Percent | Number | No | Agent 1 percentage |
| Agent1Amount | Number | No | Agent 1 amount |
| Agent2 | String | No | Secondary agent |
| Agent2Percent | Number | No | Agent 2 percentage |
| Agent2Amount | Number | No | Agent 2 amount |
| PaymentStatus | String | Yes | Pending/Paid/Partial |
| PaymentDate | Date | No | Payment date |
| CreatedDate | Date | Yes | Creation date |

### Reference Sheets

#### CONFIG Sheet
System configuration

| Column | Type | Description |
|--------|------|-------------|
| Key | String | Configuration key |
| Value | String | Configuration value |
| Type | String | String/Number/Boolean |
| Description | String | Purpose |

**Example Config:**
```
Commission_Rate | 2.5 | Number | Default commission rate
Lead_Score_Hot | 80 | Number | Score threshold for Hot leads
Lead_Score_Warm | 50 | Number | Score threshold for Warm leads
MaxLeadsPerPage | 50 | Number | Pagination limit
SessionTimeout | 30 | Number | Session timeout in minutes
```

#### PERMISSIONS Sheet
RBAC permission matrix

| Role | Resource | Action | Allowed |
|------|----------|--------|----------|
| Agent | Leads | Create | Yes |
| Agent | Leads | Read Own | Yes |
| Agent | Leads | Update Own | Yes |
| Agent | Reports | View | Limited |
| Manager | Leads | Create | Yes |
| Manager | Leads | Read All | Yes |
| Manager | Leads | Update | Yes |
| Manager | Reports | View | Full |
| Admin | * | * | Yes |

#### AUDIT_LOG Sheet
Activity audit trail

| Column | Type | Description |
|--------|------|-------------|
| LogID | String | Unique log ID |
| Date | Date | Event date |
| Time | Time | Event time |
| UserID | String | User who performed action |
| Action | String | Action performed |
| Resource | String | Resource affected |
| ResourceID | String | ID of affected resource |
| OldValue | String | Previous value |
| NewValue | String | New value |
| Status | String | Success/Failure |
| Details | String | Additional details |

---

Next: See [07_GOOGLE_SHEET_STRUCTURE.md](07_GOOGLE_SHEET_STRUCTURE.md) for setup guide.
