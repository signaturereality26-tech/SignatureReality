# 08 - Backend Architecture

## Google Apps Script Structure

### Module Overview

```
Backend Architecture
├── Auth.gs (Authentication & RBAC)
├── Config.gs (Configuration)
├── Leads.gs (Lead Management)
├── Requirement.gs (Requirements)
├── Inventory.gs (Inventory Management)
├── MatchingEngine.gs (Matching & Scoring)
├── SiteVisit.gs (Site Visit Management)
├── Negotiation.gs (Negotiation Tracking)
├── TokenDeal.gs (Token & Agreements)
├── Commission.gs (Commission Calculation)
├── Reports.gs (Reporting)
├── Dashboard.gs (Dashboard Data)
├── Notifications.gs (Email & Notifications)
├── Automation.gs (Scheduled Tasks)
├── Validation.gs (Data Validation)
├── Audit.gs (Activity Logging)
├── Utils.gs (Utility Functions)
└── Setup.gs (Initialization)
```

### Module Descriptions

#### 1. Auth.gs (Authentication & Authorization)
**Purpose:** User authentication and role-based access control

**Key Functions:**
```javascript
// Authentication
function doGet(e) - Main entry point for web app
function getCurrentUser() - Get current logged-in user
function login(email, password) - User login
function logout() - User logout
function getCurrentUserRole() - Get user's role

// Authorization
function checkPermission(resource, action) - Check access
function hasAccess(userId, resource, action) - Verify access
function getPermissions(role) - Get role permissions
function enforceRBAC(resource, action) - Enforce rules
```

**Dependencies:** Config.gs, Utils.gs
**Called By:** All modules
**Sheets Used:** Users, Permissions

#### 2. Config.gs (Configuration Management)
**Purpose:** System configuration and constants

**Key Functions:**
```javascript
function getConfig(key) - Get configuration value
function setConfig(key, value) - Set configuration
function getAllConfigs() - Get all configurations
function getSheetNames() - Get all sheet names
function getLeadSources() - Get lead sources list
function getPropertyTypes() - Get property types
```

**Constants:**
```javascript
const SHEET_NAMES = { LEADS: 'Leads', USERS: 'Users', ... }
const LEAD_STATUSES = ['New', 'Verified', 'Assigned', ...]
const LEAD_SOURCES = ['WhatsApp', 'Facebook', ...]
const MAX_LEADS_PER_PAGE = 50
const COMMISSION_RATE = 2.5
```

**Sheets Used:** Config

#### 3. Leads.gs (Lead Management)
**Purpose:** Lead CRUD operations and management

**Key Functions:**
```javascript
function createLead(leadData) - Create new lead
function getLead(leadId) - Get lead details
function getAllLeads(filters) - Get leads with filters
function updateLead(leadId, leadData) - Update lead
function deleteLead(leadId) - Delete lead
function searchLeads(query) - Search leads
function assignLead(leadId, agentId) - Assign lead
function scoreLead(leadId) - Calculate lead score
function duplicateDetection(leadData) - Check duplicates
```

**Dependencies:** Auth.gs, Config.gs, Audit.gs, Validation.gs
**Called By:** Frontend, Automation.gs
**Sheets Used:** Leads, Activities

#### 4. Requirement.gs (Lead Requirements)
**Purpose:** Manage lead property requirements

**Key Functions:**
```javascript
function createRequirement(leadId, reqData) - Create requirement
function getRequirements(leadId) - Get lead requirements
function updateRequirement(reqId, data) - Update requirement
function deleteRequirement(reqId) - Delete requirement
function matchRequirements(leadId) - Find matching properties
```

**Dependencies:** Auth.gs, Leads.gs
**Sheets Used:** Requirements

#### 5. Inventory.gs (Property Inventory)
**Purpose:** Property inventory management

