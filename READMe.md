# MicroService Product Platform

A work-in-progress e-commerce backend application built with **Java 25, Spring Boot and Spring Cloud**. The project explores microservices architecture by separating core business functionalities into independent services, with centralized API routing, service discovery, fault tolerance and distributed tracing.

## 🚀 Project Overview

The MicroService Product Platform demonstrates how an e-commerce backend can be developed using a distributed microservices architecture.

The application separates business responsibilities into independent services:

- **Product Service:** Manages product information.
- **Order Service:** Handles customer orders and validates product availability.
- **Inventory Service:** Checks product stock availability.
- **Discovery Server:** Provides service registration and discovery using Netflix Eureka.
- **API Gateway:** Provides centralized request routing to backend services.

Each service is organized as a separate Maven module within a parent project.

## 🏗️ System Architecture

```text
                         Client
                           |
                           v
                     API Gateway
                        :8080
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
       Product Service  Order Service  Inventory Service
          MongoDB         MySQL           MySQL
                            |
                            |
                            v
                    Inventory Service
                    (Load-balanced call)

             +-------------+-------------+
                           |
                           v
                    Eureka Discovery
                        :8761

       Distributed Tracing: Micrometer + Zipkin
```

### How the architecture works

1. Clients send requests through the API Gateway.
2. The API Gateway uses Eureka service discovery and load-balanced routing to locate backend services.
3. Product Service manages product information using MongoDB.
4. Order Service communicates with Inventory Service using a load-balanced WebClient.
5. Inventory Service checks stock availability using SKU codes.
6. Order Service saves an order only when the requested items are reported as in stock.
7. Resilience4j provides fault-tolerance mechanisms for inventory communication.
8. Micrometer Tracing and Zipkin provide distributed tracing capabilities.

## 📦 Project Structure

```text
microService-product/
│
├── api-gateway/
│   └── src/
│
├── discovery-server1/
│   └── src/
│
├── product-service/
│   └── src/
│
├── order-service/
│   └── src/
│
├── inventory_service/
│   └── src/
│
├── pom.xml
└── README.md
```

## 🔧 Microservices

### 1. Product Service

Responsible for managing product information.

**Features**
- Create new products.
- Retrieve all available products.
- Store product information in MongoDB.
- Register with Eureka Discovery Server.

**API Endpoints**

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/product` | Create a product |
| GET | `/api/product` | Retrieve all products |

**Example product request**

```json
{
  "name": "iPhone 13",
  "description": "Apple smartphone",
  "price": 1200.00
}
```

### 2. Order Service

Responsible for processing customer orders.

**Features**
- Accept order requests.
- Generate unique order numbers.
- Communicate with Inventory Service.
- Validate stock availability before saving orders.
- Apply Circuit Breaker, Retry and TimeLimiter mechanisms.
- Persist order information using MySQL.

**API Endpoint**

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/order` | Place an order |

**Example order request**

```json
{
  "orderLineItemsDtoList": [
    {
      "skuCode": "SKU-001",
      "price": 1200.00,
      "quantity": 1
    }
  ]
}
```

### 3. Inventory Service

Responsible for checking product availability.

**Features**
- Retrieve inventory information using SKU codes.
- Check whether requested products are in stock.
- Persist inventory information using MySQL.
- Register with Eureka Discovery Server.

