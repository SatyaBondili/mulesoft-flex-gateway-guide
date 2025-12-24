# Mule Employee API: Design & Implementation Guide
This guide provides a step-by-step walkthrough for designing an Employee CRUD API using RAML and implementing it using Mule 4

#### 📋 Table of Contents
- [1. API Design](#1-api-design)
- [2. Publishing to Exchange](#2-publishing-to-exchange)
- [3. Create API Implementation in MuleSoft](#3-mule)
- [4. Gateway Registration](#3-gateway-registration)
- [5. Security & SLA Policy](#4-security--sla-policy)
- [6. Testing with Postman](#5-testing-with-postman)

#### 1. API Design
The API is designed using **RAML 1.0** to provide basic CRUD functionality for employee records.

##### Define the Specification
1. Navigate to **Design Center** in Anypoint Platform.
2. Create a new **API Specification** named `Employee System API`.
3. Paste the following RAML definition:

```yaml
#%RAML 1.0
title: Employee System API
version: v1
baseUri: http://localhost:8081/api

types:
  Employee:
    properties:
      id?: integer
      name: string
      role: string
      email: string

/employees:
  get:
    description: Fetch all employees
    responses:
      200:
        body:
          application/json:
            type: Employee[]
            example:
              - id: 101
                name: "Satya Bondili"
                role: "Software Developer"
                email: "satya.bondili@company.com"
  
  post:
    description: Create a new employee
    body:
      application/json:
        type: Employee
    responses:
      201:
        body:
          application/json:
            example: { "message": "Employee created successfully" }

  /{empId}:
    uriParameters:
      empId: integer
    
    get:
      description: Get details of a specific employee
      responses:
        200:
          body:
            application/json:
              type: Employee
    
    put:
      description: Update an employee record
      body:
        application/json:
          type: Employee
      responses:
        200:
          body:
            application/json:
              example: { "message": "Employee updated" }

    delete:
      description: Remove an employee from the table
      responses:
        204:
          description: Successfully deleted
```
#### 2. Publishing to Exchange
Publishing allows your API to be managed by **API Manager** and discovered by other developers within your organization.

1. In the **Design Center** editor, click the **Publish** button in the top-right corner.
2. Set the following versioning details:
    * **Asset Version:** `1.0.0`
    * **API Version:** `v1`
3. Click **Publish to Exchange**.

#### 3. Create API implementation in MuleSoft
Create Mule application for employee system api in Anypoint Studio and run at port 8083 locally.
- host: localhost
- port: 8081
- listener path: /api/*

#### 4. Gateway Registration
Use your locally running **Flex Gateway** (Docker) to manage and proxy your API traffic.

1. Navigate to **API Manager** > **Add API** > **Add New API**.
2. Select **Flex Gateway** as the runtime.
    * **Important:**: Before this step, flex-gateway must be in running mode locally and connected to Runtime Manager in Anypoint Platform.
3. Choose your **Connected Gateway** instance from the list.
4. Link the instance to the **Employee System API** asset you just published to Exchange.
5. **Configure Endpoints:**
    * **Implementation URI:** `http://host.docker.internal:8083/` (Points to your local backend service).
    * **Consumer Endpoint:** `http://localhost:8081/` (The external port users will call).
    * **Important:** In consumer endpoint use port number of flexgateway like 8081, because consumer need to hit flexgateway endpoint. Use / at the end of endpoint name.
6. Click **Save & Deploy:**
   * After deployment you can see API status as Active, which means your flex-gateway connected to your API insatnce to           download security policies.
#### 5. Security & SLA Policy
We will apply an **SLA-based Rate Limiting** policy to restrict traffic based on specific client credentials and tiers.

##### A. Create SLA Tier
1. Navigate to your API instance in **API Manager** > **SLA Tiers**.
2. Click **Add SLA Tier** (Manual or Automatic):
    * **Name:** `Silver`
    * **Quota:** `2 requests per 1 minute`.

##### B. Apply Policy
1. Go to **Policies** > **Add Policy**.
2. Select **Rate Limiting: SLA-based**.
3. Configure the DataWeave expressions for the headers:
    * **Client ID:** `#[attributes.headers['client_id']]`
    * **Client Secret:** `#[attributes.headers['client_secret']]`
4. Click **Apply**.

##### C. Request API Access
1. Go to **Exchange** and search for the **Employee System API**.
2. Click **Request Access**, create a new **Application**, and select the `Silver` tier.
3. **Important:** Copy and save your **Client ID** and **Client Secret** for testing.

#### 6. Testing with Postman
Verify the enforcement of your **SLA-based Rate Limiting** policy by simulating consumer requests using Postman.

##### Request Configuration
Set up a new request in Postman with the following details:

* **Method:** `GET`
* **URL:** `http://localhost:8081/api/employees`
* **Headers:**
    | Key | Value |
    | :--- | :--- |
    | `client_id` | `<YOUR_CLIENT_ID>` |
    | `client_secret` | `<YOUR_CLIENT_SECRET>` |

##### Result Expectations
Since the **Silver Tier** is configured for **2 requests per minute**, you should observe the following behavior:

* **✅ Success (Requests 1-2):** The gateway allows the traffic. You will receive a `200 OK` status code with the employee data payload.
* **🛑 Throttled (Request 4+):** Within the same 1-minute window, the 3rd request will be rejected. You will receive a `429 Too Many Requests` error.
* **🔒 Unauthorized:** If you remove or use incorrect headers, the gateway will return a `401 Unauthorized` status.
