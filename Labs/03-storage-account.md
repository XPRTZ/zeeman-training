## 🧪 Module 4: Storage Account - Store Order Payload

### Objective
Store the order data in Azure Blob Storage.

### Steps
1. Create a **Storage Account** in Azure.
2. Create a **Blob container** named `orders`.
3. In your existing Logic App, add an action: **Create Blob** (Storage Blob connector).
4. Configure the action to use the order ID as the filename and body as content.
5. Save and verify the blobs are created correctly.
6. (Optional) Add blob metadata like content type or order source.
