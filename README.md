# Azure Logic App - Cost Monitor Workflow

This Azure Logic App workflow monitors the previous day's Azure costs and sends an email alert when the cost exceeds $300.

## Overview

The workflow runs daily at 9:00 AM (China Standard Time) and performs the following actions:

1. **Get Access Token**: Authenticates with Azure AD using service principal credentials
2. **Get Cost Data**: Retrieves the previous day's cost data from Azure Cost Management API
3. **Parse Cost Response**: Extracts cost information from the API response
4. **Extract Total Cost**: Calculates the total cost for the previous day
5. **Check Cost Threshold**: Compares the cost against the $300 threshold
6. **Send Email Alert**: If cost exceeds $300, sends an email notification via Office 365

## Prerequisites

Before deploying this Logic App, ensure you have:

1. **Azure Subscription**: An active Azure subscription with Cost Management enabled
2. **Azure AD Service Principal**: A service principal with the following permissions:
   - `CostManagementReader` role on the subscription (or appropriate cost management permissions)
   - Application registration in Azure AD
3. **Office 365 Connection**: An Office 365 connector configured in your Logic App
4. **Logic App Resource**: An Azure Logic App resource created in your subscription

## Configuration

### Required Parameters

Configure the following parameters in the Logic App:

1. **subscriptionId** (String)
   - Your Azure subscription ID
   - Example: `12345678-1234-1234-1234-123456789012`

2. **tenantId** (String)
   - Your Azure AD tenant ID
   - Example: `87654321-4321-4321-4321-210987654321`

3. **clientId** (String)
   - Service principal application (client) ID
   - Found in Azure AD > App registrations > Your app > Overview

4. **clientSecret** (String)
   - Service principal client secret
   - Create a new secret in Azure AD > App registrations > Your app > Certificates & secrets

5. **emailRecipient** (String)
   - Email address to receive cost alerts
   - Example: `admin@example.com`

6. **$connections** (Object)
   - Office 365 connection configuration
   - Configure this through the Logic App designer UI

### Setting Up Service Principal

1. Go to Azure Portal > Azure Active Directory > App registrations
2. Click "New registration"
3. Enter a name and click "Register"
4. Note the **Application (client) ID** and **Directory (tenant) ID**
5. Go to "Certificates & secrets" > "New client secret"
6. Create a secret and **copy the value immediately** (it won't be shown again)
7. Go to your subscription > Access control (IAM)
8. Click "Add role assignment"
9. Select role: **Cost Management Reader**
10. Assign to your service principal

## Deployment

### Option 1: Import via Azure Portal

1. Open Azure Portal and navigate to your Logic App
2. Click "Logic app designer" in the left menu
3. Click "Code view" in the toolbar
4. Copy the contents of `cost-monitor-workflow.json`
5. Paste into the code view editor
6. Click "Save"
7. Configure the parameters in the Logic App settings

### Option 2: Deploy via Azure CLI

```bash
az logicapp workflow import \
  --resource-group <your-resource-group> \
  --name <your-logic-app-name> \
  --location <your-location> \
  --definition cost-monitor-workflow.json
```

### Option 3: Deploy via ARM Template

You can wrap this workflow definition in an ARM template for automated deployment.

## Configuration Steps

1. **Configure Office 365 Connection**:
   - In Logic App designer, click on the "Send_Email_Alert" action
   - If not connected, click "Add new connection"
   - Sign in with your Office 365 account
   - Authorize the connection

2. **Set Parameters**:
   - Go to Logic App > Settings > Workflow settings
   - Or configure parameters in the code view under the `parameters` section
   - Update all required parameter values

3. **Test the Workflow**:
   - Click "Run trigger" to manually test
   - Check the run history to verify each step executes successfully

## Workflow Details

### Trigger

- **Type**: Recurrence
- **Frequency**: Daily
- **Interval**: 1 day
- **Schedule**: 9:00 AM (China Standard Time)

### Cost Threshold

The default threshold is set to **$300**. To modify:

1. Open the workflow in code view
2. Find the `Check_Cost_Threshold` action
3. Change the value `300` to your desired threshold
4. Update the email body message accordingly

### Email Alert Content

The email alert includes:
- Previous day's total cost
- Threshold value ($300)
- Date of the cost period
- High importance flag

## Monitoring

Monitor your Logic App runs:

1. Go to Logic App > Overview
2. View run history and execution status
3. Click on any run to see detailed execution steps
4. Check for any errors or failures

## Troubleshooting

### Common Issues

1. **Authentication Failed**
   - Verify service principal credentials are correct
   - Ensure service principal has proper permissions
   - Check if client secret has expired

2. **Cost Data Not Retrieved**
   - Verify subscription ID is correct
   - Ensure Cost Management API is enabled for your subscription
   - Check service principal has `CostManagementReader` role

3. **Email Not Sent**
   - Verify Office 365 connection is configured
   - Check email recipient address is valid
   - Ensure Office 365 connector is authorized

4. **No Cost Data Returned**
   - Cost data may not be available immediately
   - Azure Cost Management data can have a delay of 24-48 hours
   - Consider adjusting the date range if needed

## Customization

### Change Schedule

Modify the trigger recurrence:

```json
"schedule": {
  "hours": [9],
  "minutes": [0]
},
"timeZone": "China Standard Time"
```

### Change Cost Threshold

Update the threshold value in the `Check_Cost_Threshold` action:

```json
"greater": [
  "@{variables('totalCost')}",
  300  // Change this value
]
```

### Modify Email Template

Edit the email body in the `Send_Email_Alert` action to customize the message format and content.

## Security Best Practices

1. **Store Secrets Securely**:
   - Use Azure Key Vault for storing client secrets
   - Reference secrets from Key Vault in Logic App parameters

2. **Use Managed Identity** (Recommended):
   - Consider using Logic App managed identity instead of service principal
   - Reduces need to manage client secrets

3. **Limit Permissions**:
   - Grant only necessary permissions to service principal
   - Use least privilege principle

## Support

For issues or questions:
- Check Azure Logic Apps documentation
- Review Azure Cost Management API documentation
- Contact Azure support if needed

## License

This workflow definition is provided as-is for use in Azure Logic Apps.
