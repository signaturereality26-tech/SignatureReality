# 17 - Commission System

## Commission Calculation & Management

### Commission Structure

```
Deal Amount (Final Price)
        ↓
[Commission Percentage]
        ↓
[Gross Commission]
        ↓
[Agent Split]
        ↓
[GST Calculation]
        ↓
[TDS Deduction]
        ↓
[Net Commission]
        ↓
[Payment Status]
```

## Commission Record Structure

```javascript
const CommissionRecord = {
  CommissionID: String,          // Auto-generated
  DealID: String,                // Link to agreement
  LeadID: String,                // Buyer
  PropertyID: String,            // Property
  AgentID: String,               // Commission agent
  DealAmount: Number,            // Final sale price
  CommissionPercentage: Number,  // % of deal amount
  GrossCommission: Number,       // Before GST/TDS
  GST: Number,                   // 18% of commission
  TDS: Number,                   // Tax deducted at source
  NetCommission: Number,         // Final amount
  AgentSplit: Number,            // Agent's share
  CompanySplit: Number,          // Company's share
  Status: String,                // Pending/Received/Held
  PaymentDate: Date,
  DueDate: Date,
  CreatedDate: Date
};
```

## Commission Calculation

```javascript
function calculateCommission(dealData) {
  try {
    // 1. Get deal details
    const agreement = getAgreement(dealData.agreementId);
    const lead = getLead(agreement.LeadID);
    const agent = getAgent(lead.AssignedAgent);
    
    if (!agreement || !agent) {
      throw new Error('Invalid deal or agent details');
    }
    
    // 2. Determine commission percentage (configurable)
    const commissionPercentage = getCommissionPercentage(
      agreement.PropertyID,
      agent.AgentID
    );
    
    // 3. Calculate gross commission
    const grossCommission = (agreement.FinalAmount * commissionPercentage) / 100;
    
    // 4. Calculate GST (18%)
    const gstAmount = (grossCommission * 18) / 100;
    
    // 5. Calculate TDS (if applicable - 5% on commission)
    const tdsAmount = (grossCommission * 5) / 100;
    
    // 6. Calculate net commission
    const netCommission = grossCommission + gstAmount - tdsAmount;
    
    // 7. Agent-Company split
    // Example: 60% to agent, 40% to company
    const agentSplit = (netCommission * agent.CommissionSplit) / 100;
    const companySplit = netCommission - agentSplit;
    
    // 8. Create commission record
    const commission = {
      CommissionID: generateUniqueId('COMM'),
      DealID: dealData.agreementId,
      LeadID: agreement.LeadID,
      PropertyID: agreement.PropertyID,
      AgentID: agent.AgentID,
      DealAmount: agreement.FinalAmount,
      CommissionPercentage: commissionPercentage,
      GrossCommission: grossCommission,
      GST: gstAmount,
      TDS: tdsAmount,
      NetCommission: netCommission,
      AgentSplit: agentSplit,
      CompanySplit: companySplit,
      Status: 'Pending',
      CreatedDate: new Date(),
      DueDate: calculateDueDate(new Date())
    };
    
    // 9. Store commission
    storeCommission(commission);
    
    // 10. Update agent stats
    updateAgentCommissionStats(agent.AgentID, commission);
    
    // 11. Log activity
    logActivity('COMMISSION_CALCULATED', agreement.LeadID, 
      agreement.PropertyID, commission.CommissionID);
    
    return commission;
    
  } catch (error) {
    Logger.log('Error calculating commission: ' + error);
    throw error;
  }
}

function getCommissionPercentage(propertyId, agentId) {
  // Check property-specific commission
  const propCommission = getPropertyCommissionPercentage(propertyId);
  if (propCommission) return propCommission;
  
  // Check agent-specific commission
  const agentCommission = getAgentCommissionPercentage(agentId);
  if (agentCommission) return agentCommission;
  
  // Default commission (e.g., 2%)
  return CONFIG.DEFAULT_COMMISSION_PERCENTAGE || 2;
}

function calculateDueDate(commissionDate) {
  // Commission due 15 days after deal closure
  const dueDate = new Date(commissionDate);
  dueDate.setDate(dueDate.getDate() + 15);
  return dueDate;
}
```

