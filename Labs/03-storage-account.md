# Module 4 – Lab: Store and Retrieve Order Data in Blob Storage

## Objective

In this lab, you'll enhance the previous Logic Apps to store the full order payload in Azure Blob Storage and retrieve it during downstream processing.

This adds durability and traceability to your integration, as full message bodies are stored for future reference or auditing.

## Scenario

When a valid order is received:

- The OrderIntakeApp stores the original JSON payload in a blob.
- The blob is named using the order ID and timestamp for uniqueness.
- The filename is passed to the Service Bus queue as part of the message.
- The OrderProcessorApp is triggered by the message, reads this filename and retrieves the blob content.

## Steps

### 1. Create a Storage Account

- In the Azure Portal, search for **Storage account**, select the one from Microsoft, and click **Create**
- Choose your existing **Resource Group** and **Region**
- Give the storage account a **globally** unique name (e.g., `{yourinitials}orders1006`)
- Select `Standard` performance and `Locally-redundant storage (LRS)`
- Click **Review + create**, then **Create**

### 2. Create a Container

- Go to the created storage account
- Under **Data storage**, select **Containers**
- Click **+ Add Container**
- Name it `orders`
- Set **Public access level** to **Private (no anonymous access)**
- Click **Create**

### 3. Grant Access to the Logic Apps
- You need to grant the Logic App's managed identities access to the Storage Account so they can create blobs and read them later.
- Go to **Access Control (IAM)** of the Storage Account
- First we give the OrderIntakeApp access to create blobs:
  - Click **+ Add > Add role assignment**
  - Role: **Storage Blob Data Contributor**
  - Assign access to: **Managed identity**
  - Click **Select Members**, Set **Managed Identity** to `Logic App`, select the `OrderIntakeApp` and click **Review + assign** (sometimes you need to click it twice)
- Next, we give the OrderProcessorApp access to read blobs:
  - Click **+ Add > Add role assignment**
  - Role: **Storage Blob Data Reader**
  - Assign access to: **Managed identity**
  - Click **Select Members**, Set **Managed Identity** to `Logic App`, select the `OrderProcessorApp` and click **Review + assign** (sometimes you need to click it twice)

### 4. Update OrderIntakeApp: Save Order in Blob

#### 4.1 Add a variable to store the blob filename
- Navigate to your **OrderIntakeApp** Logic App
- In the **Logic App Designer**, add a new action after the **When a HTTP request is received** trigger
- Add an **Initialize variable** action before storing the blob:
  - Name: `blobFilename`
  - Type: String
- Then, in the **True** branch (after the **Condition** that checks if the order is valid and before the **Send message** action), add a new action: **Append to string variable**:
  - Name: `blobFilename`
  - Value: `@{triggerBody()?['orderId']}-@{formatDateTime(utcNow(), 'yyyyMMddHHmm')}.json`

#### 4.2 Add the Create Blob action
- In the **True** branch, after the **Append to string variable** action, add a new action:
- Action: **Create blob (V2)**
- Create a new API connection using **Managed Identity**
- Enter the following details:
  - **Storage account name**: Enter the name of your storage account
  - **Folder path**: `orders`
  - **Blob name**: `@{variables('blobFilename')}`
  - **Blob content**: Use the dynamic content `Body` from the trigger (this is the raw request body)
- Container name: `orders`
- Blob name: `@{variables('blobFilename')}`
- Blob content: use the raw request body `@{triggerBody()}`
> You can only declare variables at the global level, not within scopes, conditions, and loops.

#### 4.3 Test the OrderIntakeApp
- Save the Logic App
- Use the **Test** feature to send a valid order payload
- Check the **Run history** to ensure the blob was created successfully
- Go to the **Storage Account > Containers > orders** and verify that the blob was created with the expected filename (e.g., `order123-2023-10-06T12:34:56Z.json`)

### 5. Update the Message Sent to the Queue

- In the Service Bus **Send message** action, update the Content with:

```json
{
  "orderId": "@{triggerBody()?['orderId']}",
  "status": "@{triggerBody()?['status']}",
  "blobName": "@{variables('blobFilename')}"
}
```

### 6. Update the OrderProcessorApp: Read Blob Content

- After decoding the Base64 message, add an action: **Get blob content (V2)**
- (IMPORTANT) Create a **NEW** connection using **Managed Identity**
- Container: `orders`
- Blob: use dynamic content to retrieve the blob name from the parsed message. Then prefix it with the container name:
  - Use the expression: `orders/@{body('Parse_JSON')?['blobName']}`
- (Optional) Add another **Compose** step to inspect the full blob content

## Summary

You now have:

- A full message payload stored in Blob Storage
- Lightweight metadata passed through Service Bus
- A downstream Logic App that retrieves and processes full data from blob

This structure is robust, efficient, and supports replay/debugging scenarios in production.
