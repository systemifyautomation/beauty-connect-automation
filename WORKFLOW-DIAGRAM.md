# Visual Workflow Diagram

## Automation Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SHOPIFY STORE                                │
│                                                                     │
│  Customer approved / Tagged as "lead-approved"                     │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             │ Webhook Trigger
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      MAKE.COM SCENARIO                              │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Module 1: Custom Webhook                                     │  │
│  │ - Receives Shopify webhook payload                           │  │
│  │ - Captures customer data (name, email, phone, tags, etc.)    │  │
│  └──────────────────────┬───────────────────────────────────────┘  │
│                         │                                           │
│                         ▼                                           │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Module 2: Router                                             │  │
│  │ - Directs flow to main process and error handling            │  │
│  └──────────┬───────────────────────────────────┬────────────────┘  │
│             │                                   │                   │
│             │ Main Flow                         │ Error Path        │
│             ▼                                   ▼                   │
│  ┌──────────────────────┐            ┌─────────────────────────┐   │
│  │ Module 3:            │            │ Module 7:               │   │
│  │ Set Variables        │            │ Error Handler           │   │
│  │ - Extract & format   │            │ - Log failed execution  │   │
│  │   customer data      │            │ - Store error details   │   │
│  └──────────┬───────────┘            └─────────────────────────┘   │
│             │                                                       │
│             ▼                                                       │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Module 4: HTTP Request to GoHighLevel                        │  │
│  │ - POST /v1/contacts/                                         │  │
│  │ - Create/update contact with:                                │  │
│  │   * First name, Last name                                    │  │
│  │   * Email, Phone                                             │  │
│  │   * Tags: shopify-lead, approved, beauty-connect             │  │
│  │   * Custom fields: shopify_customer_id, total_spent, etc.    │  │
│  └──────────────────────┬───────────────────────────────────────┘  │
│                         │                                           │
│                         ▼                                           │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Module 5: Set Variables                                      │  │
│  │ - Extract GoHighLevel contact ID                             │  │
│  │ - Store for WhatsApp message                                 │  │
│  └──────────────────────┬───────────────────────────────────────┘  │
│                         │                                           │
│                         ▼                                           │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Module 6: HTTP Request to GoHighLevel                        │  │
│  │ - POST /v1/conversations/messages                            │  │
│  │ - Send WhatsApp message with:                                │  │
│  │   * Personalized greeting                                    │  │
│  │   * Beauty Connect branding                                  │  │
│  │   * Call to action                                           │  │
│  └──────────────────────┬───────────────────────────────────────┘  │
│                         │                                           │
└─────────────────────────┼───────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     GOHIGHLEVEL CRM                                 │
│                                                                     │
│  ✓ Contact created/updated                                         │
│  ✓ Tags applied (shopify-lead, approved, beauty-connect)           │
│  ✓ Custom fields populated                                         │
│  ✓ Contact ready for engagement                                    │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    WHATSAPP BUSINESS                                │
│                                                                     │
│  📱 Message delivered to customer                                   │
│  💬 Conversation initiated                                          │
│  🤖 AI agent ready to engage (optional)                             │
└─────────────────────────────────────────────────────────────────────┘
```

## Data Mapping

### Shopify → GoHighLevel

| Shopify Field      | GoHighLevel Field        | Notes                          |
|--------------------|--------------------------|--------------------------------|
| first_name         | firstName                | Direct mapping                 |
| last_name          | lastName                 | Direct mapping                 |
| email              | email                    | Direct mapping                 |
| phone              | phone                    | Formatted to E.164             |
| customer_id        | shopify_customer_id      | Custom field                   |
| tags               | shopify_tags             | Custom field (comma-separated) |
| total_spent        | total_spent              | Custom field (number)          |
| orders_count       | orders_count             | Custom field (number)          |
| -                  | approval_date            | Custom field (timestamp)       |
| -                  | lead_source              | Custom field (set to "Shopify")|
| -                  | tags                     | Add: shopify-lead, approved    |

## Module Flow Details

### 1. Webhook Trigger (Module 1)
**Purpose**: Receive and parse Shopify webhook  
**Input**: Shopify customer data (JSON)  
**Output**: Structured customer object  
**Timing**: Instant (real-time)

### 2. Router (Module 2)
**Purpose**: Direct flow and handle errors  
**Routes**:
- Route 1: Main processing path
- Route 2: Error handling path  
**Logic**: Conditional based on success/failure

### 3. Set Variables (Module 3)
**Purpose**: Transform and prepare data  
**Actions**:
- Extract relevant fields
- Format phone numbers
- Add timestamps
- Prepare for API calls  
**Timing**: < 1 second

### 4. Create Contact (Module 4)
**Purpose**: Create/update contact in GoHighLevel  
**API**: POST /v1/contacts/  
**Authentication**: Bearer token  
**Response**: Contact object with ID  
**Timing**: 2-5 seconds

### 5. Extract Contact ID (Module 5)
**Purpose**: Store contact ID for next step  
**Input**: GoHighLevel API response  
**Output**: Contact ID variable  
**Timing**: < 1 second

### 6. Send WhatsApp (Module 6)
**Purpose**: Initiate WhatsApp conversation  
**API**: POST /v1/conversations/messages  
**Message Type**: WhatsApp  
**Template**: Personalized greeting  
**Timing**: 2-5 seconds

### 7. Error Handler (Module 7)
**Purpose**: Log errors and failures  
**Trigger**: HTTP status != 200  
**Actions**:
- Log error details
- Store customer data
- Timestamp failure  
**Use Case**: Debugging and retry

## Error Handling Scenarios

### Scenario 1: Invalid Phone Number
```
Shopify → Make.com → [VALIDATE] → Error: Invalid phone format
                                → Log error
                                → Notify admin
