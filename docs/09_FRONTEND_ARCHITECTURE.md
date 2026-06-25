# 09 - Frontend Architecture

## Web Application Interface

### Frontend Technology Stack

- **HTML5**: Semantic markup
- **CSS3**: Responsive design with Flexbox/Grid
- **JavaScript ES6+**: Client-side logic
- **Google Apps Script HTML Service**: Integration

### Frontend Structure

```
frontend/
├── index.html (Main container)
├── css/
│   ├── style.css (Global styles)
│   ├── dashboard.css (Dashboard specific)
│   ├── leads.css (Lead page styles)
│   ├── inventory.css (Inventory styles)
│   ├── responsive.css (Mobile responsive)
│   └── components.css (Reusable components)
├── js/
│   ├── main.js (Application initialization)
│   ├── dashboard.js (Dashboard logic)
│   ├── leads.js (Lead management)
│   ├── inventory.js (Inventory logic)
│   ├── matching.js (Matching logic)
│   ├── reports.js (Report generation)
│   ├── api.js (Backend API calls)
│   └── utils.js (Utility functions)
├── pages/
│   ├── DashboardPage.html
│   ├── LeadPage.html
│   ├── InventoryPage.html
│   ├── MatchingPage.html
│   ├── ReportsPage.html
│   ├── AdminPanel.html
│   └── UserProfile.html
└── components/
    ├── header.html
    ├── sidebar.html
    ├── modals.html
    └── dialogs.html
```

### Main Pages

#### 1. Dashboard Page
**Purpose:** Overview and KPI tracking

**Components:**
- KPI Cards (Leads, Revenue, Conversion)
- Lead Pipeline Chart
- Revenue Trend Graph
- Agent Leaderboard
- Recent Activities
- Quick Actions

**Data Source:** Dashboard.gs

#### 2. Lead Management Page
**Purpose:** Lead CRUD operations

**Components:**
- Lead List View (Paginated)
- Lead Search & Filters
- Create Lead Form
- Lead Detail View
- Timeline View
- Bulk Actions

**API Calls:**
```javascript
google.script.run
  .withSuccessHandler(displayLeads)
  .getAllLeads(filters)

google.script.run
  .withSuccessHandler(showSuccess)
  .withFailureHandler(showError)
  .createLead(leadData)
```

#### 3. Inventory Page
**Purpose:** Property inventory management

**Components:**
- Property List View
- Property Search
- Filters (Location, Type, Price)
- Property Detail Modal
- Photo Gallery
- Status Toggle
- Media Upload

#### 4. Matching & Shortlist Page
**Purpose:** Match properties to leads

**Components:**
- Lead Selection
- Match Results
- Filter Options
- Property Cards
- Match Score Display
- Shortlist Builder
- Comparison View

#### 5. Reports Page
**Purpose:** Business analytics

**Components:**
- Report Type Selector
- Date Range Filter
- Data Table
- Charts & Graphs
- Export Options (PDF, CSV)
- Custom Report Builder

#### 6. Admin Panel
**Purpose:** System administration

**Components:**
- User Management
- System Configuration
- Backup & Restore
- Audit Log Viewer
- Permission Manager
- System Health

### UI Components

#### Navigation
```html
<nav class="sidebar">
  <ul class="nav-menu">
    <li><a href="#dashboard">Dashboard</a></li>
    <li><a href="#leads">Leads</a></li>
    <li><a href="#inventory">Inventory</a></li>
    <li><a href="#matching">Matching</a></li>
    <li><a href="#reports">Reports</a></li>
    <li><a href="#admin" class="admin-only">Admin</a></li>
  </ul>
</nav>
```

#### Data Table
```html
<table class="data-table sortable">
  <thead>
    <tr>
      <th>Lead ID</th>
      <th>Name</th>
      <th>Status</th>
      <th>Score</th>
      <th>Actions</th>
    </tr>
  </thead>
  <tbody id="tableBody">
    <!-- Rows populated by JS -->
  </tbody>
</table>
```

