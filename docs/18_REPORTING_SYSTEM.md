# 18 - Reporting System

## Comprehensive CRM Reporting

### Report Types & Categories

```
Reporting System
        |
    ┌───┴───┬───────────┬─────────┬──────────┐
    |       |           |         |          |
  Lead   Sales      Revenue    Agent     Inventory
 Reports Reports    Reports   Reports     Reports
```

## Lead Report

```javascript
function generateLeadReport(fromDate, toDate, filters = {}) {
  try {
    // 1. Get all leads in date range
    let leads = getLeadsByDateRange(fromDate, toDate);
    
    // 2. Apply filters
    if (filters.source) {
      leads = leads.filter(l => l.Source === filters.source);
    }
    if (filters.status) {
      leads = leads.filter(l => l.Status === filters.status);
    }
    if (filters.agent) {
      leads = leads.filter(l => l.AssignedAgent === filters.agent);
    }
    
    // 3. Build report data
    const report = {
      ReportID: generateUniqueId('REPORT'),
      ReportType: 'LEAD_REPORT',
      GeneratedDate: new Date(),
      DateRange: { fromDate, toDate },
      Filters: filters,
      Summary: {
        TotalLeads: leads.length,
        NewLeads: leads.filter(l => l.Status === 'New').length,
        QualifiedLeads: leads.filter(l => l.Score >= 60).length,
        ConvertedLeads: leads.filter(l => l.Status === 'Deal_Closed').length,
        LostLeads: leads.filter(l => l.Status === 'Lost').length,
        ConversionRate: 0 // Will calculate below
      },
      LeadsBySource: groupAndCount(leads, 'Source'),
      LeadsByStatus: groupAndCount(leads, 'Status'),
      LeadsByAgent: groupAndCount(leads, 'AssignedAgent'),
      LeadsByScore: {
        Hot: leads.filter(l => l.Score >= 80).length,
        Warm: leads.filter(l => l.Score >= 60 && l.Score < 80).length,
        Cold: leads.filter(l => l.Score < 60).length
      },
      LeadDetails: leads.map(l => ({
        LeadID: l.LeadID,
        Name: l.Name,
        Source: l.Source,
        Score: l.Score,
        Status: l.Status,
        Agent: l.AssignedAgent,
        CreatedDate: l.CreatedDate
      }))
    };
    
    // 4. Calculate conversion rate
    report.Summary.ConversionRate = 
      (report.Summary.ConvertedLeads / report.Summary.TotalLeads) * 100;
    
    // 5. Store report
    storeReport(report);
    
    // 6. Generate PDF
    const pdf = generateReportPDF(report);
    report.PDFUrl = uploadToDrive(pdf, 'Reports');
    
    return report;
    
  } catch (error) {
    Logger.log('Error generating lead report: ' + error);
    throw error;
  }
}
```

## Sales Report

```javascript
function generateSalesReport(fromDate, toDate, filters = {}) {
  try {
    // 1. Get all closed deals
    let agreements = getAgreementsByDateRange(fromDate, toDate)
      .filter(a => a.Status === 'Confirmed');
    
    // 2. Apply filters
    if (filters.agent) {
      const leads = agreements.map(a => getLead(a.LeadID));
      agreements = agreements.filter((a, i) => 
        leads[i].AssignedAgent === filters.agent
      );
    }
    if (filters.property) {
      agreements = agreements.filter(a => a.PropertyID === filters.property);
    }
    
    // 3. Build report
    const report = {
      ReportID: generateUniqueId('REPORT'),
      ReportType: 'SALES_REPORT',
      GeneratedDate: new Date(),
      DateRange: { fromDate, toDate },
      Summary: {
        TotalDeals: agreements.length,
        TotalSalesValue: 0,
        AverageDealSize: 0,
        HighestDeal: 0,
        LowestDeal: 0
      },
      SalesByAgent: {},
      SalesByProperty: {},
      DealDetails: []
    };
    
    // 4. Aggregate sales data
    let totalSales = 0;
    
    agreements.forEach(agreement => {
      const lead = getLead(agreement.LeadID);
      const agent = lead.AssignedAgent;
      const dealAmount = agreement.FinalAmount;
      
      totalSales += dealAmount;
      
      // By Agent
      if (!report.SalesByAgent[agent]) {
        report.SalesByAgent[agent] = { Count: 0, Amount: 0 };
      }
      report.SalesByAgent[agent].Count++;
      report.SalesByAgent[agent].Amount += dealAmount;
      
      // By Property
      const property = getProperty(agreement.PropertyID);
      if (!report.SalesByProperty[property.Location]) {
        report.SalesByProperty[property.Location] = { Count: 0, Amount: 0 };
      }
      report.SalesByProperty[property.Location].Count++;
      report.SalesByProperty[property.Location].Amount += dealAmount;
      
      // Track high/low
      if (dealAmount > report.Summary.HighestDeal) 
        report.Summary.HighestDeal = dealAmount;
      if (!report.Summary.LowestDeal || dealAmount < report.Summary.LowestDeal)
        report.Summary.LowestDeal = dealAmount;
      
      // Deal details
      report.DealDetails.push({
        DealID: agreement.AgreementID,
        LeadName: lead.Name,
        Agent: agent,
        Property: property.PropertyName,
        Amount: dealAmount,
        ClosureDate: agreement.ConfirmationDate
      });
    });
    
    report.Summary.TotalSalesValue = totalSales;
    report.Summary.AverageDealSize = 
      agreements.length > 0 ? totalSales / agreements.length : 0;
    
    storeReport(report);
    return report;
    
  } catch (error) {
    Logger.log('Error generating sales report: ' + error);
    throw error;
  }
}
```

