# Module 3 – Lab: Service Bus Integration

## Objective

In this lab, you'll extend your existing Logic App to send validated order data to an Azure Service Bus queue. You'll also create a second Logic App that listens for new messages on the queue and logs their content.

This lab demonstrates how to integrate Logic Apps with messaging systems to support asynchronous processing.

## Scenario

When a valid order is received via HTTP, your Logic App should:

- Forward the order metadata to a Service Bus queue
- Trigger a second Logic App whenever a new message appears in that queue
- Log the message content for inspection

## Steps

### 1. Create a Service Bus Resource

- In the Azure Portal, search for **Service Bus** and click **Create**
- Choose your existing **Resource Group** and **Region**
- Enter a unique **Namespace name**
- Select the `BASIC` or `STANDARD` pricing tier
- Click **Review + Create**, then **Create**

### 2. Create a Queue

- Go to your newly created **Service Bus Namespace**
- In the left menu, select **Queues**, then click **+ Queue**
- Give the queue a name (e.g., `orders`)
- Leave default settings and click **Create**

### 3. Grant Access to Logic App

- Still within the Service Bus resource, go to **Access control (IAM)**
- Click **+ Add > Add role assignment**
- Role: **Azure Service Bus Data Sender**
- Assign access to: **Managed identity**
- Click **Select members**
- Set **Managed identity** to `Logic App`
- Select your existing Logic App (OrderIntakeApp)
- Click **Select**
- Click **Review + assign** to apply the role (sometimes you need to click it twice)
- Go to the **Overview** tab of your Service Bus Namespace
- Copy the **Host name** for later use in the Logic App

### 4. Update Logic App to Send Messages

- Go to your **OrderIntakeApp** Logic App
- In the **Logic App Designer**, beneath the **True** branch, click the **+** icon and select **insert a new step**
- Search for **Service Bus** and select **Send message**
- Choose **Service Bus - Send message** action
- Choose to connect using **Managed Identity**
- In the **Namespace Endpoint** field, paste the **Service bus hostname** you copied earlier. 
  - **N.B.** You have to manually prefix it with `sb://` (e.g., `sb://your-namespace.servicebus.windows.net`)
- Once your connection is established, select the **Queue name** you created (e.g., `orders`)
- You need to fill the **Content** parameter, if it is not shown, open the **Advanced parameters** dropdown and select it
- In the **Content** field, you can use the dynamic content from the previous steps to build your message body. For example, if you want to send the order ID and status, you can do the following:

```json
{
  "orderId": "@{triggerBody()?['orderId']}",
  "status": "@{triggerBody()?['status']}"
}
```
- Click **Save**

### 5. Create the Listener Logic App (OrderProcessorApp)

- Follow the same steps as in lab 1 to create a new Logic App named `OrderProcessorApp` (see step 3 in Lab 1)
  - Make sure to use the same **Resource Group** and **Region** as your first Logic App
  - Name the Logic App `OrderProcessorApp`
- After creation, go to **Identity** under your Logic App settings and enable System-assigned managed identity.
- Then, in the **Service Bus Namespace**, go to **Access control (IAM)** and assign the **Azure Service Bus Data Receiver** role to this Logic App.

### 6. Add a Trigger: Peek-Lock Service Bus Message

- In the **Logic App Designer**, click **Add a trigger**
- Search for **Service Bus** and select **When a message is received in a queue (peek-lock)**
- **Don't re-use** the existing connection created before in the other logic app; instead, create a new one using **Managed Identity**.
- In the **Namespace Endpoint** field, paste the same **Service bus hostname** you used earlier (e.g., `sb://your-namespace.servicebus.windows.net`)
- Set the correct **Queue name** (e.g., `orders`)
- Leave the other settings as default for now
> Microsoft: Each managed identity that accesses a queue or topic subscription should use its own Service Bus API connection.

### 7. Add an Action: Process Message

- Click the **+** icon below the trigger and select **Add an action**
- Add a **Compose** action and set the **Inputs** to the message body.
- You need to decode the message content since it will be in Base64 format. You can use the following expression:

```json
@decodeBase64(triggerBody()?['ContentData'])
```
- The first part of the expression decodes the Base64 content, and the second part extracts the `ContentData` field from the message body.
> When using the service bus trigger, the message content is delivered in Base64. This is expected behavior and must be decoded to read the original message body.

### 8. Add Complete Message Action

- After processing the message, you need to complete it so it is removed from the queue.
- Click the **+** icon below the **Compose** action and select **Add an action**
- Search for **Service Bus** and select **Complete the message in a queue**
- For the **Queue name**, select the same queue you used earlier (e.g., `orders`)
- For the **Lock token**, use the dynamic content from the trigger. Type '/' and select **Insert Dynamic Content**. Then select **Lock token** from the list.
- Click **Save**

### 9. Save and Test End-to-End

- Click **Save** in both Logic Apps
- Use Postman, curl, or the provided test script to send a request like you did in Lab 1
- Verify:
  - The HTTP-triggered Logic App accepts the order and sends it to Service Bus
  - The queue-triggered Logic App picks up the message and processes it
  - The service bus queue is empty after processing
- Check the run history of both Logic Apps to see the execution details and any errors

## Summary
You now have a complete integration flow:

- HTTP order intake
- Validation and queuing
- Downstream processing via Service Bus

This decoupled architecture is powerful for scalable, async integrations.
