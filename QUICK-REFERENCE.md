# Quick Reference Guide

## Essential Configuration Values

### API Endpoints

**GoHighLevel API Base URL**: `https://rest.gohighlevel.com/v1`

**Endpoints Used**:
- Create Contact: `POST /contacts/`
- Send Message: `POST /conversations/messages`

### Required API Scopes

GoHighLevel API Key must have:
- ✅ `contacts.read`
- ✅ `contacts.write`
- ✅ `conversations.write`
- ✅ `conversations.messages.write`

### Custom Fields in GoHighLevel

Create these custom fields before importing the scenario:

| Field Name              | Type   | Description                    |
|-------------------------|--------|--------------------------------|
| `shopify_customer_id`   | Text   | Shopify customer ID            |
| `shopify_tags`          | Text   | Customer tags from Shopify     |
| `total_spent`           | Number | Total amount spent by customer |
| `orders_count`          | Number | Number of orders placed        |
| `approval_date`         | Date   | Date lead was approved         |
| `lead_source`           | Text   | Always set to "Shopify"        |

### Tags Applied to Contacts

The automation applies these tags in GoHighLevel:
- `shopify-lead`
- `approved`
- `beauty-connect`

### Webhook Payload Structure

Expected Shopify webhook data:

```json
{
  "customer_id": "123456789",
  "first_name": "Jane",
  "last_name": "Doe",
  "email": "jane.doe@example.com",
  "phone": "+1234567890",
  "tags": "lead-approved, vip-customer",
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z",
  "total_spent": "150.00",
  "orders_count": 3
}
```

## Make.com Scenario Modules

### Module 1: Custom Webhook
- **Type**: Trigger
- **Name**: Shopify Lead Approval Webhook
- **Configuration**: Copy webhook URL to Shopify

### Module 2: Router
- **Type**: Flow Control
- **Routes**: 2 (main process + error handler)

### Module 3: Set Variables
- **Type**: Tools
- **Variables**: 9 (customer data fields)
- **Scope**: One cycle (roundtrip)

### Module 4: HTTP Request (Create Contact)
- **Method**: POST
- **URL**: `https://rest.gohighlevel.com/v1/contacts/`
- **Headers**:
  - `Authorization: Bearer YOUR_API_KEY`
  - `Content-Type: application/json`
- **Body**: JSON with contact data

### Module 5: Set Variables
- **Type**: Tools
- **Variables**: 2 (contact ID and phone)
- **Scope**: One cycle (roundtrip)

### Module 6: HTTP Request (Send WhatsApp)
- **Method**: POST
- **URL**: `https://rest.gohighlevel.com/v1/conversations/messages`
- **Headers**:
  - `Authorization: Bearer YOUR_API_KEY`
  - `Content-Type: application/json`
- **Body**: JSON with message data

### Module 7: Error Handler
- **Type**: Tools (Set Variable)
- **Trigger**: Status code != 200
- **Purpose**: Log errors for troubleshooting

## Configuration Checklist

### Before Import
- [ ] Make.com account created
- [ ] GoHighLevel API key generated
- [ ] GoHighLevel custom fields created
- [ ] WhatsApp Business connected to GoHighLevel

### During Import
- [ ] Scenario imported from JSON file
- [ ] Webhook created and URL copied
- [ ] API keys replaced in modules 4 & 6
- [ ] WhatsApp message customized

### After Import
- [ ] Shopify webhook configured with Make.com URL
- [ ] Test execution completed successfully
- [ ] Contact verified in GoHighLevel
- [ ] WhatsApp message confirmed delivered
- [ ] Scenario enabled (turned ON)

## Testing Commands

### Test Webhook Reception
Use Make.com's webhook tester or send this cURL command:

```bash
curl -X POST https://hook.us1.make.com/YOUR_WEBHOOK_ID \
  -H "Content-Type: application/json" \
  -d '{
    "customer_id": "test123",
    "first_name": "Test",
    "last_name": "User",
    "email": "test@example.com",
    "phone": "+1234567890",
    "tags": "lead-approved",
    "created_at": "2024-01-15T10:00:00Z",
    "updated_at": "2024-01-15T10:00:00Z",
    "total_spent": "0.00",
    "orders_count": 0
  }'
```

### Test GoHighLevel Contact Creation
Use GoHighLevel API directly:

```bash
curl -X POST https://rest.gohighlevel.com/v1/contacts/ \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "firstName": "Test",
    "lastName": "User",
    "email": "test@example.com",
    "phone": "+1234567890",
    "tags": ["shopify-lead", "approved"],
    "source": "Shopify Automation"
  }'
```

## Troubleshooting Quick Fixes

### Webhook Not Receiving Data
```bash
# Check webhook in Make.com
1. Click on webhook module
2. View execution history
3. Check "Received data" tab
4. Verify JSON structure matches expected format
```