**Key Functions:**
```javascript
function createProperty(propData) - Add property
function getProperty(propertyId) - Get property details
function getAllProperties(filters) - Get properties
function updateProperty(propertyId, data) - Update property
function deleteProperty(propertyId) - Delete property
function searchProperties(query) - Search properties
function updatePropertyStatus(propertyId, status) - Update status
function uploadPropertyPhotos(propertyId, folderLink) - Upload media
```

**Dependencies:** Auth.gs, Config.gs
**Sheets Used:** Inventory

#### 6. MatchingEngine.gs (Intelligent Matching)
**Purpose:** Match properties to leads using hard/soft filters and AI scoring

**Key Functions:**
```javascript
function matchLeadToProperties(leadId) - Find matches
function applyHardFilters(leadReq, properties) - Hard filtering
function applySoftFilters(leadReq, properties) - Soft filtering
function calculateMatchScore(lead, property) - Calculate score
function getTopMatches(leadId, limit) - Get top matches
function hourlyMatchingEngine() - Scheduled matching
```

**Dependencies:** Auth.gs, Leads.gs, Inventory.gs, Requirement.gs
**Called By:** Automation.gs (hourly trigger)
**Sheets Used:** Leads, Inventory, Requirements

#### 7. SiteVisit.gs (Site Visit Management)
**Purpose:** Schedule and track site visits with GPS

**Key Functions:**
```javascript
function scheduleSiteVisit(visitData) - Schedule visit
function getSiteVisit(visitId) - Get visit details
function updateSiteVisit(visitId, data) - Update visit
function recordCheckIn(visitId, gpsLocation) - GPS check-in
function recordCheckOut(visitId, gpsLocation) - GPS check-out
function submitFeedback(visitId, feedback) - Record feedback
function getSiteVisitsByLead(leadId) - Get lead visits
```

**Dependencies:** Auth.gs, Leads.gs, Notifications.gs
**Sheets Used:** SiteVisits

#### 8. Negotiation.gs (Negotiation Tracking)
**Purpose:** Track negotiation rounds and deal probability

**Key Functions:**
```javascript
function createNegotiation(negData) - Start negotiation
function recordOffer(negId, amount, side) - Record offer
function recordCounterOffer(negId, amount) - Record counter
function updateNegotiationStatus(negId, status) - Update status
function calculateDealProbability(negId) - Calculate probability
function getNegotiationHistory(negId) - Get history
```

**Dependencies:** Auth.gs, Leads.gs
**Sheets Used:** Negotiations

#### 9. TokenDeal.gs (Token & Agreements)
**Purpose:** Manage token receipts and agreements

**Key Functions:**
```javascript
function createTokenReceipt(tokenData) - Create token record
function generateTokenReceipt(tokenId) - Generate receipt
function createAgreement(agreementData) - Create agreement
function uploadAgreement(agreementId, docLink) - Upload doc
function finalizeAgreement(agreementId) - Mark finalized
function getTokensByLead(leadId) - Get lead tokens
```

**Dependencies:** Auth.gs, Commission.gs
**Sheets Used:** Tokens, Agreements

#### 10. Commission.gs (Commission Calculation)
**Purpose:** Automatic commission calculation with splits

**Key Functions:**
```javascript
function calculateCommission(dealData) - Calculate commission
function applyTDS(amount) - Apply TDS
function applyGST(amount) - Apply GST
function splitCommission(amount, agents) - Split commission
function createCommissionRecord(commData) - Create record
function processCommissionPayment(commId) - Mark as paid
function generateCommissionReport() - Generate report
```

**Formulas:**
```
Gross Commission = Sale Price × Commission Rate
TDS = Gross Commission × TDS%
GST = Gross Commission × GST%
Net Commission = Gross - TDS - GST
Agent Amount = Net × Agent%
```

**Sheets Used:** Commissions, Config

#### 11. Reports.gs (Reporting Engine)
**Purpose:** Generate various business reports

