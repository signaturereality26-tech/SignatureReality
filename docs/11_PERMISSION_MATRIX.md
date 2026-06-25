# 11 - Permission Matrix (RBAC)

## Role-Based Access Control

### User Roles

| Role | Purpose | Permissions |
|------|---------|-------------|
| **Admin** | System Administration | Full Access (All Resources, All Actions) |
| **Manager** | Team Management & Reporting | Team Data, Reports, Commission, Limited Config |
| **Broker** | Inventory & Commission Management | All Inventory, Commission, Own Area Data |
| **Agent** | Lead & Deal Management | Own Leads, Site Visits, Own Area Properties |
| **Builder** | Builder Inventory | Own Builder Inventory, Own Properties |

### Resource Access Matrix

#### Leads Resource
```
┌─────────┬────────┬────────┬────────┬────────┬────────────┐
│ Action  │ Admin  │Manager │Broker  │ Agent  │  Builder   │
├─────────┼────────┼────────┼────────┼────────┼────────────┤
│ Create  │   ✅   │   ✅   │   ✅   │   ✅   │     ❌     │
│ Read    │   ✅   │   ✅   │   ✅   │  Own   │     ❌     │
│ Update  │   ✅   │   ✅   │   ✅   │  Own   │     ❌     │
│ Delete  │   ✅   │   ✅   │   ❌   │   ❌   │     ❌     │
│ Assign  │   ✅   │   ✅   │   ✅   │   ❌   │     ❌     │
│ View All│   ✅   │   ✅   │   ✅   │   ❌   │     ❌     │
│ Export  │   ✅   │   ✅   │   ✅   │  Own   │     ❌     │
└─────────┴────────┴────────┴────────┴────────┴────────────┘
```

#### Inventory Resource
```
┌─────────┬────────┬────────┬────────┬────────┬────────────┐
│ Action  │ Admin  │Manager │Broker  │ Agent  │  Builder   │
├─────────┼────────┼────────┼────────┼────────┼────────────┤
│ Create  │   ✅   │   ❌   │   ✅   │   ❌   │     ✅     │
│ Read    │   ✅   │   ✅   │   ✅   │   ✅   │  Own Only  │
│ Update  │   ✅   │   ❌   │   ✅   │   ❌   │  Own Only  │
│ Delete  │   ✅   │   ❌   │   ✅   │   ❌   │     ❌     │
│ View All│   ✅   │   ✅   │   ✅   │   ✅   │     ❌     │
│ Upload  │   ✅   │   ❌   │   ✅   │   ❌   │     ✅     │
│ Export  │   ✅   │   ✅   │   ✅   │   ✅   │  Own Only  │
└─────────┴────────┴────────┴────────┴────────┴────────────┘
```

#### Reports Resource
```
┌──────────┬────────┬────────┬────────┬────────┬────────────┐
│ Action   │ Admin  │Manager │Broker  │ Agent  │  Builder   │
├──────────┼────────┼────────┼────────┼────────┼────────────┤
│ View     │   ✅   │   ✅   │   ✅   │Limited │     ❌     │
│ Custom   │   ✅   │   ✅   │   ❌   │   ❌   │     ❌     │
│ Schedule │   ✅   │   ✅   │   ❌   │   ❌   │     ❌     │
│ Export   │   ✅   │   ✅   │   ✅   │Limited │     ❌     │
│ Share    │   ✅   │   ✅   │   ❌   │   ❌   │     ❌     │
│ Archive  │   ✅   │   ✅   │   ❌   │   ❌   │     ❌     │
└──────────┴────────┴────────┴────────┴────────┴────────────┘
```

#### Commission Resource
```
┌──────────┬────────┬────────┬────────┬────────┬────────────┐
│ Action   │ Admin  │Manager │Broker  │ Agent  │  Builder   │
├──────────┼────────┼────────┼────────┼────────┼────────────┤
│ View     │   ✅   │   ✅   │   ✅   │ Own    │     ❌     │
│ Calculate│   ✅   │   ✅   │   ✅   │   ❌   │     ❌     │
│ Approve  │   ✅   │   ✅   │   ✅   │   ❌   │     ❌     │
│ Pay      │   ✅   │   ✅   │   ✅   │   ❌   │     ❌     │
│ Edit     │   ✅   │   ✅   │   ✅   │   ❌   │     ❌     │
│ Reverse  │   ✅   │   ✅   │   ✅   │   ❌   │     ❌     │
└──────────┴────────┴────────┴────────┴────────┴────────────┘
```

#### User Management Resource
```
┌──────────┬────────┬────────┬────────┬────────┬────────────┐
│ Action   │ Admin  │Manager │Broker  │ Agent  │  Builder   │
├──────────┼────────┼────────┼────────┼────────┼────────────┤
│ Create   │   ✅   │   ❌   │   ❌   │   ❌   │     ❌     │
│ Read     │   ✅   │   ✅   │   ❌   │   ❌   │     ❌     │
│ Update   │   ✅   │  Own   │   ❌   │  Own   │    Own     │
│ Delete   │   ✅   │   ❌   │   ❌   │   ❌   │     ❌     │
│ Assign   │   ✅   │   ❌   │   ❌   │   ❌   │     ❌     │
│ Role Mgmt│   ✅   │   ❌   │   ❌   │   ❌   │     ❌     │
└──────────┴────────┴────────┴────────┴────────┴────────────┘
```