## Commission Payment Processing

```javascript
function processCommissionPayment(commissionId, paymentData) {
  try {
    const commission = getCommission(commissionId);
    
    // 1. Validate commission status
    if (commission.Status !== 'Pending' && commission.Status !== 'Held') {
      throw new Error(`Cannot pay commission with status: ${commission.Status}`);
    }
    
    // 2. Validate payment amount
    if (paymentData.amount !== commission.NetCommission) {
      throw new Error('Payment amount mismatch');
    }
    
    // 3. Record payment
    const payment = {
      PaymentID: generateUniqueId('PAY'),
      CommissionID: commissionId,
      Amount: paymentData.amount,
      PaymentMethod: paymentData.method, // Bank Transfer, Cheque, Cash
      PaymentDate: new Date(),
      TransactionID: paymentData.transactionId,
      ProcessedBy: getCurrentUser().Email,
      Status: 'Processed'
    };
    
    // 4. Update commission status
    commission.Status = 'Received';
    commission.PaymentDate = payment.PaymentDate;
    commission.PaymentID = payment.PaymentID;
    
    // 5. Store payment record
    storePayment(payment);
    updateCommission(commission);
    
    // 6. Update agent balance
    updateAgentBalance(commission.AgentID, commission.AgentSplit);
    
    // 7. Update company revenue
    updateCompanyRevenue(commission.CompanySplit);
    
    // 8. Send payment notification
    sendPaymentConfirmationNotification(commission.AgentID, payment);
    
    // 9. Log activity
    logActivity('COMMISSION_PAID', commission.LeadID, 
      commissionId, commission.NetCommission);
    
    return payment;
    
  } catch (error) {
    Logger.log('Error processing payment: ' + error);
    throw error;
  }
}
```

## Commission Splits (Multi-Agent)