**Key Functions:**
```javascript
function generateLeadReport(filters) - Lead reports
function generateRevenueReport(period) - Revenue reports
function generateAgentReport(agentId) - Agent performance
function generateCommissionReport(period) - Commission reports
function generateInventoryReport() - Inventory status
function exportReport(reportType) - Export to PDF/CSV
```

**Report Types:**
- Lead Source Performance
- Revenue by Agent
- Commission Tracking
- Monthly Summary
- Conversion Funnel

**Sheets Used:** All data sheets

#### 12. Dashboard.gs (Dashboard Data)
**Purpose:** Provide aggregated data for dashboard

**Key Functions:**
```javascript
function getDashboardData(userId) - Get dashboard data
function getKPIs() - Get KPI values
function getLeadPipeline() - Get pipeline status
function getRevenueMetrics() - Get revenue data
function getAgentMetrics() - Get agent metrics
function getConversionMetrics() - Get conversion data
```

**KPIs Calculated:**
- Total Leads
- Lead Conversion Rate
- Average Deal Value
- Total Revenue
- Commission Paid
- Pending Deals

**Sheets Used:** All data sheets

#### 13. Notifications.gs (Notifications)
**Purpose:** Send email and WhatsApp notifications

**Key Functions:**
```javascript
function sendEmail(to, subject, body) - Send email
function sendLeadCreatedNotification(leadId) - Lead notification
function sendSiteVisitReminder(visitId) - Visit reminder
function sendNegotiationNotification(negId) - Deal notification
function sendCommissionNotification(commId) - Commission alert
function sendFollowUpReminder(leadId) - Follow-up reminder
```

**Integration:** Gmail, WhatsApp Business API

#### 14. Automation.gs (Automation & Tasks)
**Purpose:** Handle scheduled tasks and automation

**Key Functions:**
```javascript
function hourlyMatching() - Run hourly matching
function dailyBackup() - Daily data backup
function sendReminders() - Send due reminders
function cleanupOldRecords() - Cleanup aged records
function generateDailyReports() - Daily report generation
function auditInactiveLeads() - Mark inactive leads
```

**Triggers:**
- Time-based: Hourly, Daily at 2 AM, 3 times daily
- Event-based: Lead creation, Status change

#### 15. Validation.gs (Data Validation)
**Purpose:** Validate all input data

**Key Functions:**
```javascript
function validateLeadData(data) - Validate lead
function validatePropertyData(data) - Validate property
function validateEmail(email) - Validate email
function validatePhone(phone) - Validate phone
function validateBudget(budget) - Validate budget
function validateCommissionData(data) - Validate commission
```

#### 16. Audit.gs (Audit Logging)
**Purpose:** Log all user actions for audit trail

**Key Functions:**
```javascript
function logActivity(userId, action, resource, details) - Log action
function getActivityLog(filters) - Get audit log
function exportAuditLog() - Export audit trail
function trackDataChange(resource, oldVal, newVal) - Track changes
```

**Logged Events:**
- Lead creation/modification
- Lead deletion
- Status changes
- Commission calculations
- User actions
- Failed operations

#### 17. Utils.gs (Utility Functions)
**Purpose:** Common utility functions

**Key Functions:**
```javascript
function generateUniqueId(prefix) - Generate IDs
function getSheet(sheetName) - Get sheet reference
function getSheetData(sheetName, range) - Get range data
function appendToSheet(sheetName, data) - Append row
function updateSheetData(sheetName, range, data) - Update range
function getLastRow(sheetName) - Get last row
function formatCurrency(amount) - Format currency
function formatDate(date) - Format date
```

#### 18. Setup.gs (Initialization)
**Purpose:** Initialize CRM on first deployment

**Key Functions:**
```javascript
function setupCRM() - Complete setup
function initializeSheets() - Create sheets
function loadDefaultData() - Load defaults
function setupTriggers() - Setup automation
function setupPermissions() - Setup RBAC
function validateSetup() - Validate setup
```

---

Next: See [09_FRONTEND_ARCHITECTURE.md](09_FRONTEND_ARCHITECTURE.md) for frontend design.
