# 14 - Matching Engine

## Advanced Lead-to-Property Matching

### Matching Algorithm Overview

```
Matching Process
    ↓
[Hard Filters] - Exact Match Requirements
    ↓
[Soft Filters] - Preference-Based Filtering
    ↓
[AI Scoring] - Match Quality Scoring
    ↓
[Ranking] - Sort by Score
    ↓
[Results] - Top 5-10 Matches
```

## Hard Filters (Exact Match)

**Purpose:** Filter properties that MUST match lead requirements

### Hard Filter Criteria

```javascript
function applyHardFilters(lead, properties) {
  return properties.filter(property => {
    // 1. Budget Match
    const budgetMatch = property.Price >= lead.MinBudget && 
                        property.Price <= lead.MaxBudget;
    
    // 2. Property Type Match
    const typeMatch = lead.PropertyType === property.Type;
    
    // 3. Location Match (must contain lead's preferred location)
    const locationMatch = lead.Locations.includes(property.Location);
    
    // 4. Area/BHK Match (if specified)
    const areaMatch = !lead.MinArea || 
                      (property.Area >= lead.MinArea && 
                       property.Area <= lead.MaxArea);
    
    // 5. Availability Match (not sold)
    const availabilityMatch = property.Status !== 'Sold';
    
    return budgetMatch && typeMatch && locationMatch && 
           areaMatch && availabilityMatch;
  });
}
```

## Soft Filters (Preference-Based)

**Purpose:** Filter based on secondary preferences

### Soft Filter Criteria

```javascript
function applySoftFilters(lead, properties) {
  return properties.filter(property => {
    let matches = true;
    
    // 1. Facing Preference
    if (lead.Facing && property.Facing !== lead.Facing) {
      matches = false; // Soft filter, but can still include
    }
    
    // 2. Parking Requirement
    if (lead.Parking && !property.Parking) {
      matches = false;
    }
    
    // 3. Amenities (at least 50% match)
    if (lead.Amenities) {
      const leadAmenities = lead.Amenities.split(',');
      const propAmenities = property.Amenities.split(',');
      const matchPercentage = calculateMatchPercentage(
        leadAmenities, propAmenities
      );
      if (matchPercentage < 50) {
        matches = false;
      }
    }
    
    // 4. Possession Timeline
    if (lead.Possession && lead.Possession !== 'Any') {
      if (property.Possession !== lead.Possession) {
        matches = false;
      }
    }
    
    return matches;
  });
}
```

## AI Scoring Algorithm

**Purpose:** Calculate match quality score (0-100)

### Score Calculation

```javascript
function calculateMatchScore(lead, property) {
  let score = 0;
  
  // 1. Budget Proximity (0-25 points)
  const budgetScore = calculateBudgetScore(lead, property);
  score += budgetScore; // 0-25
  
  // 2. Location Preference (0-20 points)
  const locationScore = calculateLocationScore(lead, property);
  score += locationScore; // 0-20
  
  // 3. Property Features (0-20 points)
  const featureScore = calculateFeatureScore(lead, property);
  score += featureScore; // 0-20
  
  // 4. Amenities Match (0-15 points)
  const amenitiesScore = calculateAmenitiesScore(lead, property);
  score += amenitiesScore; // 0-15
  
  // 5. Historical Match (0-10 points)
  const historyScore = calculateHistoryScore(lead, property);
  score += historyScore; // 0-10
  
  // 6. Availability & Status (0-10 points)
  const statusScore = calculateStatusScore(property);
  score += statusScore; // 0-10
  
  return Math.min(score, 100); // Cap at 100
}

// Sub-score calculations

function calculateBudgetScore(lead, property) {
  const leadBudget = (lead.MinBudget + lead.MaxBudget) / 2;
  const priceDiff = Math.abs(leadBudget - property.Price);
  const maxDiff = lead.MaxBudget - lead.MinBudget;
  
  if (priceDiff === 0) return 25; // Exact match
  if (priceDiff <= maxDiff * 0.1) return 20; // Within 10%
  if (priceDiff <= maxDiff * 0.2) return 15; // Within 20%
  if (priceDiff <= maxDiff * 0.5) return 10; // Within 50%
  return 5; // Beyond 50%
}

function calculateLocationScore(lead, property) {
  // 20 points: Primary location match
  if (property.Location === lead.Locations[0]) return 20;
  
  // 15 points: Any preferred location
  if (lead.Locations.includes(property.Location)) return 15;
  
  // 10 points: Same city, different location
  if (property.City === lead.City) return 10;
  
  // 5 points: Different city
  return 5;
}

function calculateFeatureScore(lead, property) {
  let score = 0;
  
  // BHK Match (0-10)
  if (lead.BHK && property.BHK === lead.BHK) score += 10;
  else if (lead.BHK && Math.abs(property.BHK - lead.BHK) === 1) score += 5;
  
  // Area Match (0-10)
  if (lead.MinArea && lead.MaxArea) {
    if (property.Area >= lead.MinArea && property.Area <= lead.MaxArea) {
      score += 10;
    } else if (Math.abs(property.Area - lead.MinArea) < 100) {
      score += 5;
    }
  }
  
  return score;
}

function calculateAmenitiesScore(lead, property) {
  if (!lead.Amenities) return 0;
  
  const leadAmenities = lead.Amenities.split(',').map(a => a.trim());
  const propAmenities = property.Amenities.split(',').map(a => a.trim());
  
  let matchCount = 0;
  leadAmenities.forEach(amenity => {
    if (propAmenities.includes(amenity)) matchCount++;
  });
  
  const matchPercentage = (matchCount / leadAmenities.length) * 100;
  return Math.floor((matchPercentage / 100) * 15); // 0-15 points
}

function calculateHistoryScore(lead, property) {
  // Check if lead has viewed this property before
  const viewHistory = getViewHistory(lead.LeadID, property.PropertyID);
  if (viewHistory) return 10; // Lead has shown interest before
  
  // Check successful matches from other leads
  const successRate = getPropertySuccessRate(property.PropertyID);
  if (successRate > 70) return 8;
  if (successRate > 50) return 5;
  if (successRate > 30) return 3;
  
  return 0;
}

function calculateStatusScore(property) {
  // Exclusive properties get bonus
  if (property.Status === 'Exclusive') return 10;
  
  // Recently added properties get bonus
  const daysSinceAdded = Math.floor(
    (new Date() - new Date(property.CreatedDate)) / (1000 * 60 * 60 * 24)
  );
  if (daysSinceAdded < 7) return 8; // New property
  if (daysSinceAdded < 30) return 5; // Recent property
  
  return 0;
}
```