## Revenue Report

```javascript
function generateRevenueReport(fromDate, toDate, filters = {}) {
  try {
    // 1. Get all commission payments
    const payments = getPaymentsByDateRange(fromDate, toDate)
      .filter(p => p.Status === 'Processed');
    
    // 2. Calculate revenue
    const report = {
      ReportID: generateUniqueId('REPORT'),
      ReportType: 'REVENUE_REPORT',
      GeneratedDate: new Date(),
      DateRange: { fromDate, toDate },
      Summary: {
        TotalGrossCommission: 0,
        TotalGST: 0,
        TotalTDS: 0,
        TotalNetCommission: 0,
        CompanyRevenue: 0,
        AgentPayouts: 0
      },
      RevenueByAgent: {},
      PaymentBreakdown: {
        ByMethod: {},
        ByStatus: {}
      },
      TopRevenueGenerators: []
    };
    
    // 3. Process payments
    payments.forEach(payment => {
      const commission = getCommission(payment.CommissionID);
      
      report.Summary.TotalGrossCommission += commission.GrossCommission;
      report.Summary.TotalGST += commission.GST;
      report.Summary.TotalTDS += commission.TDS;
      report.Summary.TotalNetCommission += commission.NetCommission;
      report.Summary.CompanyRevenue += commission.CompanySplit;
      report.Summary.AgentPayouts += commission.AgentSplit;
      
      // By Agent
      const agent = commission.AgentID;
      if (!report.RevenueByAgent[agent]) {
        report.RevenueByAgent[agent] = {
          AgentName: getAgent(agent).Name,
          Deals: 0,
          Revenue: 0
        };
      }
      report.RevenueByAgent[agent].Deals++;
      report.RevenueByAgent[agent].Revenue += commission.AgentSplit;
      
      // Payment method
      const method = payment.PaymentMethod;
      report.PaymentBreakdown.ByMethod[method] = 
        (report.PaymentBreakdown.ByMethod[method] || 0) + payment.Amount;
    });
    
    // 4. Top revenue generators
    report.TopRevenueGenerators = Object.values(report.RevenueByAgent)
      .sort((a, b) => b.Revenue - a.Revenue)
      .slice(0, 10);
    
    storeReport(report);
    return report;
    
  } catch (error) {
    Logger.log('Error generating revenue report: ' + error);
    throw error;
  }
}
```

## Agent Performance Report