#### Form Components
```html
<form class="form-group">
  <div class="form-field">
    <label>Lead Name</label>
    <input type="text" id="leadName" required>
  </div>
  <div class="form-field">
    <label>Mobile Number</label>
    <input type="tel" id="mobile" pattern="[0-9]{10}" required>
  </div>
  <button type="submit" class="btn btn-primary">Create Lead</button>
</form>
```

#### Modal Dialog
```html
<div id="leadModal" class="modal">
  <div class="modal-content">
    <span class="close-btn">&times;</span>
    <h2>Lead Details</h2>
    <div id="modalBody"></div>
    <button class="btn btn-primary">Save</button>
    <button class="btn btn-secondary">Close</button>
  </div>
</div>
```

### JavaScript Functions

#### API Communication
```javascript
// Create lead
function createLead(formData) {
  google.script.run
    .withSuccessHandler(function(result) {
      showSuccess('Lead created: ' + result.leadId);
      loadLeads();
    })
    .withFailureHandler(showError)
    .createLead(formData);
}

// Get all leads
function loadLeads(filters = {}) {
  google.script.run
    .withSuccessHandler(displayLeads)
    .withFailureHandler(showError)
    .getAllLeads(filters);
}

// Display leads in table
function displayLeads(leads) {
  const tbody = document.getElementById('tableBody');
  tbody.innerHTML = leads.map(lead => `
    <tr>
      <td>${lead.LeadID}</td>
      <td>${lead.Name}</td>
      <td><span class="badge ${lead.Status}">${lead.Status}</span></td>
      <td>${lead.Score}</td>
      <td>
        <button onclick="editLead('${lead.LeadID}')">Edit</button>
        <button onclick="deleteLead('${lead.LeadID}')">Delete</button>
      </td>
    </tr>
  `).join('');
}
```

#### UI State Management
```javascript
// Show/hide pages
function showPage(pageName) {
  document.querySelectorAll('.page').forEach(p => p.style.display = 'none');
  document.getElementById(pageName).style.display = 'block';
  // Load page data
  switch(pageName) {
    case 'dashboard': loadDashboard(); break;
    case 'leads': loadLeads(); break;
    case 'inventory': loadInventory(); break;
  }
}

// Show notifications
function showSuccess(message) {
  const notification = document.createElement('div');
  notification.className = 'notification success';
  notification.textContent = message;
  document.body.appendChild(notification);
  setTimeout(() => notification.remove(), 3000);
}

function showError(error) {
  console.error(error);
  showSuccess('Error: ' + error.message);
}
```

### CSS Styling

#### Responsive Design
```css
/* Desktop */
.container {
  display: grid;
  grid-template-columns: 250px 1fr;
  gap: 20px;
}

/* Tablet */
@media (max-width: 768px) {
  .container {
    grid-template-columns: 1fr;
  }
  .sidebar {
    position: fixed;
    left: -250px;
    transition: left 0.3s;
  }
}

/* Mobile */
@media (max-width: 480px) {
  .table {
    font-size: 12px;
  }
  .btn {
    padding: 8px 12px;
  }
}
```

#### Color Theme
```css
:root {
  --primary-color: #2196F3;
  --success-color: #4CAF50;
  --warning-color: #FFC107;
  --danger-color: #f44336;
  --light-bg: #f5f5f5;
  --border-color: #ddd;
}
```

### Accessibility

- Semantic HTML elements
- ARIA labels for screen readers
- Keyboard navigation support
- Color contrast compliance
- Focus indicators

### Performance Optimization

1. **Lazy Loading**: Load pages on demand
2. **Caching**: Cache API results
3. **Pagination**: Limit table rows (50 per page)
4. **Minification**: Minify CSS and JS
5. **Debouncing**: Debounce search input

---

Next: See [10_FEATURE_LIST.md](10_FEATURE_LIST.md) for complete feature breakdown.
