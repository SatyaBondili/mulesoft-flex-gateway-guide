# MuleSoft Flex Gateway Setup Guide 🚀
### 📋 Table of Contents
- Overview
- Prerequisites
- Quick setup your gateway using Docker
- Configuration Modes
- POC: Apply security policies on Mule based Employee API using Flex Gateway
- POC: Apply security policies on Spring Boot based Orders API using Flex Gateway
- Troubleshooting

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

### POC: Apply security policies on Mule based Employee API using Flex Gateway
[https://github.com/SatyaBondili/mulesoft-flex-gateway-guide/blob/staging/mule-employee-api.md](https://github.com/SatyaBondili/mulesoft-flex-gateway-guide/blob/staging/mule-employee-api.md)

### POC: Apply security policies on Spring Boot based Orders API using Flex Gateway
[https://github.com/SatyaBondili/mulesoft-flex-gateway-guide/blob/staging/springboot-orders-api.md](https://github.com/SatyaBondili/mulesoft-flex-gateway-guide/blob/staging/springboot-orders-api.md)

### Troubleshooting 
[View Troubleshooting Guide](troubleshooting.md)

> [!IMPORTANT]
> **Connectivity Sync:** Ensure your Docker container has an active internet connection. The Flex Gateway must communicate with the Anypoint control plane periodically to download updated policies and validate client contracts.

[^1]: In cybersecurity, **mTLS** stands for Mutual Transport Layer Security. It is an extension of the standard TLS protocol where both parties (the client and the server) must authenticate each other using digital certificates. In standard TLS (the "S" in HTTPS), only the server proves its identity to your browser. With mTLS, the server says, "I won't talk to you unless you prove who you are, too." Why use mTLS?: Zero Trust Security: It follows the "never trust, always verify" principle. Even if an attacker gets inside your network, they can't talk to your services without a valid certificate.Common usecase: B2B APIs: When two companies exchange sensitive financial data via an API. Since the certificate is the identity, you don't need to manage or rotate complex API keys or passwords between services.
[^2]: **Envoy engine means** The engine is written in C++ programming language rather than Java, it doesn't require a Java Virtual Machine (JVM). This leads to significantly lower memory usage and faster startup times (milliseconds vs. minutes)