```javascript
function generateAgentReport(fromDate, toDate, agentId = null) {
  try {
    const agents = agentId ? [getAgent(agentId)] : getAllAgents();
    
    const report = {
      ReportID: generateUniqueId('REPORT'),
      ReportType: 'AGENT_REPORT',
      GeneratedDate: new Date(),
      DateRange: { fromDate, toDate },
      Agents: []
    };
    
    agents.forEach(agent => {
      // Get agent's leads
      const leads = getLeadsByAgent(agent.AgentID, fromDate, toDate);
      const closedDeals = leads.filter(l => l.Status === 'Deal_Closed');
      const commissions = getCommissionsByAgent(agent.AgentID);
      
      report.Agents.push({
        AgentID: agent.AgentID,
        AgentName: agent.Name,
        Email: agent.Email,
        Performance: {
          TotalLeads: leads.length,
          ClosedDeals: closedDeals.length,
          ConversionRate: (closedDeals.length / leads.length) * 100,
          TotalRevenue: closedDeals.reduce(
            (sum, d) => sum + d.DealAmount, 0
          ),
          TotalEarnings: commissions.reduce(
            (sum, c) => sum + c.AgentSplit, 0
          ),
          AverageDealSize: closedDeals.length > 0 ?
            closedDeals.reduce((sum, d) => sum + d.DealAmount, 0) / 
            closedDeals.length : 0
        },
        Rating: calculateAgentRating(agent.AgentID)
      });
    });
    
    storeReport(report);
    return report;
    
  } catch (error) {
    Logger.log('Error generating agent report: ' + error);
    throw error;
  }
}

function calculateAgentRating(agentId) {
  const leads = getLeadsByAgent(agentId);
  const closedDeals = leads.filter(l => l.Status === 'Deal_Closed');
  const conversionRate = (closedDeals.length / leads.length) * 100;
  
  // Rating out of 5
  if (conversionRate >= 80) return 5;
  if (conversionRate >= 60) return 4;
  if (conversionRate >= 40) return 3;
  if (conversionRate >= 20) return 2;
  return 1;
}
```

## Inventory Report

```javascript
function generateInventoryReport() {
  try {
    const properties = getAllProperties();
    
    const report = {
      ReportID: generateUniqueId('REPORT'),
      ReportType: 'INVENTORY_REPORT',
      GeneratedDate: new Date(),
      Summary: {
        TotalProperties: properties.length,
        ActiveListings: 0,
        SoldProperties: 0,
        OnHold: 0,
        Exclusive: 0,
        TotalValue: 0
      },
      ByLocation: {},
      ByType: {},
      ByOwner: {},
      PropertyDetails: []
    };
    
    properties.forEach(property => {
      // Count by status
      if (property.Status === 'Active') report.Summary.ActiveListings++;
      if (property.Status === 'Sold') report.Summary.SoldProperties++;
      if (property.Status === 'Hold') report.Summary.OnHold++;
      if (property.Status === 'Exclusive') report.Summary.Exclusive++;
      
      report.Summary.TotalValue += property.Price;
      
      // By Location
      report.ByLocation[property.Location] = 
        (report.ByLocation[property.Location] || 0) + 1;
      
      // By Type
      report.ByType[property.Type] = 
        (report.ByType[property.Type] || 0) + 1;
      
      // By Owner
      report.ByOwner[property.Owner] = 
        (report.ByOwner[property.Owner] || 0) + 1;
      
      // Details
      report.PropertyDetails.push({
        PropertyID: property.PropertyID,
        Name: property.PropertyName,
        Location: property.Location,
        Type: property.Type,
        Price: property.Price,
        Status: property.Status,
        Owner: property.Owner
      });
    });
    
    storeReport(report);
    return report;
    
  } catch (error) {
    Logger.log('Error generating inventory report: ' + error);
    throw error;
  }
}
```

## Dashboard Metrics

```javascript
function getDashboardMetrics() {
  const today = new Date();
  const thisMonth = new Date(today.getFullYear(), today.getMonth(), 1);
  const thisYear = new Date(today.getFullYear(), 0, 1);
  
  return {
    Today: {
      NewLeads: getLeadsByDate(today).length,
      SiteVisits: getSiteVisitsByDate(today).length,
      TokensReceived: getTokensByDate(today).length
    },
    ThisMonth: {
      NewLeads: getLeadsByDateRange(thisMonth, today).length,
      DealsClosedValue: calculateDealsClosedValue(thisMonth, today),
      CommissionPaid: calculateCommissionPaid(thisMonth, today)
    },
    ThisYear: {
      TotalDeals: getAgreementsByDateRange(thisYear, today)
        .filter(a => a.Status === 'Confirmed').length,
      TotalRevenue: calculateTotalRevenue(thisYear, today),
      TopAgent: getTopAgent(thisYear, today)
    }
  };
}
```

---

Next: See [19_SECURITY.md](19_SECURITY.md) for security framework.
