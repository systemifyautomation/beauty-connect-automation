# Make.com Scenario Setup Guide

## Overview
This guide provides step-by-step instructions for importing and configuring the Make.com scenario that automates the flow of approved leads from Shopify to GoHighLevel and initiates WhatsApp conversations.

## Prerequisites

Before starting, ensure you have:
- [ ] Active Make.com account (Free or paid plan)
- [ ] Shopify store admin access
- [ ] GoHighLevel account with API access
- [ ] GoHighLevel WhatsApp integration configured
- [ ] Required API keys and credentials

## File Description

### `make-scenario.json`
This is the Make.com scenario blueprint that contains:
- **Module 1**: Custom Webhook (Shopify trigger)
- **Module 2**: Router (for flow control)
- **Module 3**: Set Variables (data transformation)
- **Module 4**: HTTP Request (create GoHighLevel contact)
- **Module 5**: Set Variables (extract contact ID)
- **Module 6**: HTTP Request (send WhatsApp message)
- **Module 7**: Error Handler (logging failed executions)

## Import Instructions

### Step 1: Import the Scenario to Make.com

1. **Login to Make.com**
   - Navigate to https://www.make.com/
   - Sign in to your account

2. **Create New Scenario**
   - Click on "Scenarios" in the left menu
   - Click "Create a new scenario" button

3. **Import Blueprint**
   - Click on the three dots menu (⋮) in the bottom left
   - Select "Import Blueprint"
   - Choose the `make-scenario.json` file from this repository
   - Click "Import"

4. **Scenario Imported**
   - The scenario will appear with all modules configured
   - Note: Some connections and API keys will need to be configured

### Step 2: Configure Shopify Webhook

