## 🛠 Troubleshooting Guide

Use this section to identify and resolve common issues encountered while using this project.

### Detailed Error Solutions

<details>
<summary><b>1. 503 Service Unavailable (Upstream Connect Error)</b></summary>

**Error Message:** `upstream connect error or disconnect/reset before headers. reset reason: remote connection failure, transport failure reason: delayed connect error: Connection refused`

**The Fix:**
1. Makesure API implementation is in running status like Spring boot app or Mule App.

</details>

<details>
<summary><b>2. 404 Not Found</b></summary>

**Error Message:** `404 Not Found`

**The Fix:**
1. **Verify URI Path:** Ensure the resource URL you are calling matches the path defined in your API.
</details>

<details>
<summary><b>3. API Manager - Flex Gateway Config</b></summary>

1. **Upstream URL:** Makesure host mentioned as `host.docker.internal` when docker running in your laptop/system.
2. **Base path:** Makesure basepath ends with forward slash (`/`).
3. **Port:** When you want to use same port to manage different APIs then base path must be different.
4. **What is API Instance:**
   - API Instance and API Implementation are not same.
   - API instance is created by API Manager when you register your API from exchange. As a result you will see API Instance ID also called API Auto Discovery ID.
   - API instance is like a proxy application whose main target is to validate incoming requests.
   - Mule App downloads this proxy app or API instance while starting the mule runtime, if you configured API Auto discovery Id in your mule application.
   - API instance or Proxy application gets deployed in flexgateway software which is running in docker container of your laptop. Proxy application acts as broker between API Manager and flexgateway.
   - API instance or Proxy application downloads security polices when you add any security policies on your API in API Manager. And also it shares data to API Manager. 
</details>
