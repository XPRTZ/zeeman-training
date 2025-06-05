## 🧪 Module 5: Monitor with Log Analytics

### Objective
Monitor the Logic App and Service Bus using Log Analytics.

### Steps
1. Create a **Log Analytics Workspace**.
2. Enable **diagnostic settings** for the Logic App and Service Bus namespace.
3. Send logs to the workspace.
4. Go to **Logs (Analytics)** in the workspace.
5. Run the following KQL to find failed Logic App runs:

```kusto
AzureDiagnostics
| where ResourceType == "LOGICAPPS"
| where Status_s == "Failed"
| project TimeGenerated, Resource, Status_s, ErrorCode_s
| order by TimeGenerated desc
```

6. Run another KQL to analyze Service Bus message volume:

```kusto
AzureDiagnostics
| where EntityName_s contains "orderqueue"
| summarize count() by bin(TimeGenerated, 1h)
```

7. Save queries or create alerts if needed.