# MuleSoft Flex Gateway Setup Guide 🚀
### 📋 Table of Contents
- Overview
- Prerequisites
- Quick setup your gateway using Docker
- Configuration Modes
- Troubleshooting
- POC: Apply security policies on Employee API using Flex Gateway

### Overview
Anypoint Flex Gateway is MuleSoft’s new API gateway designed to manage and secure both non-mule and mule APIs running anywhere.
##### Note
* **Flex Gateway:** It is not like JVM which runs java applications or Mule Runtime which runs Mule Applications. It is similar to Amazon API Gateway, Google Apigee, Microsoft Azure API Management, Kong Gateway and MuleSoft's API Gateway. 
* **Running anywhere:** Means APIs deployed on client machine or client datacenter or any cloud services like AWS, GCP and AZURE.

##### Key Features
* **Enterprise Security:** Built-in policies for Rate Limiting, JWT Validation, OAuth and mTLS [^1].
##### Comparison: Flex vs. Mule Gateway
| Feature | Flex Gateway | Mule Gateway |
| :--- | :--- | :--- |
| **Engine** | Envoy (C++)[^2] | Mule Runtime (Java) |
| **API Support** | Mule & Non-Mule APIs | Only Mule APIs |

### 🛠 Prerequisites
- **Anypoint Platform Account:** (Organization Administrator or Gateway Manager role)
- **Environment:** Docker, Linux (RHEL/Ubuntu), or Kubernetes
- **Network:** Port 8081 (default traffic) and 443 (outbound to Anypoint)
- **Install Docker in Windows 11:** To run flex gateway software, docker container is required. To install docker in your windows machine, use below procedue.
#### Installing Docker on Windows 11 for Anypoint Flex Gateway

Installing Docker on Windows 11 is the foundational step for running **Anypoint Flex Gateway** (or Factor House Flex) in local mode. On Windows, Docker uses the **WSL 2 (Windows Subsystem for Linux)** backend for the best performance.

##### Step 1: Check System Requirements
Before starting, ensure your Windows 11 machine meets these criteria:

* **Processor:** 64-bit.
* **RAM:** Minimum 4GB (8GB recommended).
* **Virtualization:** Must be enabled in your BIOS/UEFI settings.

##### Step 2: Install Docker Desktop

1.  **Download the Installer:** Go to the [official Docker Desktop website](https://docs.docker.com/desktop/setup/install/windows-install/) and click **Download for Windows**.
2.  **Run the Installer:** Double-click the `.exe` file.
3.  **Configuration:** When prompted, ensure the option **"Use WSL 2 instead of Hyper-V"** is checked. This is the modern standard for Windows 11.
4.  **Restart:** After the installation finishes, Windows will prompt you to close and log out or restart your computer.
5.  **Accept Terms:** Upon restart, open Docker Desktop from the Start menu and accept the Service Agreement.

##### Step 3: Verify the Installation
Open **PowerShell** or **Command Prompt** and run the following command to ensure Docker is responsive:
```bash
docker --version
````
### Quick setup your gateway using Docker
Open CMD or Windows Powershell in your system to run below commands
1. **Pull the Image:** Download the Flex Gateway container image
   ```bash
   docker pull mulesoft/flex-gateway:latest (or docker pull mulesoft/flex-gateway) ```
2. **Register the Gateway:**
   Create a new directory called flex-registration (or similar), then register Flex Gateway to Anypoint Platform by running the following command in the created directory, replacing <gateway-name> by your own value.
   ```bash
   docker run --entrypoint flexctl -v C:/Users/bonsingh/flexgateway:/registration mulesoft/flex-gateway registration create --organization=6d87364f-f01a-4f1c-8d1d-52fe8f19402c --token=a86a2b29-096f-4a8a-8b2e-8bdf62da38e3 --output-directory=/registration --connected=true satya-flex-gateway ```

  After you have executed the command, you should see your new Flex Gateway in Runtime Manager after clicking Flex Gateway in the left navigation.
  You should also see one new file called registration.yaml where you executed the command.
  
3. **Start the Gateway:**
```bash
docker run --rm -v C:/Users/bonsingh/flexgateway:/usr/local/share/mulesoft/flex-gateway/conf.d -p 8081:8081 mulesoft/flex-gateway
```
4. **View in Anypoint Platform:**
If gateway started successfully, you can view the gateway connected in Anypoint Platform under Runtime Manager > Flex Gateways.

### Configuration Modes
* **Connected Mode:** Flex Gateway code deployed on Client Machine but Managed via the Anypoint Platform UI,
* **Local Mode:** The Flex Gateway is deployed on a client machine and managed via local declarative configuration file (YAML) on the same machine.

### Troubleshooting 
[View Troubleshooting Guide](TROUBLESHOOTING.md)
  
### POC: Apply security policies on Employee API using Flex Gateway
##### Employee System API: End-to-End Lifecycle Documentation
This repository provides a step-by-step guide for designing, publishing, securing, and testing an Employee System API using the **Anypoint Platform** and **MuleSoft Flex Gateway**.

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
title: Employee Data API
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
* **URL:** `http://localhost:8081/employees`
* **Headers:**
    | Key | Value |
    | :--- | :--- |
    | `client_id` | `<YOUR_CLIENT_ID>` |
    | `client_secret` | `<YOUR_CLIENT_SECRET>` |

##### Result Expectations
Since the **Silver Tier** is configured for **2 requests per minute**, you should observe the following behavior:

* **✅ Success (Requests 1-2):** The gateway allows the traffic. You will receive a `200 OK` status code with the employee data payload.
* **🛑 Throttled (Request 4+):** Within the same 1-minute window, the 4th request will be rejected. You will receive a `429 Too Many Requests` error.
* **🔒 Unauthorized:** If you remove or use incorrect headers, the gateway will return a `401 Unauthorized` status.

> [!IMPORTANT]
> **Connectivity Sync:** Ensure your Docker container has an active internet connection. The Flex Gateway must communicate with the Anypoint control plane periodically to download updated policies and validate client contracts.

[^1]: In cybersecurity, **mTLS** stands for Mutual Transport Layer Security. It is an extension of the standard TLS protocol where both parties (the client and the server) must authenticate each other using digital certificates. In standard TLS (the "S" in HTTPS), only the server proves its identity to your browser. With mTLS, the server says, "I won't talk to you unless you prove who you are, too." Why use mTLS?: Zero Trust Security: It follows the "never trust, always verify" principle. Even if an attacker gets inside your network, they can't talk to your services without a valid certificate.Common usecase: B2B APIs: When two companies exchange sensitive financial data via an API. Since the certificate is the identity, you don't need to manage or rotate complex API keys or passwords between services.
[^2]: **Envoy engine means** The engine is written in C++ programming language rather than Java, it doesn't require a Java Virtual Machine (JVM). This leads to significantly lower memory usage and faster startup times (milliseconds vs. minutes)
