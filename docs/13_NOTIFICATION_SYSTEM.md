# 13 - Notification System

## Multi-Channel Notifications

### Notification Channels

```
Notification System
├─ Email Notifications (Gmail)
├─ WhatsApp Notifications (WhatsApp Business API)
├─ In-App Notifications (System)
├─ SMS (Optional)
└─ Web Push (Future)
```

## Email Notifications

### Email Types

#### 1. Lead Created Notification
```
Recipient: Assigned Agent
Subject: New Lead Assignment - [Lead Name]
Content:
- Lead Details
- Contact Information
- Lead Score
- Recommended Next Steps
```

#### 2. Site Visit Reminder
```
Recipient: Assigned Agent
Subject: Site Visit Reminder - [Property Name]
Content:
- Visit Date & Time
- Property Address
- Lead Details
- Check-in Instructions
```

#### 3. Follow-Up Reminder
```
Recipient: Assigned Agent
Subject: Follow-up Reminder - [Lead Name]
Content:
- Lead Name
- Last Contact Date
- Action Required
- Suggested Message Template
```

#### 4. Deal Closure Notification
```
Recipient: Assigned Agent, Manager
Subject: Deal Closed Successfully - [Lead Name]
Content:
- Deal Summary
- Final Amount
- Commission Earned
- Congratulations Message
```

#### 5. Commission Notification
```
Recipient: Agent
Subject: Commission Calculated - Deal [Deal ID]
Content:
- Deal Details
- Commission Breakdown
- Payment Status
- Expected Payment Date
```

#### 6. Weekly Report
```
Recipient: Manager
Subject: Weekly Summary Report
Content:
- Leads Added
- Deals Closed
- Revenue Generated
- Top Performers
- Pending Actions
Attachment: PDF Report
```

### Email Implementation

```javascript
function sendEmail(to, subject, body, attachmentUrl = null) {
  try {
    const options = {
      htmlBody: htmlTemplateEmail(body),
      name: 'SignatureReality CRM'
    };
    
    if (attachmentUrl) {
      options.attachments = [Blob.createFromUrl(attachmentUrl)];
    }
    
    GmailApp.sendEmail(to, subject, body, options);
    
    // Log email sent
    logNotification('EMAIL', to, subject, 'SUCCESS');
    
  } catch (error) {
    logNotification('EMAIL', to, subject, 'FAILED', error.message);
    throw error;
  }
}

function sendLeadCreatedEmail(leadId) {
  const lead = getLead(leadId);
  const agent = getUser(lead.Agent);
  
  const subject = `New Lead: ${lead.Name}`;
  const body = `
    <h2>New Lead Assignment</h2>
    <p><strong>Lead Name:</strong> ${lead.Name}</p>
    <p><strong>Mobile:</strong> ${lead.Mobile}</p>
    <p><strong>Score:</strong> ${lead.Score} (${lead.ScoreCategory})</p>
    <p><strong>Budget:</strong> ₹${lead.Budget}</p>
    <p><strong>Preferred Location:</strong> ${lead.Location}</p>
    <p><strong>Next Steps:</strong></p>
    <ul>
      <li>Call lead to verify details</li>
      <li>Capture complete requirements</li>
      <li>Find matching properties</li>
    </ul>
  `;
  
  sendEmail(agent.Email, subject, body);
}
```

## WhatsApp Notifications

### WhatsApp Message Templates

#### 1. Lead Created
```
Hi {AGENT_NAME},

New lead assigned to you:

Name: {LEAD_NAME}
Mobile: {LEAD_MOBILE}
Score: {LEAD_SCORE}
Budget: ₹{BUDGET}
Location: {LOCATION}

Please verify and follow up.

Thank you,
SignatureReality CRM
```

#### 2. Site Visit Reminder
```
Hi {AGENT_NAME},

Reminder: Site visit in 2 hours

Property: {PROPERTY_NAME}
Time: {VISIT_TIME}
Location: {ADDRESS}

Lead: {LEAD_NAME}
Contact: {LEAD_MOBILE}

Best regards,
SignatureReality Team
```

#### 3. Follow-Up Alert
```
Hi {AGENT_NAME},

Lead follow-up due:

Name: {LEAD_NAME}
Mobile: {LEAD_MOBILE}
Last Contact: {LAST_DATE}

Please follow up now.

SignatureReality CRM
```

#### 4. Deal Closed Congratulations
```
Congratulations {AGENT_NAME}! 🎉

Deal closed successfully!

Lead: {LEAD_NAME}
Property: {PROPERTY_NAME}
Amount: ₹{DEAL_AMOUNT}
Commission: ₹{COMMISSION}

Great work!
SignatureReality Team
```

