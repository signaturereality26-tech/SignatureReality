# 12 - Automation System

## Scheduled Tasks & Triggers

### Automation Overview

```
Automation Engine
├─ Time-Based Triggers
│  ├─ Hourly (Matching Engine)
│  ├─ 3x Daily (Follow-up Reminders)
│  ├─ Daily 2 AM (Backup)
│  ├─ Daily 6 AM (Cleanup)
│  └─ Weekly (Summary Report)
├─ Event-Based Triggers
│  ├─ Lead Creation
│  ├─ Status Change
│  ├─ Deal Closure
│  └─ Commission Calculation
└─ Manual Triggers
   ├─ Admin Panel
   ├─ One-Click Actions
   └─ Batch Operations
```

## Time-Based Automation

### 1. Hourly Matching Engine

**Trigger:** Every hour (00:00 UTC)
**Duration:** 5-10 minutes
**Purpose:** Run automatic lead-to-property matching

**Process:**
```javascript
function hourlyMatchingEngine() {
  // 1. Get all qualified leads without matches
  const leads = getUnmatchedLeads();
  
  // 2. For each lead, run matching algorithm
  leads.forEach(lead => {
    const matches = matchLeadToProperties(lead);
    
    // 3. If matches found and lead not contacted
    if (matches.length > 0 && !lead.matchesSent) {
      sendShortlistToLead(lead, matches);
      logActivity('HOURLY_MATCHING', lead.LeadID);
    }
  });
  
  // 4. Log execution
  logAutomationRun('hourlyMatchingEngine', leads.length);
}
```

**Configuration:**
```
Hourly Matching Enabled: Yes
Max Matches per Lead: 10
Minimum Match Score: 50%
Send Notification: Yes
Log Results: Yes
```

### 2. Daily Backup (2 AM IST)

**Trigger:** Every day at 2:00 AM IST
**Duration:** 5-15 minutes
**Purpose:** Backup all system data

**Process:**
```javascript
function dailyBackup() {
  const backupId = generateBackupId('daily');
  
  try {
    // 1. Export all sheets
    const sheets = SHEET_NAMES;
    sheets.forEach(sheet => {
      const data = getSheetData(sheet);
      storeBackup(backupId, sheet, data);
    });
    
    // 2. Encrypt backup
    encryptBackup(backupId);
    
    // 3. Store to Google Drive
    uploadBackupToDrive(backupId);
    
    // 4. Mark backup as complete
    logBackupCompletion(backupId, 'SUCCESS');
    
    // 5. Notify admin
    sendBackupNotification('Backup completed: ' + backupId);
    
  } catch (error) {
    logBackupCompletion(backupId, 'FAILED', error);
    sendErrorNotification('Backup failed: ' + error);
  }
}
```

**Configuration:**
```
Backup Schedule: Daily 2 AM IST
Retention Days: 30
Encryption: Yes
Notification: Yes
Google Drive Storage: Yes
```

### 3. Follow-Up Reminders (3x Daily)

**Trigger:** 3 times daily (8 AM, 12 PM, 4 PM IST)
**Duration:** 5-10 minutes
**Purpose:** Send follow-up reminders to agents

**Process:**
```javascript
function sendReminders() {
  // 1. Get leads needing follow-up
  const overdueLeads = getOverdueFollowUps();
  const upcomingVisits = getUpcomingSiteVisits(2); // Next 2 hours
  
  // 2. Send reminders to assigned agents
  overdueLeads.forEach(lead => {
    const agent = getUser(lead.Agent);
    sendEmail(agent.Email, 
      'Follow-up Reminder: ' + lead.Name,
      'Lead ' + lead.Name + ' needs follow-up action');
    logActivity('REMINDER_SENT', lead.LeadID);
  });
  
  // 3. Send site visit reminders
  upcomingVisits.forEach(visit => {
    const agent = getUser(visit.Agent);
    sendWhatsAppMessage(agent.Mobile,
      'Site visit reminder for ' + visit.PropertyID + 
      ' in ' + visit.ScheduledTime);
  });
}
```

**Configuration:**
```
Reminder Times: 8 AM, 12 PM, 4 PM IST
Overdue Threshold: 2 days
Visit Alert Lead: 2 hours
Notification Channels: Email, WhatsApp
Agent Notification: Yes
```

### 4. Daily Cleanup (6 AM IST)

**Trigger:** Every day at 6:00 AM IST
**Duration:** 10-20 minutes
**Purpose:** Cleanup old data and maintain database

**Process:**
```javascript
function dailyCleanup() {
  // 1. Archive old leads (1 year)
  const oldLeads = getLeadsOlderThan(365);
  oldLeads.forEach(lead => {
    archiveLead(lead.LeadID);
  });
  
  // 2. Mark inactive leads
  const inactiveLeads = getLeadsWithNoActivityFor(30);
  inactiveLeads.forEach(lead => {
    updateLeadStatus(lead.LeadID, 'INACTIVE');
  });
  
  // 3. Cleanup temp files
  deleteOldTempFiles(30); // Delete files older than 30 days
  
  // 4. Compact audit logs
  compactAuditLogs(90); // Keep 90 days of audit logs
  
  // 5. Remove duplicate entries
  removeDuplicateActivities();
  
  // 6. Log cleanup
  logAutomationRun('dailyCleanup', {
    archivedLeads: oldLeads.length,
    inactiveLeads: inactiveLeads.length
  });
}
```