```javascript
function createCommissionSplit(dealData, agents) {
  try {
    const agreement = getAgreement(dealData.agreementId);
    const grossCommission = dealData.grossCommission;
    
    // 1. Validate agents
    if (!agents || agents.length === 0) {
      throw new Error('No agents specified for split');
    }
    
    // 2. Calculate individual splits
    const splits = [];
    let totalPercentage = 0;
    
    agents.forEach(agent => {
      const splitPercentage = agent.percentage;
      totalPercentage += splitPercentage;
      
      const splitAmount = (grossCommission * splitPercentage) / 100;
      
      splits.push({
        AgentID: agent.agentId,
        SplitPercentage: splitPercentage,
        SplitAmount: splitAmount,
        Role: agent.role // Lead Agent, Co-Agent, Referral
      });
    });
    
    // 3. Validate total splits to 100%
    if (totalPercentage !== 100) {
      throw new Error(
        `Commission splits don't add up to 100% (Total: ${totalPercentage}%)`
      );
    }
    
    // 4. Store split records
    splits.forEach(split => {
      storeCommissionSplit({
        SplitID: generateUniqueId('SPLIT'),
        DealID: agreement.AgreementID,
        ...split,
        CreatedDate: new Date()
      });
    });
    
    // 5. Log activity
    logActivity('COMMISSION_SPLIT_CREATED', agreement.LeadID, 
      `Split among ${agents.length} agents`);
    
    return splits;
    
  } catch (error) {
    Logger.log('Error creating commission split: ' + error);
    throw error;
  }
}
```

## Agent Commission Summary

```javascript
function getAgentCommissionSummary(agentId, fromDate, toDate) {
  try {
    // 1. Get all commissions for agent in date range
    const commissions = getCommissionsByAgent(agentId, fromDate, toDate);
    
    // 2. Calculate totals
    const summary = {
      AgentID: agentId,
      PeriodStart: fromDate,
      PeriodEnd: toDate,
      TotalDeals: commissions.length,
      TotalDealAmount: 0,
      TotalGrossCommission: 0,
      TotalGST: 0,
      TotalTDS: 0,
      TotalNetCommission: 0,
      TotalAgentEarnings: 0,
      PendingCommission: 0,
      ReceivedCommission: 0,
      CommissionByStatus: {},
      TopProperty: null,
      AverageCommission: 0
    };
    
    // 3. Aggregate data
    commissions.forEach(commission => {
      summary.TotalDealAmount += commission.DealAmount;
      summary.TotalGrossCommission += commission.GrossCommission;
      summary.TotalGST += commission.GST;
      summary.TotalTDS += commission.TDS;
      summary.TotalNetCommission += commission.NetCommission;
      summary.TotalAgentEarnings += commission.AgentSplit;
      
      if (commission.Status === 'Pending' || commission.Status === 'Held') {
        summary.PendingCommission += commission.NetCommission;
      } else if (commission.Status === 'Received') {
        summary.ReceivedCommission += commission.NetCommission;
      }
      
      // Count by status
      summary.CommissionByStatus[commission.Status] = 
        (summary.CommissionByStatus[commission.Status] || 0) + 1;
    });
    
    // 4. Calculate average
    if (commissions.length > 0) {
      summary.AverageCommission = 
        summary.TotalNetCommission / commissions.length;
    }
    
    // 5. Find top property
    if (commissions.length > 0) {
      const topDeal = commissions.reduce((max, c) => 
        c.DealAmount > max.DealAmount ? c : max
      );
      summary.TopProperty = getProperty(topDeal.PropertyID).PropertyName;
    }
    
    return summary;
    
  } catch (error) {
    Logger.log('Error generating commission summary: ' + error);
    throw error;
  }
}
```

## Commission Analytics

```javascript
function getCommissionAnalytics() {
  const allCommissions = getAllCommissions();
  
  return {
    totalCommissions: allCommissions.length,
    totalCommissionValue: allCommissions.reduce(
      (sum, c) => sum + c.NetCommission, 0
    ),
    averageCommissionPerDeal: calculateAverage(
      allCommissions.map(c => c.NetCommission)
    ),
    pendingAmount: allCommissions
      .filter(c => c.Status === 'Pending' || c.Status === 'Held')
      .reduce((sum, c) => sum + c.NetCommission, 0),
    paidAmount: allCommissions
      .filter(c => c.Status === 'Received')
      .reduce((sum, c) => sum + c.NetCommission, 0),
    totalTDS: allCommissions.reduce((sum, c) => sum + c.TDS, 0),
    totalGST: allCommissions.reduce((sum, c) => sum + c.GST, 0),
    topEarners: getTopEarningAgents(10),
    highestDeal: getHighestDealCommission(),
    averageSettlementTime: calculateAverageSettlementTime()
  };
}

function getTopEarningAgents(limit = 10) {
  const agents = getAllAgents();
  const earnings = agents.map(agent => {
    const commissions = getCommissionsByAgent(agent.AgentID);
    const totalEarnings = commissions.reduce(
      (sum, c) => sum + c.AgentSplit, 0
    );
    
    return {
      AgentID: agent.AgentID,
      AgentName: agent.Name,
      TotalEarnings: totalEarnings,
      DealCount: commissions.length,
      AveragePerDeal: totalEarnings / commissions.length
    };
  });
  
  return earnings
    .sort((a, b) => b.TotalEarnings - a.TotalEarnings)
    .slice(0, limit);
}
```

---

Next: See [18_REPORTING_SYSTEM.md](18_REPORTING_SYSTEM.md) for comprehensive reporting.
