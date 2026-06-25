# 15 - Negotiation Engine

## Deal Negotiation Management

### Negotiation Flow

```
Negotiation Process
        ↓
[Lead Interest] - Lead wants to buy
        ↓
[Initial Offer] - Lead makes offer
        ↓
[Owner Response] - Owner counters
        ↓
[Counter Rounds] - Multiple rounds
        ↓
[Deal Probability] - Calculate success %
        ↓
[Agreement] - Both agree on price
        ↓
[Token Received] - Deal confirmed
```

## Negotiation Record Structure

```javascript
const NegotiationRecord = {
  NegotiationID: String,      // Auto-generated
  LeadID: String,             // Link to lead
  PropertyID: String,         // Link to property
  OwnerID: String,            // Property owner
  AgentID: String,            // Handling agent
  PropertyPrice: Number,      // Original asking price
  InitialOffer: Number,       // Lead's first offer
  CounterOffer: Number,       // Owner's response
  FinalAmount: Number,        // Agreed amount
  Rounds: Number,             // Number of rounds
  Status: String,             // InProgress/Agreed/Failed
  DealProbability: Number,    // 0-100%
  CreatedDate: Date,
  LastUpdatedDate: Date,
  Notes: String               // Negotiation notes
};
```

## Negotiation Offer Tracking

```javascript
function recordInitialOffer(leadId, propertyId, offerAmount) {
  // 1. Create negotiation record
  const negotiation = {
    NegotiationID: generateUniqueId('NEG'),
    LeadID: leadId,
    PropertyID: propertyId,
    InitialOffer: offerAmount,
    PropertyPrice: getProperty(propertyId).Price,
    Status: 'InProgress',
    CreatedDate: new Date(),
    OfferHistory: []
  };
  
  // 2. Calculate initial discount
  const property = getProperty(propertyId);
  const discountPercentage = 
    ((property.Price - offerAmount) / property.Price) * 100;
  
  // 3. Record offer
  negotiation.OfferHistory.push({
    Type: 'INITIAL_OFFER',
    Amount: offerAmount,
    Discount: discountPercentage,
    By: 'Lead',
    Date: new Date()
  });
  
  // 4. Calculate initial deal probability
  negotiation.DealProbability = calculateInitialProbability(
    offerAmount, 
    property.Price, 
    discountPercentage
  );
  
  // 5. Store negotiation
  storeNegotiation(negotiation);
  
  // 6. Log activity
  logActivity('INITIAL_OFFER', leadId, propertyId, offerAmount);
  
  return negotiation;
}

function calculateInitialProbability(offerAmount, askingPrice, discountPercentage) {
  // If offer >= 95% of asking price: High probability
  if (discountPercentage <= 5) return 85;
  
  // If offer >= 90% of asking price: Medium-High
  if (discountPercentage <= 10) return 70;
  
  // If offer >= 85% of asking price: Medium
  if (discountPercentage <= 15) return 55;
  
  // If offer >= 75% of asking price: Medium-Low
  if (discountPercentage <= 25) return 40;
  
  // Below 75%: Low probability
  return 25;
}
```

## Counter-Offer Management

```javascript
function recordCounterOffer(negotiationId, counterAmount, notes = '') {
  const negotiation = getNegotiation(negotiationId);
  
  // 1. Record counter-offer
  negotiation.CounterOffer = counterAmount;
  negotiation.OfferHistory.push({
    Type: 'COUNTER_OFFER',
    Amount: counterAmount,
    Gap: negotiation.InitialOffer - counterAmount,
    By: 'Owner',
    Date: new Date(),
    Notes: notes
  });
  
  // 2. Increment round count
  negotiation.Rounds = (negotiation.Rounds || 0) + 1;
  
  // 3. Update deal probability
  negotiation.DealProbability = calculateUpdatedProbability(negotiation);
  
  // 4. Check for convergence
  const gapPercentage = 
    ((counterAmount - negotiation.InitialOffer) / counterAmount) * 100;
  
  // 5. Store updated record
  updateNegotiation(negotiation);
  
  // 6. Log activity
  logActivity('COUNTER_OFFER', negotiationId, counterAmount, notes);
  
  return negotiation;
}

function calculateUpdatedProbability(negotiation) {
  const rounds = negotiation.Rounds || 1;
  const gap = Math.abs(negotiation.CounterOffer - negotiation.InitialOffer);
  const property = getProperty(negotiation.PropertyID);
  const gapPercentage = (gap / property.Price) * 100;
  
  let probability = negotiation.DealProbability || 50;
  
  // Gap is closing: increase probability
  if (gapPercentage < 5) {
    probability = 90; // Very close to agreement
  } else if (gapPercentage < 10) {
    probability = 75;
  } else if (gapPercentage < 15) {
    probability = 60;
  } else if (gapPercentage < 20) {
    probability = 45;
  } else if (gapPercentage > 30) {
    probability = Math.max(25, probability - 10); // Gap widening
  }
  
  // More rounds = lower probability (deal fatigue)
  if (rounds > 5) probability *= 0.9;
  if (rounds > 10) probability *= 0.8;
  
  return Math.min(probability, 100);
}
```

## Deal Probability Calculation