### API Key Invalid
```bash
# Verify API key in GoHighLevel
1. Go to Settings > Integrations > API Keys
2. Check key is active
3. Verify scopes are correct
4. Regenerate if necessary
5. Update in Make.com modules 4 & 6
```

### Phone Number Format Error
```javascript
// Phone must be in E.164 format
// Correct: +1234567890
// Incorrect: (123) 456-7890
// Fix in Module 3 with formatPhone() function
```

### WhatsApp Message Not Sending
```bash
# Check WhatsApp connection
1. GoHighLevel > Settings > Integrations > WhatsApp
2. Verify connection is active
3. Check phone number is verified
4. Ensure message template is approved
```

## Common Variables Used

### Shopify Data (Module 1 Output)
- `{{1.customer_id}}` - Shopify customer ID
- `{{1.first_name}}` - Customer first name
- `{{1.last_name}}` - Customer last name
- `{{1.email}}` - Customer email
- `{{1.phone}}` - Customer phone number
- `{{1.tags}}` - Customer tags (comma-separated)
- `{{1.total_spent}}` - Total customer spend
- `{{1.orders_count}}` - Number of orders

### Processed Variables (Module 3 Output)
- `{{3.firstName}}` - Cleaned first name
- `{{3.lastName}}` - Cleaned last name
- `{{3.email}}` - Validated email
- `{{3.phone}}` - Formatted phone number
- `{{3.shopifyCustomerId}}` - Shopify ID
- `{{3.shopifyTags}}` - Tags string
- `{{3.totalSpent}}` - Spend amount
- `{{3.ordersCount}}` - Order count
- `{{3.approvalDate}}` - Timestamp of approval

### GoHighLevel Response (Module 4 Output)
- `{{4.data.contact.id}}` - GoHighLevel contact ID
- `{{4.data.contact.phone}}` - Contact phone in GHL
- `{{4.statusCode}}` - HTTP response status

### Contact Data (Module 5 Output)
- `{{5.ghlContactId}}` - Stored contact ID
- `{{5.ghlContactPhone}}` - Stored phone number

## Message Template Variables

Default WhatsApp message uses:
- `{{3.firstName}}` - Personalizes greeting
- Static text for Beauty Connect branding
- Call-to-action for conversation

Customize in Module 6:
```
Hi {{3.firstName}}! 👋

Thank you for your interest in Beauty Connect! We're excited to help you discover amazing beauty products.

Our team has reviewed your profile and we'd love to chat with you about personalized recommendations. When would be a good time for a quick conversation?

Looking forward to connecting with you! 💄✨
```

## Performance Optimization

### Reduce Execution Time
1. Enable auto-commit in scenario settings
2. Use batch operations if available
3. Minimize unnecessary data transformations
4. Cache frequently accessed data

### Handle High Volume
1. Add sleep/delay between operations if hitting rate limits
2. Use Make.com's queue feature
3. Consider upgrading Make.com plan
4. Monitor operation usage daily

### Error Recovery
1. Enable automatic retries (3 attempts)
2. Add exponential backoff
3. Log all errors to external system
4. Set up email alerts for critical failures

## Maintenance Schedule

### Daily
- [ ] Check scenario execution history
- [ ] Review error logs
- [ ] Verify contact creation rate

### Weekly
- [ ] Analyze execution patterns
- [ ] Review WhatsApp response rates
- [ ] Update message templates if needed

### Monthly
- [ ] Audit data mapping accuracy
- [ ] Review and optimize performance
- [ ] Update API keys (if rotated)
- [ ] Test disaster recovery

## Support Contacts

- **Make.com Support**: https://www.make.com/en/help/support
- **GoHighLevel Help**: https://help.gohighlevel.com/
- **Shopify Support**: https://help.shopify.com/
- **Systemify Automation**: support@systemifyautomation.com

## Useful Links

- [Make.com Documentation](https://www.make.com/en/help/modules)
- [Shopify Webhooks Guide](https://shopify.dev/docs/api/admin-rest/webhooks)
- [GoHighLevel API Docs](https://highlevel.stoplight.io/docs/integrations/)
- [WhatsApp Business Policy](https://www.whatsapp.com/legal/business-policy)

## Version Info

- **Scenario Version**: 1.0
- **Last Updated**: 2024
- **Compatibility**: Make.com v2, GoHighLevel API v1, Shopify Admin API 2024-01

---

For detailed setup instructions, see [SETUP-GUIDE.md](./SETUP-GUIDE.md)  
For complete strategy, see [STRATEGY.md](./STRATEGY.md)  
For visual workflow, see [WORKFLOW-DIAGRAM.md](./WORKFLOW-DIAGRAM.md)
