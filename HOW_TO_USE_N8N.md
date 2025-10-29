# Getting Started with n8n on DigitalOcean App Platform

Welcome to your n8n deployment! This guide will help you create your first workflow and unlock the power of automation.

## 📋 Table of Contents

- [First-Time Setup](#first-time-setup)
- [Creating Your First Workflow](#creating-your-first-workflow)
- [Example Workflows](#example-workflows)
- [Connecting External Services](#connecting-external-services)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## 🚀 First-Time Setup

### 1. Access Your n8n Instance

After deployment, access your n8n instance at:
```
https://your-app-name.ondigitalocean.app
```

### 2. Create Your Owner Account

On first visit, you'll be prompted to create an owner account:

1. **Enter your email** and choose a strong password
2. **Set up your profile** (name, company - optional)
3. **Click "Get Started"**

> **Important**: The first user becomes the **owner** with full administrative rights.

### 3. Explore the Interface

- **Workflows**: List of all your workflows
- **Credentials**: Stored authentication for external services
- **Executions**: History of workflow runs
- **Settings**: User and instance configuration

## 🎯 Creating Your First Workflow

Let's create a simple workflow that demonstrates n8n's power.

### Example 1: Daily Weather Email

This workflow fetches weather data and sends you an email every morning.

**Steps:**

1. **Create a New Workflow**
   - Click "+ New Workflow" button
   - Name it "Daily Weather Report"

2. **Add a Schedule Trigger**
   - Click the "+" button
   - Search for "Schedule Trigger"
   - Set to run daily at 8:00 AM

3. **Fetch Weather Data**
   - Click "+" after the trigger
   - Search for "HTTP Request"
   - Configure:
     ```
     Method: GET
     URL: https://wttr.in/Amsterdam?format=j1
     ```

4. **Format the Message**
   - Add "Code" node
   - Use JavaScript to format weather data:
     ```javascript
     const weather = $json.current_condition[0];
     return {
       subject: `Weather for ${new Date().toLocaleDateString()}`,
       body: `
         Temperature: ${weather.temp_C}°C
         Condition: ${weather.weatherDesc[0].value}
         Humidity: ${weather.humidity}%
       `
     };
     ```

5. **Send Email** (requires SMTP credentials)
   - Add "Send Email" node
   - Configure your SMTP settings
   - Use the formatted message

6. **Test & Activate**
   - Click "Execute Workflow" to test
   - Click the toggle to "Active"

## 💡 Example Workflows

### Example 2: Webhook-Triggered Slack Notification

Perfect for integrating with other services.

```mermaid
graph LR
    A[Webhook Trigger] --> B[Set Variables]
    B --> C[Send to Slack]
```

**Use Case**: Post to Slack when your app receives a form submission.

**Setup**:
1. Add "Webhook" trigger node
2. Copy the webhook URL
3. Add "Slack" node
4. Configure Slack credentials
5. Test with:
   ```bash
   curl -X POST https://your-app.ondigitalocean.app/webhook/your-path \
     -H "Content-Type: application/json" \
     -d '{"message": "Test notification"}'
   ```

### Example 3: Database Backup to Spaces

Automate PostgreSQL backups to DigitalOcean Spaces.

```mermaid
graph LR
    A[Schedule: Daily] --> B[Postgres Node]
    B --> C[Compress Data]
    C --> D[Upload to Spaces]
    D --> E[Send Confirmation Email]
```

**Setup**:
1. Schedule Trigger (daily at 2 AM)
2. Postgres node: `SELECT * FROM your_table`
3. Code node: Convert to JSON
4. S3 node: Upload to Spaces (use DigitalOcean Spaces endpoint)
5. Email node: Send confirmation

### Example 4: API Integration Workflow

Connect multiple APIs to automate data flow.

**Scenario**: When a new customer signs up, create records in:
- CRM (Pipedrive/HubSpot)
- Project Management (Asana/Monday.com)
- Communication (Slack/Discord)

**Pattern**:
```
Webhook → Validate Data → Split into Parallel Branches → Update Multiple Services
```

## 🔐 Connecting External Services

### Adding Credentials

1. Go to **Credentials** in the left menu
2. Click **"+ New Credential"**
3. Select your service (Slack, Google, etc.)
4. Follow the OAuth flow or enter API keys
5. Test the connection
6. Save

### Common Integrations

#### Slack
- **Type**: OAuth2
- **Use**: Notifications, bot interactions
- **Setup**: Create Slack App → Get OAuth token

#### Google Sheets
- **Type**: OAuth2
- **Use**: Read/write spreadsheet data
- **Setup**: Google Cloud Console → Enable Sheets API

#### HTTP Request (Generic API)
- **Type**: Header Auth / Basic Auth
- **Use**: Any REST API
- **Setup**: API key from service

#### PostgreSQL (Your Database)
- **Type**: Credentials
- **Use**: Query your app's database
- **Setup**: Use database connection details from App Platform

### Webhook URLs

Your webhook URLs follow this pattern:
```
https://your-app.ondigitalocean.app/webhook/your-custom-path
```

Or production webhooks:
```
https://your-app.ondigitalocean.app/webhook-test/your-custom-path
```

## ✨ Best Practices

### 1. Workflow Organization
- **Use descriptive names**: "Customer Onboarding" not "Workflow 1"
- **Add notes**: Document complex logic
- **Use folders/tags**: Group related workflows (available in settings)

### 2. Error Handling
- Add "Error Trigger" nodes to catch failures
- Set up email notifications for errors
- Use Try/Catch pattern with "IF" nodes

Example:
```mermaid
graph LR
    A[Trigger] --> B[Try: Main Logic]
    B -->|Success| C[Continue]
    B -->|Error| D[Error Handler]
    D --> E[Send Alert]
```

### 3. Credential Security
- **Never hardcode** API keys in workflows
- **Use Credentials** for all sensitive data
- **Rotate keys** regularly
- **Limit permissions** (principle of least privilege)

### 4. Performance
- **Batch operations** when possible
- **Use pagination** for large datasets
- **Add delays** to respect rate limits
- **Monitor execution time** in Executions tab

### 5. Testing
- Use "Execute Node" for step-by-step testing
- Test with sample data before going live
- Keep test workflows separate from production

## 🛠️ Troubleshooting

### Workflow Not Triggering

**Problem**: Scheduled workflow doesn't run

**Solutions**:
- Ensure workflow is **Active** (toggle on)
- Check timezone settings (`GENERIC_TIMEZONE`)
- Review execution history for errors

### Webhook Not Receiving Data

**Problem**: Webhook returns 404 or doesn't execute

**Solutions**:
- Verify webhook URL is correct
- Check `WEBHOOK_URL` environment variable
- Ensure workflow is active
- Test with curl:
  ```bash
  curl -X POST https://your-app.ondigitalocean.app/webhook/test \
    -H "Content-Type: application/json" \
    -d '{"test": "data"}'
  ```

### Credential Authentication Failed

**Problem**: API connections fail

**Solutions**:
- Re-authenticate OAuth connections
- Verify API keys haven't expired
- Check service status (third-party downtime)
- Review scopes/permissions

### Database Connection Issues

**Problem**: Can't connect to PostgreSQL

**Solutions**:
- Verify `DB_POSTGRESDB_*` environment variables
- DB_POSTGRESDB_SSL_ENABLED=true
  DB_POSTGRESDB_SSL_REJECT_UNAUTHORIZED=false
- Check database firewall (trusted sources)
- Ensure database is running
- Test connection with:
  ```bash
  psql "postgresql://user:pass@host:port/db?sslmode=require"
  ```

### Workflow Execution Timeout

**Problem**: Long-running workflows fail

**Solutions**:
- Split into smaller workflows
- Use "Wait" node for asynchronous operations
- Optimize database queries
- Increase timeout in workflow settings

## 📚 Advanced Features

### 1. Sub-Workflows
Create reusable workflows that can be called from other workflows:
- Create a workflow with "Execute Workflow" trigger
- Call it from other workflows with "Execute Workflow" node
- Perfect for common operations (data validation, notifications)

### 2. Code Nodes
Write custom JavaScript or Python:
```javascript
// Access input data
const items = $input.all();

// Process data
const result = items.map(item => ({
  ...item.json,
  processed: true,
  timestamp: new Date().toISOString()
}));

// Return results
return result;
```

### 3. Expressions
Use expressions to reference data:
```
{{ $json.customer.email }}
{{ $now.format('YYYY-MM-DD') }}
{{ $workflow.id }}
```

### 4. Binary Data
Handle files (images, PDFs, etc.):
- Read from HTTP/S3/local storage
- Transform (resize images, convert formats)
- Upload to external services

## 🎓 Learning Resources

- **Official Docs**: [docs.n8n.io](https://docs.n8n.io)
- **Workflow Templates**: [n8n.io/workflows](https://n8n.io/workflows) (900+ templates)
- **Community Forum**: [community.n8n.io](https://community.n8n.io)
- **YouTube Tutorials**: [n8n YouTube Channel](https://www.youtube.com/c/n8n-io)
- **Discord Community**: [n8n Discord](https://discord.gg/n8n)

## 🚀 Next Steps

1. **Explore Templates**: Import pre-built workflows
2. **Connect Your Apps**: Add credentials for services you use
3. **Automate Tasks**: Identify repetitive processes to automate
4. **Join Community**: Share and learn from other users
5. **Go Pro**: Consider n8n Cloud for advanced features

## 💬 Need Help?

- **Documentation**: [docs.n8n.io](https://docs.n8n.io)
- **Community Forum**: [community.n8n.io](https://community.n8n.io)
- **GitHub Issues**: [github.com/n8n-io/n8n/issues](https://github.com/n8n-io/n8n/issues)
- **DigitalOcean Support**: For infrastructure issues

---

**Happy Automating!** 🎉

Start small, experiment often, and gradually build more complex workflows. The n8n community is here to help you succeed!