## Complete Matching Function

```javascript
function matchLeadToProperties(leadId) {
  try {
    const lead = getLead(leadId);
    const requirement = getRequirement(leadId);
    
    if (!requirement) {
      Logger.log('No requirements found for lead: ' + leadId);
      return [];
    }
    
    // Get all available properties
    let properties = getAllProperties().filter(p => p.Status !== 'Sold');
    
    // 1. Apply hard filters
    properties = applyHardFilters(requirement, properties);
    Logger.log('After hard filters: ' + properties.length);
    
    // 2. Apply soft filters
    properties = applySoftFilters(requirement, properties);
    Logger.log('After soft filters: ' + properties.length);
    
    // 3. Calculate match scores
    const scoredProperties = properties.map(property => ({
      property: property,
      score: calculateMatchScore(requirement, property)
    }));
    
    // 4. Sort by score (descending)
    scoredProperties.sort((a, b) => b.score - a.score);
    
    // 5. Get top 10 matches
    const topMatches = scoredProperties.slice(0, 10);
    
    // 6. Store matches
    topMatches.forEach((match, index) => {
      storeMatch({
        LeadID: leadId,
        PropertyID: match.property.PropertyID,
        Score: match.score,
        Rank: index + 1,
        MatchDate: new Date(),
        Status: 'Generated'
      });
    });
    
    return topMatches;
    
  } catch (error) {
    Logger.log('Error in matching: ' + error);
    sendAdminAlert('MATCHING_ERROR', 
      'Lead matching failed for lead: ' + leadId + 
      ': ' + error.message);
    return [];
  }
}
```

## Hourly Matching Engine

```javascript
function hourlyMatchingEngine() {
  try {
    const startTime = new Date();
    Logger.log('Starting hourly matching at ' + startTime);
    
    // Get all qualified leads without matches
    const leads = getQualifiedLeads().filter(lead => {
      const matches = getExistingMatches(lead.LeadID);
      return !matches || matches.length === 0;
    });
    
    Logger.log('Processing ' + leads.length + ' leads');
    
    let processedCount = 0;
    let successCount = 0;
    let errorCount = 0;
    
    leads.forEach(lead => {
      try {
        const matches = matchLeadToProperties(lead.LeadID);
        if (matches && matches.length > 0) {
          successCount++;
          
          // Send shortlist to lead if configured
          if (CONFIG.AUTO_SEND_SHORTLIST) {
            sendShortlistToLead(lead.LeadID, matches);
          }
        }
      } catch (error) {
        errorCount++;
        Logger.log('Error processing lead ' + lead.LeadID + ': ' + error);
      }
      processedCount++;
    });
    
    const endTime = new Date();
    const duration = (endTime - startTime) / 1000; // seconds
    
    // Log results
    logAutomationRun('hourlyMatchingEngine', {
      processedLeads: processedCount,
      successfulMatches: successCount,
      errors: errorCount,
      durationSeconds: duration,
      timestamp: new Date()
    });
    
    Logger.log('Hourly matching completed in ' + duration + ' seconds');
    
  } catch (error) {
    Logger.log('Critical error in hourly matching: ' + error);
    sendAdminAlert('MATCHING_ENGINE_FAILURE', 
      'Hourly matching failed: ' + error.message);
  }
}
```

## Match Feedback

```javascript
function recordMatchFeedback(leadId, propertyId, feedback) {
  // 1. Store feedback
  storeMatchFeedback({
    LeadID: leadId,
    PropertyID: propertyId,
    Feedback: feedback, // 'Interested', 'Not Interested', 'Send More'
    FeedbackDate: new Date(),
    Agent: getCurrentUser().Email
  });
  
  // 2. Update match score based on feedback
  if (feedback === 'Interested') {
    incrementPropertySuccessRate(propertyId);
  } else if (feedback === 'Not Interested') {
    decrementPropertySuccessRate(propertyId);
  }
  
  // 3. Update AI model with feedback
  updateAIModel(leadId, propertyId, feedback);
  
  logActivity('MATCH_FEEDBACK', leadId, propertyId, feedback);
}
```

## Match Statistics

```javascript
function getMatchStatistics() {
  const allMatches = getAllMatches();
  
  return {
    totalMatches: allMatches.length,
    averageScore: calculateAverage(allMatches.map(m => m.Score)),
    highQualityMatches: allMatches.filter(m => m.Score >= 80).length,
    mediumQualityMatches: allMatches.filter(m => m.Score >= 50 && m.Score < 80).length,
    lowQualityMatches: allMatches.filter(m => m.Score < 50).length,
    matchSuccessRate: calculateSuccessRate(),
    topProperties: getTopProperties(10),
    topLeads: getTopLeads(10)
  };
}
```

---

Next: See [15_NEGOTIATION_ENGINE.md](15_NEGOTIATION_ENGINE.md) for negotiation tracking.