1. **Generate Webhook URL**
   - In Make.com, click on the Webhook module (Module 1)
   - Click "Add" to create a new webhook
   - Name it "Shopify Lead Approval Webhook"
   - Click "Save"
   - Copy the generated webhook URL (you'll need this for Shopify)

2. **Create Shopify Webhook**
   - Log in to your Shopify Admin
   - Go to **Settings** > **Notifications** > **Webhooks**
   - Click "Create webhook"
   - Configure:
     - **Event**: Customer creation or Customer update
     - **Format**: JSON
     - **URL**: Paste the Make.com webhook URL
     - **API version**: Latest stable version
   - Click "Save webhook"

3. **Alternative: Tag-Based Trigger**
   If you want to trigger only when a specific tag is added:
   - Create webhook for "Customer update" event
   - Add a filter in Make.com to check for specific tag (e.g., "lead-approved")
   - This provides more control over which customers trigger the automation

### Step 3: Configure GoHighLevel Integration

1. **Get GoHighLevel API Key**
   - Log in to your GoHighLevel account
   - Navigate to **Settings** > **Integrations** > **API Keys**
   - Click "Create API Key"
   - Name it "Make.com Shopify Integration"
   - Select required scopes:
     - ✅ contacts.write
     - ✅ contacts.read
     - ✅ conversations.write
     - ✅ conversations.messages.write
   - Copy the generated API key

2. **Configure Custom Fields in GoHighLevel**
   - Go to **Settings** > **Custom Fields**
   - Create the following custom fields:
     - `shopify_customer_id` (Text)
     - `shopify_tags` (Text)
     - `total_spent` (Number)
     - `orders_count` (Number)
     - `approval_date` (Date)
     - `lead_source` (Text)

3. **Update Make.com Scenario**
   - In Make.com, click on Module 4 (HTTP: Create GHL Contact)
   - Find the header: `Authorization: Bearer {{YOUR_GOHIGHLEVEL_API_KEY}}`
   - Replace `{{YOUR_GOHIGHLEVEL_API_KEY}}` with your actual API key
   - Click "OK" to save

   - Click on Module 6 (HTTP: Send WhatsApp Message)
   - Find the header: `Authorization: Bearer {{YOUR_GOHIGHLEVEL_API_KEY}}`
   - Replace `{{YOUR_GOHIGHLEVEL_API_KEY}}` with your actual API key
   - Click "OK" to save

### Step 4: Configure WhatsApp Integration

1. **WhatsApp Business Setup in GoHighLevel**
   - In GoHighLevel, go to **Settings** > **Integrations** > **WhatsApp**
   - Connect your WhatsApp Business account
   - Verify your phone number
   - Submit message templates for approval (if required)

2. **Customize WhatsApp Message**
   - In Make.com, click on Module 6 (Send WhatsApp Message)
   - Edit the message template in the `message` field:
   ```
   Hi {{3.firstName}}! 👋

   Thank you for your interest in Beauty Connect! We're excited to help you discover amazing beauty products.

   Our team has reviewed your profile and we'd love to chat with you about personalized recommendations. When would be a good time for a quick conversation?

   Looking forward to connecting with you! 💄✨
   ```
   - Customize this message to match your brand voice
   - Ensure it complies with WhatsApp Business Policy
   - Click "OK" to save

### Step 5: Testing the Scenario

1. **Enable Scenario**
   - Toggle the scenario to "ON" (switch in bottom left)
   - The webhook is now active and listening

2. **Test with Shopify**
   - **Option A: Create Test Customer**
     - In Shopify Admin, create a new customer
     - Add the "lead-approved" tag (if using tag-based trigger)
     - Save the customer

   - **Option B: Update Existing Customer**
     - Edit an existing customer
     - Add required tags
     - Save changes

3. **Verify Execution**
   - In Make.com, check the scenario execution history
   - Click on the execution to see detailed logs
   - Verify each module executed successfully

4. **Check Results**
   - **GoHighLevel**: Verify the contact was created/updated
     - Check if all custom fields are populated
     - Verify tags are applied correctly
   - **WhatsApp**: Confirm the message was sent
     - Check the conversation in GoHighLevel
     - Verify the message content is correct

### Step 6: Error Handling Configuration

1. **Review Error Handler**
   - Module 7 logs errors when contact creation fails
   - Currently stores error in a variable

2. **Optional: Add Error Notifications**
   To receive error notifications, add these modules after Module 7:
   
   **Option A: Email Notification**
   - Add "Email" module
   - Configure to send error details to your team
   
   **Option B: Google Sheets Logging**
   - Add "Google Sheets" module
   - Create a new row with error details
   
   **Option C: Slack Notification**
   - Add "Slack" module
   - Post error message to a monitoring channel

## Configuration Checklist

Before going live, verify:

- [ ] Webhook URL is correctly configured in Shopify
- [ ] GoHighLevel API key is added to both HTTP modules
- [ ] Custom fields are created in GoHighLevel
- [ ] WhatsApp Business account is connected
- [ ] WhatsApp message template is approved (if required)
- [ ] Test execution completed successfully
- [ ] Contact appears in GoHighLevel
- [ ] WhatsApp message was delivered
- [ ] Error handling is configured
- [ ] Scenario is enabled (ON)

## Advanced Configuration

### Adding Conditional Logic

**Filter by Customer Tags**
1. Add a "Filter" after Module 1
2. Condition: `Tags contains "lead-approved"`
3. Only matching customers will proceed

**Filter by Order Value**
1. Add a "Filter" after Module 3
2. Condition: `Total Spent > 50`
3. Only high-value customers proceed

### Rate Limiting

If processing high volumes:
1. Add "Sleep" module between operations
2. Configure delay (e.g., 1-2 seconds)
3. Prevents API rate limit errors

### Multiple WhatsApp Templates

To send different messages based on customer data:
1. Add a "Router" after Module 5
2. Create multiple routes with filters
3. Send different messages per route

### Adding SMS Fallback

If WhatsApp fails:
1. Add error handler to Module 6
2. Add "HTTP" or "SMS" module
3. Send SMS if WhatsApp fails

## Monitoring & Maintenance

### Daily Monitoring
- Check Make.com execution history
- Review success/failure rates
- Monitor GoHighLevel contact creation

### Weekly Reviews
- Analyze message response rates
- Review error logs
- Optimize message templates

### Monthly Audits
- Verify data mapping accuracy
- Update API keys if rotated
- Review and update documentation

## Troubleshooting

### Issue: Webhook Not Triggering

**Solution:**
1. Verify webhook is active in Shopify
2. Check webhook URL is correct
3. Test webhook using Shopify's "Send test notification"
4. Review Shopify webhook logs for errors

### Issue: Contact Not Created in GoHighLevel

**Solution:**
1. Verify API key has correct permissions
2. Check required fields are present in payload
3. Review GoHighLevel API error response
4. Ensure phone number is in correct format

### Issue: WhatsApp Message Not Sending

**Solution:**
1. Verify WhatsApp Business account is connected
2. Check message template compliance
3. Ensure contact phone number is valid
4. Verify 24-hour messaging window or use approved template

### Issue: Duplicate Contacts

**Solution:**
1. Add filter to check if contact exists
2. Use GoHighLevel's update endpoint instead of create
3. Add conditional logic based on email/phone lookup

## API Documentation References

- **Make.com**: https://www.make.com/en/help/modules
- **Shopify Webhooks**: https://shopify.dev/docs/api/admin-rest/webhooks
- **GoHighLevel API**: https://highlevel.stoplight.io/docs/integrations/
- **WhatsApp Business API**: https://developers.facebook.com/docs/whatsapp/

## Support

For issues or questions:
- **Systemify Automation**: support@systemifyautomation.com
- **Make.com Support**: https://www.make.com/en/help/support
- **GoHighLevel Support**: https://help.gohighlevel.com/

## Version History

- **v1.0** (2024): Initial scenario release
  - Shopify webhook integration
  - GoHighLevel contact creation
  - WhatsApp message automation
  - Basic error handling

## Next Steps

After successful setup:
1. Monitor the first few executions closely
2. Gather feedback on message templates
3. Optimize based on response rates
4. Consider implementing AI agent for responses
5. Add advanced segmentation and routing

## License

This scenario is provided for use by Beauty Connect and its automation partners (Systemify Automation and One Umbrella).