**API Endpoint**

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/inventory?skuCode=SKU-001` | Check stock availability |

Multiple SKU codes can be supplied using repeated `skuCode` query parameters.

### 4. Discovery Server

Built using Netflix Eureka.

**Responsibilities**
- Register participating microservices.
- Maintain the service registry.
- Enable service discovery.
- Support load-balanced service communication.

**Dashboard**

```text
http://localhost:8761
```

The Eureka dashboard is protected using HTTP Basic authentication.

### 5. API Gateway

Built using Spring Cloud Gateway Server WebMVC.

**Responsibilities**
- Provide a centralized entry point.
- Route requests to backend services.
- Use Eureka-based service discovery.
- Support JWT-based authentication through Spring Security OAuth2 Resource Server.

**Configured routes**

| Route | Destination |
|---|---|
| `/api/product/**` | Product Service |
| `/api/order/**` | Order Service |
| `/api/inventory/**` | Inventory Service |
| `/eureka/**` | Eureka Server |

## 🛡️ Fault Tolerance and Resilience

The Order Service uses Resilience4j to improve reliability when communicating with the Inventory Service.

Implemented mechanisms include:

- **Circuit Breaker:** Helps prevent repeated calls to an unhealthy service.
- **Retry:** Allows failed operations to be attempted again.
- **TimeLimiter:** Limits the time spent waiting for an operation to complete.
- **Fallback Method:** Provides an alternative response when the protected operation fails.

## 📊 Distributed Tracing

The project includes Micrometer Tracing and Zipkin integration.

These technologies help track requests as they travel between microservices.

Zipkin endpoint:

```text
http://localhost:9411
```

Zipkin must be running and configured to receive the application's trace data.

## 🛠️ Technology Stack

| Category | Technologies |
|---|---|
| Programming Language | Java 25 |
| Backend Framework | Spring Boot 4.1.2 |
| Microservices | Spring Cloud |
| Service Discovery | Netflix Eureka |
| API Gateway | Spring Cloud Gateway Server WebMVC |
| Inter-service Communication | Spring WebClient |
| Fault Tolerance | Resilience4j |
| Product Database | MongoDB |
| Order Database | MySQL |
| Inventory Database | MySQL |
| Security | Spring Security, OAuth2 Resource Server, JWT |
| Distributed Tracing | Micrometer Tracing, Zipkin |
| ORM | Spring Data JPA, Hibernate |
| Build Tool | Apache Maven |
| Testing | JUnit 5, Spring Boot Test, Testcontainers |

## ⚙️ Prerequisites

Before running the application, ensure you have:

- JDK 25
- Apache Maven
- MySQL Server
- MongoDB
- Docker Engine (required for the Product Service Testcontainers integration test)
- Keycloak or another compatible JWT issuer configured for the gateway
- Zipkin (for distributed tracing)

## 🚀 Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/tripleAhoy/microService-product.git
```

### 2. Navigate into the project

```bash
cd microService-product
```

### 3. Configure databases

Create the required databases:

```sql
CREATE DATABASE order_service;
CREATE DATABASE inventory_service;
```

The Product Service uses MongoDB with the database named `test`.

### 4. Configure application properties

Configure the database connections, Eureka authentication and JWT issuer in the relevant service configuration files.

For database credentials, use environment-variable placeholders rather than committing passwords to the repository.

Example configuration:

```properties
spring.datasource.url=${ORDER_DB_URL:jdbc:mysql://localhost:3306/order_service}
spring.datasource.username=${ORDER_DB_USERNAME:root}
spring.datasource.password=${ORDER_DB_PASSWORD}

spring.jpa.hibernate.ddl-auto=update
```

Apply the corresponding configuration to the Inventory Service and Product Service.

Also configure the Eureka client credentials consistently across the services.

The API Gateway's JWT issuer must match the issuer configured in your identity provider.

### 5. Start the supporting infrastructure

Start:
- MySQL
- MongoDB
- Eureka Discovery Server
- Keycloak with the configured realm
- Zipkin

### 6. Start the microservices

Start the services in the following order:

1. Discovery Server
2. Product Service
3. Inventory Service
4. Order Service
5. API Gateway

You can run each Spring Boot application from its module using IntelliJ IDEA or Maven.

### 7. Access the application

| Component | Address |
|---|---|
| API Gateway | `http://localhost:8080` |
| Eureka Dashboard | `http://localhost:8761` |
| Zipkin Dashboard | `http://localhost:9411` |
| Keycloak | `http://localhost:8181` |

The gateway requires a valid JWT for protected requests. The Keycloak issuer must be configured correctly.

## 🧪 Testing

Run the tests from the parent project:

```bash
mvn test
```

The Product Service includes an integration test using MongoDB Testcontainers.

The Order and Inventory Services currently include Spring application context-loading tests.

## 📌 Development Status

This project is actively under development.

### Implemented in the current source code

- [x] Multi-module Maven project structure
- [x] Product Service with MongoDB integration
- [x] Order Service with MySQL integration
- [x] Inventory Service with MySQL integration
- [x] Eureka Discovery Server
- [x] API Gateway with service routing
- [x] Load-balanced WebClient communication
- [x] Resilience4j configuration
- [x] Micrometer and Zipkin tracing configuration
- [x] JWT resource-server configuration
- [x] Product Service Testcontainers integration test

### Planned improvements

- [ ] Complete end-to-end service integration testing
- [ ] Improve inventory and order workflows
- [ ] Add inventory management endpoints
- [ ] Improve request validation and exception handling
- [ ] Improve security and externalize all credentials
- [ ] Add comprehensive automated tests
- [ ] Add Dockerfiles and Docker Compose orchestration
- [ ] Complete API documentation
- [ ] Prepare production deployment

## 🎯 Project Objectives

The objectives of this project are to:

- Develop practical experience with microservices architecture.
- Understand service discovery and load balancing.
- Implement reliable inter-service communication.
- Explore fault-tolerance patterns in distributed systems.
- Implement authentication and authorization.
- Work with both relational and NoSQL databases.
- Understand distributed tracing and application observability.

## 👨‍💻 Author

**Olawale Adegbite**

Software Engineer | Java | Spring Boot | Microservices

- GitHub: [@tripleAhoy](https://github.com/tripleAhoy)
- LinkedIn: [Olawale Adegbite](https://www.linkedin.com/in/adegbite-adewale-860a22243/)

---

⭐ If you find this project interesting, feel free to explore the repository and follow its development.

*This is a learning and development project. Features and architecture are subject to change.*