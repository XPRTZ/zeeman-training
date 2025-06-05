## 🧪 Module 3: Service Bus Integration

### Objective
Extend the Logic App to send a message to Azure Service Bus upon receiving a valid order.

### Steps
1. Create a **Service Bus** resource, select a BASIC or STANDARD pricing ier
1. Go to the created service bus and create a **Queue**.
1. Go to IAM and add the logic app managed identity as a **Azure Service Bus Data Owner**
2. Go to your Logic App and add a **Service Bus - Send Message** action after validation.
3. Use connection string or managed identity to connect.
4. Populate the message with order metadata (e.g., Order ID, Status).
5. Save and test end-to-end by posting new order data.
6. (Optional) Create a second Logic App that triggers on queue messages and logs content.