**Configuration:**
```
Archive Threshold: 1 year
Inactivity Threshold: 30 days
Audit Log Retention: 90 days
Temp File Retention: 30 days
Cleanup Schedule: Daily 6 AM IST
```

### 5. Weekly Summary Report

**Trigger:** Every Monday 9 AM IST
**Duration:** 5-10 minutes
**Purpose:** Generate and send weekly summary

**Process:**
```javascript
function weeklyReport() {
  const reportPeriod = getLastWeek();
  
  // 1. Generate weekly metrics
  const metrics = {
    leadsAdded: countLeadsInPeriod(reportPeriod),
    leadsClosed: countClosedLeadsInPeriod(reportPeriod),
    revenueGenerated: getSumRevenueInPeriod(reportPeriod),
    commissionsCalculated: countCommissionsInPeriod(reportPeriod),
    activeAgents: countActiveAgentsInPeriod(reportPeriod)
  };
  
  // 2. Generate report
  const report = generateWeeklyReport(metrics);
  
  // 3. Export to PDF
  const pdfUrl = exportReportToPDF(report);
  
  // 4. Send to all managers
  const managers = getUsersByRole('Manager');
  managers.forEach(manager => {
    sendEmailWithAttachment(
      manager.Email,
      'Weekly Summary Report',
      'Please find attached the weekly summary report.',
      pdfUrl
    );
  });
  
  // 5. Log report generation
  logAutomationRun('weeklyReport', reportPeriod);
}
```

**Configuration:**
```
Schedule: Every Monday 9 AM IST
Recipients: All Managers
Format: PDF
Metrics: Leads, Revenue, Commissions
Archive: Yes
```

## Event-Based Automation

### 1. Lead Creation Automation

**Trigger:** When a new lead is created
**Purpose:** Auto-execute post-creation tasks

**Process:**
```javascript
function onLeadCreated(leadId) {
  // 1. Check for duplicates
  const isDuplicate = checkDuplicate(leadId);
  if (isDuplicate) {
    markAsDuplicate(leadId);
    return;
  }
  
  // 2. Auto-assign if settings enabled
  if (CONFIG.AUTO_ASSIGN_LEADS) {
    const agent = getAvailableAgent();
    assignLead(leadId, agent);
  }
  
  // 3. Create timeline entry
  addTimelineEntry(leadId, 'LEAD_CREATED');
  
  // 4. Send notification to manager
  const manager = getManagerForAgent(getAssignedAgent(leadId));
  sendNotification(manager, 'New lead created: ' + leadId);
}
```

### 2. Status Change Automation

**Trigger:** When lead status changes
**Purpose:** Execute status-specific actions

**Process:**
```javascript
function onStatusChanged(leadId, oldStatus, newStatus) {
  // 1. Update stage if needed
  if (shouldUpdateStage(newStatus)) {
    updateLeadStage(leadId, getStageForStatus(newStatus));
  }
  
  // 2. Trigger reminders
  if (newStatus === 'NEGOTIATING') {
    scheduleNegotiationReminder(leadId, 7); // Remind after 7 days
  }
  
  // 3. Log status change
  logActivity('STATUS_CHANGED', leadId, oldStatus, newStatus);
  
  // 4. Send notifications
  sendStatusChangeNotification(leadId, oldStatus, newStatus);
}
```

### 3. Deal Closure Automation

**Trigger:** When a deal is closed
**Purpose:** Post-closure tasks

**Process:**
```javascript
function onDealClosed(leadId) {
  // 1. Calculate final commission
  calculateCommission(leadId);
  
  // 2. Generate success report
  const report = generateDealClosureReport(leadId);
  
  // 3. Archive lead
  archiveLead(leadId);
  
  // 4. Update property status to SOLD
  updatePropertyStatus(lead.PropertyID, 'SOLD');
  
  // 5. Send congratulations email
  const agent = getAssignedAgent(leadId);
  sendEmail(agent.Email, 
    'Deal Closed Successfully!',
    report);
  
  // 6. Log closure
  logDealClosure(leadId, report);
}
```

## Manual Automation Triggers

### Admin Panel Triggers

**Available Actions:**
```
1. Run Hourly Matching Now
2. Run Backup Now
3. Send Reminders Now
4. Run Cleanup Now
5. Generate Report Now
6. Archive Old Leads
7. Reactivate Inactive Leads
8. Resync Data
9. Send Test Email
10. Validate Data Integrity
```

## Automation Monitoring

### Automation Log
```
Automation Run Tracking
├─ Execution Time
├─ Records Processed
├─ Success/Failure Status
├─ Error Messages
├─ Next Scheduled Run
└─ Admin Notification
```

### Failed Automation Handling

```javascript
function handleAutomationFailure(automationName, error) {
  // 1. Log error
  logError('AUTOMATION_FAILED', automationName, error);
  
  // 2. Retry logic
  const retryCount = getRetryCount(automationName);
  if (retryCount < 3) {
    scheduleRetry(automationName, 5); // Retry after 5 min
  }
  
  // 3. Notify admin
  sendAdminNotification(
    'ALERT: Automation failed - ' + automationName + 
    ': ' + error.message);
  
  // 4. Continue other automations
  // Don't stop entire system
}
```

---

Next: See [13_NOTIFICATION_SYSTEM.md](13_NOTIFICATION_SYSTEM.md) for notifications.
