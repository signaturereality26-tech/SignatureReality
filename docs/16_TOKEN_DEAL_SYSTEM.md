# 16 - Token Deal System

## Token Receipt & Agreement Management

### Token Receipt Process

```
Token System
     ↓
[Agreement Reached]
     ↓
[Token Receipt Generation]
     ↓
[Document Storage]
     ↓
[Signature Collection]
     ↓
[Deal Confirmation]
```

## Token Receipt Structure

```javascript
const TokenReceipt = {
  TokenID: String,                // Auto-generated
  LeadID: String,                 // Link to lead
  PropertyID: String,             // Link to property
  TokenAmount: Number,            // Token amount
  TokenPercentage: Number,        // % of final price
  ReceiptDate: Date,              // Date received
  ReceiptNumber: String,          // Receipt number
  ReceivedBy: String,             // Agent name
  PaymentMethod: String,          // Cash/Cheque/Transfer
  ChequeDetails: {
    ChequeNumber: String,
    ChequeBank: String,
    ChequeDate: Date
  },
  TransferDetails: {
    TransactionID: String,
    BankName: String,
    TransactionTime: Date
  },
  DocumentLinks: [String],        // Google Drive URLs
  Witnesses: [{
    Name: String,
    Contact: String,
    Signature: String
  }],
  Signature: {
    BuyerSignature: String,
    SellerSignature: String,
    SignatureDate: Date
  },
  Status: String,                 // Received/Cancelled/Disputed
  Remarks: String
};
```

## Token Receipt Generation

```javascript
function createTokenReceipt(tokenData) {
  try {
    // 1. Validate token amount
    if (tokenData.TokenAmount <= 0) {
      throw new Error('Invalid token amount');
    }
    
    // 2. Validate agreement exists
    const negotiation = getNegotiation(tokenData.NegotiationID);
    if (!negotiation || negotiation.Status !== 'Agreed') {
      throw new Error('No valid agreement found');
    }
    
    // 3. Create token record
    const token = {
      TokenID: generateUniqueId('TOKEN'),
      LeadID: tokenData.LeadID,
      PropertyID: tokenData.PropertyID,
      NegotiationID: tokenData.NegotiationID,
      TokenAmount: tokenData.TokenAmount,
      FinalAmount: negotiation.FinalAmount,
      TokenPercentage: (tokenData.TokenAmount / 
                        negotiation.FinalAmount) * 100,
      ReceiptDate: new Date(),
      ReceivedBy: getCurrentUser().Email,
      PaymentMethod: tokenData.PaymentMethod,
      Status: 'Received',
      CreatedDate: new Date()
    };
    
    // 4. Store payment details based on method
    if (tokenData.PaymentMethod === 'Cheque') {
      token.ChequeDetails = {
        ChequeNumber: tokenData.ChequeNumber,
        ChequeBank: tokenData.ChequeBank,
        ChequeDate: tokenData.ChequeDate
      };
    } else if (tokenData.PaymentMethod === 'Transfer') {
      token.TransferDetails = {
        TransactionID: tokenData.TransactionID,
        BankName: tokenData.BankName,
        TransactionTime: new Date()
      };
    }
    
    // 5. Generate PDF receipt
    const receiptPDF = generateTokenReceiptPDF(token);
    token.ReceiptURL = uploadToDrive(receiptPDF, 'TokenReceipts');
    
    // 6. Store token record
    storeTokenReceipt(token);
    
    // 7. Update lead status
    updateLeadStatus(tokenData.LeadID, 'TOKEN_RECEIVED');
    
    // 8. Log activity
    logActivity('TOKEN_RECEIVED', tokenData.LeadID, 
      tokenData.PropertyID, tokenData.TokenAmount);
    
    // 9. Send notification
    sendTokenReceivedNotification(token);
    
    return token;
    
  } catch (error) {
    Logger.log('Error creating token receipt: ' + error);
    logActivity('TOKEN_CREATION_FAILED', tokenData.LeadID, error.message);
    throw error;
  }
}
```