```javascript
function calculateDealProbability(negotiationId) {
  const negotiation = getNegotiation(negotiationId);
  const property = getProperty(negotiation.PropertyID);
  const lead = getLead(negotiation.LeadID);
  
  let probability = 50; // Base probability
  
  // Factor 1: Offer Gap (0-30 points)
  const gap = Math.abs(negotiation.CounterOffer - negotiation.InitialOffer);
  const gapPercentage = (gap / property.Price) * 100;
  
  if (gapPercentage < 5) probability += 30;
  else if (gapPercentage < 10) probability += 20;
  else if (gapPercentage < 15) probability += 10;
  else if (gapPercentage < 20) probability += 5;
  
  // Factor 2: Negotiation Rounds (0-15 points)
  const rounds = negotiation.Rounds || 1;
  if (rounds === 1) probability += 15;
  else if (rounds === 2) probability += 12;
  else if (rounds === 3) probability += 8;
  else if (rounds > 5) probability -= (rounds - 5) * 5; // Fatigue
  
  // Factor 3: Lead Urgency (0-15 points)
  if (lead.Urgency === 'High') probability += 15;
  else if (lead.Urgency === 'Medium') probability += 8;
  
  // Factor 4: Time Invested (0-15 points)
  const siteVisits = getSiteVisitsForLead(lead.LeadID);
  if (siteVisits.length > 2) probability += 15;
  else if (siteVisits.length > 0) probability += 8;
  
  // Factor 5: Lead Score (0-15 points)
  if (lead.Score >= 80) probability += 15;
  else if (lead.Score >= 60) probability += 8;
  
  // Factor 6: Payment Method (0-10 points)
  if (negotiation.PaymentMethod === 'Cash') probability += 10;
  else if (negotiation.PaymentMethod === 'Cheque') probability += 5;
  
  return Math.min(probability, 100);
}
```

## Negotiation Status Tracking

```javascript
function updateNegotiationStatus(negotiationId, newStatus, reason = '') {
  const negotiation = getNegotiation(negotiationId);
  const oldStatus = negotiation.Status;
  
  // Validate status transition
  const validTransitions = {
    'InProgress': ['Agreed', 'Failed'],
    'Agreed': ['Failed'], // Can still fail after agreement
    'Failed': [] // Terminal state
  };
  
  if (!validTransitions[oldStatus].includes(newStatus)) {
    throw new Error(`Invalid status transition: ${oldStatus} -> ${newStatus}`);
  }
  
  // Update status
  negotiation.Status = newStatus;
  negotiation.StatusChangeDate = new Date();
  negotiation.StatusReason = reason;
  
  // Update deal-related records if agreed
  if (newStatus === 'Agreed') {
    // Calculate final commission
    calculateCommission(negotiation);
    
    // Update lead status
    updateLeadStatus(negotiation.LeadID, 'NEGOTIATING_COMPLETE');
    
    // Send notification
    sendNegotiationAgreedNotification(negotiationId);
  }
  
  // Handle failed deal
  if (newStatus === 'Failed') {
    // Don't reverse commission yet (can negotiate again)
    updateLeadStatus(negotiation.LeadID, 'DEAL_FAILED');
    
    // Send notification
    sendDealFailedNotification(negotiationId);
  }
  
  // Store updated record
  updateNegotiation(negotiation);
  
  // Log activity
  logActivity('NEGOTIATION_STATUS_CHANGED', negotiationId, 
    oldStatus, newStatus, reason);
  
  return negotiation;
}
```

## Discussion Logging

```javascript
function recordBuyerDiscussion(negotiationId, notes, proposedAmount = null) {
  const negotiation = getNegotiation(negotiationId);
  
  negotiation.DiscussionLog = negotiation.DiscussionLog || [];
  negotiation.DiscussionLog.push({
    Type: 'BUYER_DISCUSSION',
    Date: new Date(),
    Notes: notes,
    ProposedAmount: proposedAmount,
    Agent: getCurrentUser().Email
  });
  
  updateNegotiation(negotiation);
  logActivity('BUYER_DISCUSSION', negotiationId, notes);
}

function recordSellerDiscussion(negotiationId, notes, proposedAmount = null) {
  const negotiation = getNegotiation(negotiationId);
  
  negotiation.DiscussionLog = negotiation.DiscussionLog || [];
  negotiation.DiscussionLog.push({
    Type: 'SELLER_DISCUSSION',
    Date: new Date(),
    Notes: notes,
    ProposedAmount: proposedAmount,
    Agent: getCurrentUser().Email
  });
  
  updateNegotiation(negotiation);
  logActivity('SELLER_DISCUSSION', negotiationId, notes);
}
```

## Negotiation Analytics

```javascript
function getNegotiationStatistics() {
  const allNegotiations = getAllNegotiations();
  const agreedDeals = allNegotiations.filter(n => n.Status === 'Agreed');
  const failedDeals = allNegotiations.filter(n => n.Status === 'Failed');
  
  return {
    totalNegotiations: allNegotiations.length,
    successRate: (agreedDeals.length / allNegotiations.length) * 100,
    failureRate: (failedDeals.length / allNegotiations.length) * 100,
    averageRounds: calculateAverage(
      allNegotiations.map(n => n.Rounds)
    ),
    averageDiscount: calculateAverageDiscount(agreedDeals),
    averageDealProbability: calculateAverage(
      allNegotiations.map(n => n.DealProbability)
    ),
    agreementTime: calculateAverageAgreementTime(agreedDeals),
    topNegotiators: getTopAgentsByNegotiationSuccess(10)
  };
}

function calculateAverageDiscount(deals) {
  return deals.reduce((sum, deal) => {
    const discount = ((deal.PropertyPrice - deal.FinalAmount) / 
                      deal.PropertyPrice) * 100;
    return sum + discount;
  }, 0) / deals.length;
}
```

---

Next: See [16_TOKEN_DEAL_SYSTEM.md](16_TOKEN_DEAL_SYSTEM.md) for token & agreement management.