### WhatsApp Implementation

```javascript
function sendWhatsAppMessage(phoneNumber, message) {
  try {
    const payload = {
      messaging_product: 'whatsapp',
      recipient_type: 'individual',
      to: phoneNumber,
      type: 'text',
      text: {
        body: message
      }
    };
    
    const response = UrlFetchApp.fetch(
      'https://graph.instagram.com/v12.0/{PHONE_ID}/messages',
      {
        method: 'post',
        headers: {
          'Authorization': 'Bearer ' + WHATSAPP_TOKEN,
          'Content-Type': 'application/json'
        },
        payload: JSON.stringify(payload),
        muteHttpExceptions: true
      }
    );
    
    logNotification('WHATSAPP', phoneNumber, message, 'SUCCESS');
    return JSON.parse(response.getContentText());
    
  } catch (error) {
    logNotification('WHATSAPP', phoneNumber, message, 'FAILED', error.message);
    throw error;
  }
}

function sendSiteVisitReminder(visitId) {
  const visit = getSiteVisit(visitId);
  const agent = getUser(visit.Agent);
  const property = getProperty(visit.PropertyID);
  
  const message = `
    Site visit reminder:
    
    Property: ${property.PropertyName}
    Time: ${visit.ScheduledTime}
    Location: ${property.Address}
    
    Lead: ${visit.LeadID}
  `;
  
  sendWhatsAppMessage(agent.Mobile, message);
}
```

## In-App Notifications

### Notification Types

```javascript
const NOTIFICATION_TYPES = {
  SUCCESS: 'success',
  ERROR: 'error',
  WARNING: 'warning',
  INFO: 'info'
};

// Frontend notification
function showNotification(type, message, duration = 5000) {
  const notification = document.createElement('div');
  notification.className = `notification notification-${type}`;
  notification.innerHTML = message;
  notification.style.display = 'block';
  
  document.body.appendChild(notification);
  
  setTimeout(() => {
    notification.remove();
  }, duration);
}

// Usage
showNotification('success', 'Lead created successfully!');
showNotification('error', 'Failed to create lead. Please try again.');
```

## Notification Preferences

### User Notification Settings

```javascript
const NOTIFICATION_PREFERENCES = {
  EMAIL: {
    leadCreated: true,
    siteVisitReminder: true,
    followUpReminder: true,
    dealClosed: true,
    weeklyReport: true
  },
  WHATSAPP: {
    leadCreated: true,
    siteVisitReminder: true,
    followUpReminder: true,
    dealClosed: true
  },
  SMS: {
    leadCreated: false,
    siteVisitReminder: false
  }
};

function updateNotificationPreference(userId, channel, type, enabled) {
  const preferences = getUserNotificationPreferences(userId);
  preferences[channel][type] = enabled;
  saveUserNotificationPreferences(userId, preferences);
  
  logActivity('NOTIFICATION_PREFERENCE_UPDATED', userId);
}
```

## Notification Logging

### Notification Log Sheet

```
Columns:
A: NotificationID
B: Date/Time
C: Recipient
D: Type (Email/WhatsApp/SMS)
E: Subject/Message
F: Status (Sent/Failed/Delivered)
G: Error (if any)
H: Channel
```

### Log Notification Function

```javascript
function logNotification(channel, recipient, message, status, error = null) {
  const sheet = getSheet(SHEET_NAMES.NOTIFICATION_LOG);
  const newRow = [
    generateUniqueId('NOTIF'),
    new Date(),
    recipient,
    channel,
    message.substring(0, 100),
    status,
    error || '',
    getCurrentUser().Email
  ];
  
  sheet.appendRow(newRow);
}
```

## Critical Notifications

### Admin-Only Alerts

```javascript
function sendAdminAlert(alertType, message) {
  const admins = getUsersByRole('Admin');
  
  admins.forEach(admin => {
    sendEmail(admin.Email, 
      `[ALERT] ${alertType}`,
      message);
  });
  
  logNotification('EMAIL', 'admin@signaturereality.com', message, 'SENT');
}

// Usage examples:
sendAdminAlert('BACKUP_FAILED', 'Daily backup failed at 2 AM');
sendAdminAlert('HIGH_ERROR_RATE', 'System error rate is 5%+');
sendAdminAlert('COMMISSION_DISCREPANCY', 'Commission calculation mismatch detected');
```

---

Next: See [14_MATCHING_ENGINE.md](14_MATCHING_ENGINE.md) for matching algorithm details.
