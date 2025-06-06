
# Module 2 – Lab: Build Your First Logic App

## Objective
In this lab, you'll build your first Logic App using a simple integration scenario: receive an order via an HTTP POST request, validate the data, and return a response.

This is a common pattern in integration workflows — for example, receiving test data, validating inputs, or simulating backend services.

## Scenario
A test system sends an order payload (JSON) to your Logic App. You want to:
- Accept the incoming data via HTTP
- Validate the payload using a schema
- Apply basic logic to determine if the order is valid
- Respond appropriately to the sender

## Steps

### 1. Create a Resource Group
If you have not yet created/been assigned a resource group, Create that first:
- Search for "Resource Group" in the top search bar
- In the **Resource groups** view, click "Create"
- Choose a unique name within your **Subscription**
- Choose a **Region** (e.g. **Germany West Central**)
- Click "Review + create"
- Click "Create"

### 2. Create a Log Analytics Workspace
To monitor and troubleshoot your Logic App later, you'll configure diagnostic logging to send data to a Log Analytics Workspace.
- In the Azure Portal, search for **Log Analytics workspaces**
- In the **Log Analytics Workspaces** view, click **Create**
- Choose the same **Resource Group** and **Region** you’ll use for the Logic App
- Give the workspace a name (e.g., `logicapp-monitoring`)
- Click **Review + Create**, then **Create**

### 3. Create a Logic App
- Open the [Azure Portal](https://portal.azure.com)
- Search for "Logic App" in the top search bar
- Choose **Logic App** and click "Create"
- Choose **Consumption*** and click "Select"
- Select your **Resource Group** and **Region**
- Under **Monitoring**, select the Log Analytics workspace you just created
- Enter a unique name for the Logic App
- Leave default settings and click **Review + Create**

> *The Consumption plan is serverless — you only pay when your Logic App runs.*

### 4. Open the Logic App Designer
- Once deployed, go to the Logic App resource
- In the left menu, select **Logic App Designer**
- Choose the **Blank Logic App** template

This opens the drag-and-drop workflow editor.

---

### 5. Add a Trigger: HTTP Request
- In the designer, search for “HTTP” and select **When an HTTP request is received**
- Set the method to **POST**
- Click **Use sample payload to generate schema**
- Paste the provided JSON payload (from the training resources)
- Azure will auto-generate a JSON schema for input validation

> This schema ensures only correctly structured data enters your workflow.

### 6. Add a Condition Step
- Click **New Step**
- Search for **Condition**
- Use dynamic content to check one of the payload fields, such as:
  - `orderId` is not empty
  - `amount` is greater than 0
- You may use the expression editor for conditions (ask instructor if unclear)

This logic ensures only valid test orders continue through the flow.

### 7. Add a Response Step
- Inside the **If true** branch, add a **Response** action
- Set the status code to `200` (OK)
- In the body, return something like:
```json
{
  "result": "Order accepted"
}
```

- (Optional) In the **If false** branch, add another **Response** step with a `400 Bad Request` and a rejection message.

### 8. Save and Test
- Click **Save** in the Logic App Designer
- Copy the **HTTP POST URL** shown at the top of the HTTP trigger
- Use Postman, curl, or the provided test script to send a request

Verify the response and test both valid and invalid scenarios.

### 9. Enable Diagnostics (if not set during creation)
- Go to the Logic App in Azure
- Under **Monitoring**, click **Diagnostic settings**
- Click **Add diagnostic setting**
- Select:
  - **WorkflowRuntime**
  - **ActionRuntime**
  - **TriggerRuntime**
- Choose **Send to Log Analytics workspace**
- Select the workspace you created earlier
- Click **Save**

This will allow you to query execution logs in later labs.

## Additional Checks

- Go to **Identity** under your Logic App settings
- Ensure **System-assigned managed identity** is enabled  
  This will be used in later labs when connecting to other Azure services (like Service Bus and Storage) without storing credentials

## Summary
You now have a working Logic App that:
- Listens for incoming HTTP requests
- Validates and parses incoming JSON
- Responds with status and message based on logic