#### Settings & Config Resource
```
┌──────────┬────────┬────────┬────────┬────────┬────────────┐
│ Action   │ Admin  │Manager │Broker  │ Agent  │  Builder   │
├──────────┼────────┼────────┼────────┼────────┼────────────┤
│ View     │   ✅   │  Own   │  Own   │  Own   │   Own      │
│ Update   │   ✅   │   ❌   │   ❌   │   ❌   │     ❌     │
│ Backup   │   ✅   │   ❌   │   ❌   │   ❌   │     ❌     │
│ Restore  │   ✅   │   ❌   │   ❌   │   ❌   │     ❌     │
│ Audit    │   ✅   │   ✅   │   ❌   │   ❌   │     ❌     │
└──────────┴────────┴────────┴────────┴────────┴────────────┘
```

### Site Visit Permissions
```
┌──────────┬────────┬────────┬────────┬────────┬────────────┐
│ Action   │ Admin  │Manager │Broker  │ Agent  │  Builder   │
├──────────┼────────┼────────┼────────┼────────┼────────────┤
│ Schedule │   ✅   │   ✅   │   ✅   │   ✅   │     ❌     │
│ Update   │   ✅   │   ✅   │   ✅   │ Own    │     ❌     │
│ View All │   ✅   │   ✅   │   ✅   │   ❌   │     ❌     │
│ Feedback │   ✅   │   ✅   │   ✅   │ Own    │     ❌     │
│ Report   │   ✅   │   ✅   │   ✅   │Limited │     ❌     │
└──────────┴────────┴────────┴────────┴────────┴────────────┘
```

### Negotiation Permissions
```
┌──────────┬────────┬────────┬────────┬────────┬────────────┐
│ Action   │ Admin  │Manager │Broker  │ Agent  │  Builder   │
├──────────┼────────┼────────┼────────┼────────┼────────────┤
│ Create   │   ✅   │   ✅   │   ✅   │   ✅   │     ❌     │
│ Update   │   ✅   │   ✅   │   ✅   │ Own    │     ❌     │
│ View All │   ✅   │   ✅   │   ✅   │   ❌   │     ❌     │
│ Record   │   ✅   │   ✅   │   ✅   │ Own    │     ❌     │
│ Approve  │   ✅   │   ✅   │   ✅   │   ❌   │     ❌     │
└──────────┴────────┴────────┴────────┴────────┴────────────┘
```

## Function-Level Permissions

### Critical Functions

**Admin Only Functions:**
```
- setupCRM()
- deleteUser(userId)
- restoreBackup(backupId)
- updateSystemConfig(key, value)
- createAuditLog()
- manualTriggerBackup()
```

**Admin + Manager Functions:**
```
- assignLead(leadId, agentId)
- createUser(userData)
- generateReport(reportType)
- approveCommission(commId)
- viewAuditLog()
```

**Admin + Manager + Broker Functions:**
```
- createProperty(propData)
- calculateCommission(dealData)
- viewAllLeads()
- scheduleReport()
```

**All Authenticated Users:**
```
- updateOwnProfile(userData)
- changePassword()
- viewOwnLeads()
- createActivity()
```

## Data-Level Access Control

### Lead Access Rules
```javascript
// Admin: Can see all leads
if (userRole === 'Admin') {
  return getAllLeads();
}

// Manager: Can see team leads
if (userRole === 'Manager') {
  return getLeadsForTeam(userId);
}

// Broker: Can see own area leads
if (userRole === 'Broker') {
  return getLeadsForArea(userArea);
}

// Agent: Can see only own leads
if (userRole === 'Agent') {
  return getLeadsAssignedTo(userId);
}

// Builder: Cannot see leads
if (userRole === 'Builder') {
  return [];
}
```

### Inventory Access Rules
```javascript
// Admin: Can see all properties
if (userRole === 'Admin') {
  return getAllProperties();
}

// Manager: Can see all properties
if (userRole === 'Manager') {
  return getAllProperties();
}

// Broker: Can see all properties
if (userRole === 'Broker') {
  return getAllProperties();
}

// Agent: Can see all properties (but not edit)
if (userRole === 'Agent') {
  return getAllProperties().filter(p => p.status !== 'Sold');
}

// Builder: Can see only own properties
if (userRole === 'Builder') {
  return getPropertiesOwnedBy(userId);
}
```

## Permission Enforcement

### Backend Permission Check
```javascript
function checkPermission(userId, resource, action) {
  const user = getUser(userId);
  const permissions = getPermissions(user.role);
  
  if (!permissions[resource] || !permissions[resource][action]) {
    throw new Error('Access Denied');
  }
  
  logActivity(userId, 'ACCESS_GRANTED', resource, action);
  return true;
}
```

### Frontend Permission Check
```javascript
function canUserAccess(resource, action) {
  const userRole = getUserRole();
  const permissionMatrix = PERMISSIONS[userRole];
  return permissionMatrix[resource]?.[action] || false;
}

// Hide UI elements based on permissions
if (!canUserAccess('Commission', 'Pay')) {
  hideElement('payCommissionBtn');
}
```

## Special Cases

### "Own" Data Access
- **Own Leads**: Leads assigned to the user
- **Own Area**: Properties in user's assigned area
- **Own Builder Inventory**: Properties added by the builder user
- **Own Profile**: User's own profile only

### Team Access (Manager)
- Can see all leads assigned to their team members
- Can see all site visits of their team
- Can approve commissions for their team

### Area-Based Access (Broker)
- Can manage properties in their assigned area
- Can see leads for their area
- Can override agent assignments in their area

---

Next: See [12_AUTOMATION.md](12_AUTOMATION.md) for automation setup.
