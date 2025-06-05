## 🧪 Module 2: Logic Apps - Build Your First Logic App

### Objective
Create a Logic App that receives HTTP POST order data, validates it, and stores it.

### Steps
1. Go to the Azure Portal and create a **Logic App (Consumption)**.
1. Choose a region and resource group.
1. Go to the logic app and open the **Logic App Designer**.
1. Add the **HTTP Request trigger** with a POST method.
1. Define a JSON schema for the order payload (example provided by instructor).
1. Add a **Condition** to validate order data.
1. Add a **Response** action to return 200 OK for valid requests.
1. Save and test 


### Additional checks
* Check if system assigned managed identity is enabled