## Token Receipt PDF Generation

```javascript
function generateTokenReceiptPDF(token) {
  const lead = getLead(token.LeadID);
  const property = getProperty(token.PropertyID);
  const negotiation = getNegotiation(token.NegotiationID);
  
  const html = `
    <html>
      <head>
        <title>Token Receipt</title>
        <style>
          body { font-family: Arial; margin: 20px; }
          .header { text-align: center; font-size: 20px; font-weight: bold; }
          .section { margin-top: 20px; }
          .label { font-weight: bold; }
          table { width: 100%; border-collapse: collapse; }
          td { border: 1px solid #ddd; padding: 8px; }
        </style>
      </head>
      <body>
        <div class="header">TOKEN RECEIPT</div>
        
        <div class="section">
          <p><span class="label">Receipt Number:</span> ${token.TokenID}</p>
          <p><span class="label">Date:</span> ${formatDate(token.ReceiptDate)}</p>
        </div>
        
        <div class="section">
          <h3>Buyer Information</h3>
          <p><span class="label">Name:</span> ${lead.Name}</p>
          <p><span class="label">Mobile:</span> ${lead.Mobile}</p>
          <p><span class="label">Email:</span> ${lead.Email}</p>
        </div>
        
        <div class="section">
          <h3>Property Information</h3>
          <p><span class="label">Property:</span> ${property.PropertyName}</p>
          <p><span class="label">Address:</span> ${property.Address}</p>
          <p><span class="label">Price:</span> ₹${formatCurrency(negotiation.FinalAmount)}</p>
        </div>
        
        <div class="section">
          <h3>Token Details</h3>
          <p><span class="label">Token Amount:</span> ₹${formatCurrency(token.TokenAmount)}</p>
          <p><span class="label">Token %:</span> ${token.TokenPercentage.toFixed(2)}%</p>
          <p><span class="label">Payment Method:</span> ${token.PaymentMethod}</p>
          ${token.ChequeDetails ? `
            <p><span class="label">Cheque No:</span> ${token.ChequeDetails.ChequeNumber}</p>
            <p><span class="label">Bank:</span> ${token.ChequeDetails.ChequeBank}</p>
          ` : ''}
        </div>
        
        <div class="section">
          <p>This is to certify that ₹${formatCurrency(token.TokenAmount)} has been received as token for the above property.</p>
        </div>
        
        <div class="section" style="margin-top: 50px;">
          <p>Agent: ${token.ReceivedBy}</p>
          <p>Signature: ___________________</p>
        </div>
      </body>
    </html>
  `;
  
  return convertHtmlToPdf(html);
}
```

## Agreement Management

```javascript
function createAgreement(tokenId, agreementData) {
  try {
    const token = getTokenReceipt(tokenId);
    const lead = getLead(token.LeadID);
    const property = getProperty(token.PropertyID);
    const negotiation = getNegotiation(token.NegotiationID);
    
    // 1. Create agreement record
    const agreement = {
      AgreementID: generateUniqueId('AGREE'),
      TokenID: tokenId,
      LeadID: token.LeadID,
      PropertyID: token.PropertyID,
      AgreementDate: new Date(),
      FinalAmount: negotiation.FinalAmount,
      TokenAmount: token.TokenAmount,
      BalanceAmount: negotiation.FinalAmount - token.TokenAmount,
      PaymentTerms: agreementData.PaymentTerms,
      PossessionDate: agreementData.PossessionDate,
      Status: 'Draft',
      CreatedBy: getCurrentUser().Email
    };
    
    // 2. Upload agreement template
    const agreementDoc = uploadAgreementTemplate(agreement);
    agreement.DocumentURL = agreementDoc;
    
    // 3. Store agreement
    storeAgreement(agreement);
    
    // 4. Log activity
    logActivity('AGREEMENT_CREATED', tokenId, agreement.AgreementID);
    
    return agreement;
    
  } catch (error) {
    Logger.log('Error creating agreement: ' + error);
    throw error;
  }
}
```

## Signature Collection

```javascript
function recordSignature(agreementId, signerType, signatureData) {
  try {
    const agreement = getAgreement(agreementId);
    
    // 1. Validate signer type
    if (!['BUYER', 'SELLER', 'WITNESS'].includes(signerType)) {
      throw new Error('Invalid signer type');
    }
    
    // 2. Store signature
    if (!agreement.Signatures) {
      agreement.Signatures = {};
    }
    
    agreement.Signatures[signerType] = {
      SignatureImage: signatureData.image, // Base64 or image URL
      SignatureDate: new Date(),
      IPAddress: signatureData.ipAddress,
      DeviceInfo: signatureData.deviceInfo
    };
    
    // 3. Check if all signatures collected
    const allSigned = agreement.Signatures.BUYER && 
                      agreement.Signatures.SELLER;
    
    if (allSigned) {
      agreement.Status = 'Signed';
      
      // Update lead status
      updateLeadStatus(agreement.LeadID, 'AGREEMENT_SIGNED');
      
      // Schedule registry submission
      scheduleRegistrySubmission(agreementId);
    }
    
    // 4. Store updated agreement
    updateAgreement(agreement);
    
    // 5. Log activity
    logActivity('SIGNATURE_RECORDED', agreementId, signerType);
    
    return agreement;
    
  } catch (error) {
    Logger.log('Error recording signature: ' + error);
    throw error;
  }
}
```

## Deal Confirmation

```javascript
function confirmDeal(agreementId) {
  try {
    const agreement = getAgreement(agreementId);
    
    // 1. Verify all signatures
    if (!agreement.Signatures.BUYER || !agreement.Signatures.SELLER) {
      throw new Error('All signatures required');
    }
    
    // 2. Generate final agreement PDF
    const finalPDF = generateFinalAgreementPDF(agreement);
    agreement.FinalAgreementURL = uploadToDrive(
      finalPDF, 
      'FinalAgreements'
    );
    
    // 3. Update status
    agreement.Status = 'Confirmed';
    agreement.ConfirmationDate = new Date();
    
    // 4. Calculate final commission
    const commission = calculateCommission(agreement);
    storeCommissionRecord(commission);
    
    // 5. Update lead status
    updateLeadStatus(agreement.LeadID, 'DEAL_CONFIRMED');
    
    // 6. Update property status
    updatePropertyStatus(agreement.PropertyID, 'Sold');
    
    // 7. Create activity timeline
    createDealClosureTimeline(agreement);
    
    // 8. Store updated agreement
    updateAgreement(agreement);
    
    // 9. Send confirmations
    sendDealConfirmationNotifications(agreement);
    
    // 10. Log activity
    logActivity('DEAL_CONFIRMED', agreement.AgreementID, 
      agreement.FinalAmount);
    
    return agreement;
    
  } catch (error) {
    Logger.log('Error confirming deal: ' + error);
    sendAdminAlert('DEAL_CONFIRMATION_FAILED', error.message);
    throw error;
  }
}
```

## Token & Deal Statistics

```javascript
function getTokenStatistics() {
  const allTokens = getAllTokenReceipts();
  
  return {
    totalTokensReceived: allTokens.length,
    totalTokenValue: allTokens.reduce((sum, t) => sum + t.TokenAmount, 0),
    averageTokenAmount: calculateAverage(
      allTokens.map(t => t.TokenAmount)
    ),
    tokenByPaymentMethod: groupBy(allTokens, 'PaymentMethod'),
    averageTokenPercentage: calculateAverage(
      allTokens.map(t => t.TokenPercentage)
    ),
    pendingAgreements: getAgreementsByStatus('Draft').length,
    signedAgreements: getAgreementsByStatus('Signed').length,
    confirmedDeals: getAgreementsByStatus('Confirmed').length
  };
}
```

---

Next: See [17_COMMISSION_SYSTEM.md](17_COMMISSION_SYSTEM.md) for commission calculations.