```

### Scenario 2: GoHighLevel API Failure
```
Shopify → Make.com → [CREATE CONTACT] → Error: API timeout
                                       → Retry (up to 3x)
                                       → Log if still fails
```

### Scenario 3: WhatsApp Delivery Failure
```
Shopify → Make.com → GoHighLevel → [WHATSAPP] → Error: Outside 24hr window
                                                → Use approved template
                                                → OR schedule for later
```

## Performance Metrics

### Expected Timing
- **Webhook receipt**: < 1 second
- **Data transformation**: < 1 second
- **GoHighLevel contact creation**: 2-5 seconds
- **WhatsApp message delivery**: 2-5 seconds
- **Total end-to-end**: 5-12 seconds

### Volume Capacity
- **Make.com Free Plan**: 1,000 operations/month
- **Make.com Basic Plan**: 10,000 operations/month
- **Recommended**: Monitor usage and upgrade as needed

### Success Rate Target
- **Goal**: > 95% successful executions
- **Monitoring**: Daily execution logs
- **Alerts**: Set up for failure rates > 5%

## Integration Points

### External Systems
1. **Shopify** → Webhook trigger
2. **Make.com** → Orchestration engine
3. **GoHighLevel** → CRM and WhatsApp integration
4. **WhatsApp Business** → Message delivery

### API Dependencies
- Shopify Webhooks API (outbound)
- GoHighLevel Contacts API (inbound)
- GoHighLevel Conversations API (inbound)
- WhatsApp Business API (via GoHighLevel)

### Authentication
- **Shopify**: Webhook signature verification
- **GoHighLevel**: Bearer token (API key)
- **WhatsApp**: Authenticated via GoHighLevel

## Next Steps After Deployment

1. **Monitor First 24 Hours**
   - Watch execution logs closely
   - Verify all contacts are created
   - Check WhatsApp delivery rates

2. **Optimize Based on Data**
   - Analyze response patterns
   - Refine message templates
   - Adjust error handling

3. **Scale Gradually**
   - Start with small volume
   - Increase as confidence grows
   - Monitor API limits

4. **Enhance Features**
   - Add AI agent responses
   - Implement lead scoring
   - Create follow-up sequences

## Support Resources

- **Make.com Documentation**: https://www.make.com/en/help
- **Shopify Webhooks**: https://shopify.dev/docs/api/admin-rest/webhooks
- **GoHighLevel API**: https://highlevel.stoplight.io/docs/integrations/
- **WhatsApp Business**: https://developers.facebook.com/docs/whatsapp/

## Version Control

This diagram represents version 1.0 of the automation workflow. Updates will be documented in the git commit history.
