# Module 5 – Lab: Monitor with Log Analytics

## Objective

Monitor the Logic App and Service Bus using Log Analytics.

## Assumptions

This lab assumes you've already created a **Log Analytics Workspace** and enabled diagnostic logging in previous labs (e.g., during OrderIntakeApp and OrderProcessorApp setup, and in the Service Bus setup).

## Steps

### 1. Open the Logs (Analytics) Window

- Go to your **Log Analytics Workspace**
- In the left-hand menu, select **Logs**
- If the **Queries Hub** opens automatically, **close it**
- In the query window, ensure the mode is set to **KQL mode** (not "Simple mode")

### 2. Query: Find Failed Logic App Runs

Paste the following KQL query to list failed Logic App executions:

```kusto
AzureDiagnostics
| where ResourceType == "WORKFLOWS/RUNS"
| where status_s == "Failed"
| project TimeGenerated, Resource, resource_workflowName_s, workflowId_s, status_s, error_code_s
| order by TimeGenerated desc
```

This query shows when the failures happened, on which Logic App, and what the error code was.

### 3. Query: Analyze Logic App Execution Trends

```kusto
AzureDiagnostics
| where ResourceType startswith "WORKFLOWS/RUNS"
| summarize count() by bin(TimeGenerated, 1h), Resource
| order by TimeGenerated desc
```

This shows how many times your Logic Apps ran each hour.

### 4. Query: Service Bus Queue Throughput

```kusto
AzureDiagnostics
| where ResourceType == "NAMESPACES"
| summarize operations_count = count() by bin(TimeGenerated, 1h), OperationName
| order by TimeGenerated desc
```

This query helps monitor message traffic in your queue.

### 5. Save a Query

- Click the **"Save"** button above the query editor
- Choose a meaningful name (e.g., `Failed Logic App Runs`)
- Choose to save under **My Queries** or a **Shared** folder if collaborating

### 6. (Optional) Create an Alert from a Query

To be notified when something goes wrong:

- Click **"New alert rule"** from the query editor toolbar
- Define your **Signal** (e.g., number of results > 0)
- Choose the **Resource** (your Log Analytics workspace)
- Set the **Alert logic**, **Threshold**, and **Evaluation frequency**
- Define an **Action group** (e.g., email to testers or teams)
- Click **Review + create**

> For example, trigger an alert when a failed Logic App run appears, or when message volume in Service Bus drops below a threshold.

## Summary

You've now learned how to:

- Write queries for monitoring Logic App failures and Service Bus throughput
- Visualize trends over time
- Save queries for reuse
- Create alerts to be notified of key conditions

This is critical for test automation, observability, and smooth handovers to production.

