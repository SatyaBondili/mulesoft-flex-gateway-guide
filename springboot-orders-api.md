# Orders API Implementation (Spring Boot)
This guide provides a step-by-step walkthrough for designing an orders API using RAML and implementing it using Spring Boot

#### 📋 Table of Contents
- [1. API Design](#1-api-design)
- [2. Publishing to Exchange](#2-publishing-to-exchange)
- [3. Create API Implementation in Spring Boot](#3-Springboot)
- [4. Gateway Registration](#3-gateway-registration)
- [5. Security & SLA Policy](#4-security--sla-policy)
- [6. Testing with Postman](#5-testing-with-postman)

#### 1. API Design
The API is designed using **RAML 1.0** to provide basic CRUD functionality for orders.

##### Define the Specification
1. Navigate to **Design Center** in Anypoint Platform.
2. Create a new **API Specification** named `orders-api`.
3. Paste the following RAML definition:

```yaml
#%RAML 1.0
title: Order Management API
version: v1
baseUri: http://localhost:8085/

types:
  Order:
    type: object
    properties:
      orderId?: string
      customerName: string
      amount: number
      status: 
        enum: [PENDING, SHIPPED, DELIVERED, CANCELLED]
      items: array

/orders:
  get:
    description: Retrieve a list of all orders
    responses:
      200:
        body:
          application/json:
            type: Order[]
            example: |
              [
                {"orderId": "ORD-101", "customerName": "John Doe", "amount": 150.00, "status": "PENDING", "items": ["Item A", "Item B"]}
              ]
  post:
    description: Create a new order
    body:
      application/json:
        type: Order
        example:
          customerName: "Jane Smith"
          amount: 85.50
          status: "PENDING"
          items: ["Item C"]
    responses:
      201:
        body:
          application/json:
            example: { "message": "Order created successfully", "orderId": "ORD-102" }

  /{orderId}:
    uriParameters:
      orderId:
        type: string
        description: The unique identifier for the order
    get:
      description: Get details of a specific order
      responses:
        200:
          body:
            application/json:
              type: Order
    put:
      description: Update an existing order
      body:
        application/json:
          type: Order
      responses:
        200:
          body:
            application/json:
              example: { "message": "Order updated successfully" }
    delete:
      description: Delete an order
      responses:
        204:
          description: Order deleted successfully
```
#### 2. Publishing to Exchange
Publishing allows your API to be managed by **API Manager** and discovered by other developers within your organization.

1. In the **Design Center** editor, click the **Publish** button in the top-right corner.
2. Set the following versioning details:
    * **Asset Version:** `1.0.0`
    * **API Version:** `v1`
3. Click **Publish to Exchange**.

#### 3. Create API implementation in Spring Boot

Create Spring Boot application fororders api in Intellij IDEA and run at port 8085 locally.
- host: localhost
- port: 8085
- listener path: /orders

#### Tech Stack

* **Language:** OpenJDK 17
* **Framework:** Spring Boot 4.x (Web)
* **Tools:** Maven, IntelliJ IDEA Community Edition

#### System Architecture

The application follows a clean, layered architecture to ensure maintainability and testability.
- **Controller:** Entry point for REST requests.
- **Service:** Handles business logic and transactions.
- **Repository:** Interface for database CRUD operations.
- **Entity:** Represents the `orders` table in the database.

#### 🚀 Getting Started

##### 1. Initialize via Spring Initializr
1. Go to [start.spring.io](https://start.spring.io/).
2. Select **Project:** Maven, **Language:** Java, **Spring Boot:** 4.0.1.
3. **Java Version:** OpenJdk 17.
4. Add Dependencies: `Spring Web`.

##### 2. Project Structure
```text
src/main/java/com/ps/orders
├── controller
│   └── OrderController.java
├── service
│   └── OrderService.java
├── repository
│   └── OrderRepository.java
├── model
│   └── Order.java
└── OrdersApplication.java
```
OrderController.java
```java
package com.ps.controllers;

import com.ps.model.Order;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import jakarta.validation.Valid;

import java.math.BigDecimal;
import java.util.List;
import java.util.ArrayList;

@RestController
@RequestMapping("/orders")
public class OrderController {

    /**
     * GET /orders
     * Retrieve a list of all orders
     */
    @GetMapping(produces = "application/json")
    public ResponseEntity<List<Order>> getAllOrders() {
        List<Order> orders = new ArrayList<>();

// --- Order 1: Electronics Purchase ---
        Order order1 = new Order();
        order1.setOrderId("ORD-1001");
        order1.setCustomerId("CUST-55");
        order1.setStatus("COMPLETED");
        order1.setTotalAmount(BigDecimal.valueOf(1250.00));

        Order.OrderItem item1 = new Order.OrderItem();
        item1.setProductId("PROD-MACBOOK");
        item1.setQuantity(1);
        item1.setUnitPrice(BigDecimal.valueOf(1200.00));

        Order.OrderItem item2 = new Order.OrderItem();
        item2.setProductId("PROD-MOUSE");
        item2.setQuantity(1);
        item2.setUnitPrice(BigDecimal.valueOf(50.00));

        order1.setItems(List.of(item1, item2));
        orders.add(order1);

// --- Order 2: Office Supplies ---
        Order order2 = new Order();
        order2.setOrderId("ORD-1002");
        order2.setCustomerId("CUST-88");
        order2.setStatus("PENDING");
        order2.setTotalAmount(BigDecimal.valueOf(45.98));

        Order.OrderItem item3 = new Order.OrderItem();
        item3.setProductId("PROD-PEN-PACK");
        item3.setQuantity(2);
        item3.setUnitPrice(BigDecimal.valueOf(22.99));

        order2.setItems(List.of(item3));
        orders.add(order2);

// --- Order 3: Home Decor ---
        Order order3 = new Order();
        order3.setOrderId("ORD-1003");
        order3.setCustomerId("CUST-12");
        order3.setStatus("SHIPPED");
        order3.setTotalAmount(BigDecimal.valueOf(15.00));

        Order.OrderItem item4 = new Order.OrderItem();
        item4.setProductId("PROD-CANDLE");
        item4.setQuantity(3);
        item4.setUnitPrice(BigDecimal.valueOf(5.00));

        order3.setItems(List.of(item4));
        orders.add(order3);
        // Logic to fetch orders from service
        return ResponseEntity.ok(orders);
    }

    /**
     * GET /orders/{orderId}
     * Retrieve a specific order by ID
     */
    @GetMapping(value = "/{orderId}", produces = "application/json")
    public ResponseEntity<Order> getOrderById(@PathVariable("orderId") String orderId) {
        Order order = new Order(); // Logic to fetch specific order
        return ResponseEntity.ok(order);
    }

    /**
     * POST /orders
     * Create a new order
     */
    @PostMapping(consumes = "application/json", produces = "application/json")
    public ResponseEntity<Order> createOrder(@Valid @RequestBody Order order) {
        // Logic to save the order
        return ResponseEntity.status(HttpStatus.CREATED).body(order);
    }

    /**
     * DELETE /orders/{orderId}
     * Remove an order
     */
    @DeleteMapping("/{orderId}")
    public ResponseEntity<Void> deleteOrder(@PathVariable("orderId") String orderId) {
        // Logic to delete the order
        return ResponseEntity.noContent().build();
    }
}

```
#### 4. Gateway Registration
Use your locally running **Flex Gateway** (Docker) to manage and proxy your API traffic.

1. Navigate to **API Manager** > **Add API** > **Add New API**.
2. Select **Flex Gateway** as the runtime.
    * **Important:**: Before this step, flex-gateway must be in running mode locally and connected to Runtime Manager in Anypoint Platform.
3. Choose your **Connected Gateway** instance from the list.
4. Link the instance to the **orders api** asset you just published to Exchange.
5. **Configure Endpoints:**
    * **Implementation URI:** `http://host.docker.internal:8085/` (Points to your local backend service).
    * **Consumer Endpoint:** `http://localhost:8081/` (The external port users will call).
    * **Important:** In consumer endpoint use port number of flexgateway like 8081, because consumer need to hit flexgateway endpoint. Use / at the end of endpoint name.
6. Click **Save & Deploy:**
   * After deployment you can see API status as Active, which means your flex-gateway connected to your API insatnce to           download security policies.
#### 5. Security & SLA Policy
We will apply an **SLA-based Rate Limiting** policy to restrict traffic based on specific client credentials and tiers.

##### A. Create SLA Tier
1. Navigate to your API instance in **API Manager** > **SLA Tiers**.
2. Click **Add SLA Tier** (Manual or Automatic):
    * **Name:** `Gold`
    * **Quota:** `5 requests per 1 minute`.

##### B. Apply Policy
1. Go to **Policies** > **Add Policy**.
2. Select **Rate Limiting: SLA-based**.
3. Configure the DataWeave expressions for the headers:
    * **Client ID:** `#[attributes.headers['client_id']]`
    * **Client Secret:** `#[attributes.headers['client_secret']]`
4. Click **Apply**.

##### C. Request API Access
1. Go to **Exchange** and search for the **orders-api**.
2. Click **Request Access**, create a new **Application**, and select the `Gold` tier.
3. **Important:** Copy and save your **Client ID** and **Client Secret** for testing.

#### 6. Testing with Postman
Verify the enforcement of your **SLA-based Rate Limiting** policy by simulating consumer requests using Postman.

##### Request Configuration
Set up a new request in Postman with the following details:

* **Method:** `GET`
* **URL:** `http://localhost:8081/ordersapi/orders`
* **Headers:**
    | Key | Value |
    | :--- | :--- |
    | `client_id` | `<YOUR_CLIENT_ID>` |
    | `client_secret` | `<YOUR_CLIENT_SECRET>` |

##### Result Expectations
Since the **Gold Tier** is configured for **5 requests per minute**, you should observe the following behavior:

* **✅ Success (Requests 1-5):** The gateway allows the traffic. You will receive a `200 OK` status code with the employee data payload.
* **🛑 Throttled (Request 5+):** Within the same 1-minute window, the 6th request will be rejected. You will receive a `429 Too Many Requests` error.
* **🔒 Unauthorized:** If you remove or use incorrect headers, the gateway will return a `401 Unauthorized` status.

