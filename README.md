# E-Commerce Microservices System

[![Java](https://img.shields.io/badge/Java-17-blue)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x%20%7C%204.x-6DB33F)](https://spring.io/projects/spring-boot)
[![Architecture](https://img.shields.io/badge/Architecture-Microservices-0A66C2)](https://microservices.io/)
[![Database](https://img.shields.io/badge/Database-MySQL-4479A1)](https://www.mysql.com/)
[![Build](https://img.shields.io/badge/Build-Maven-C71A36)](https://maven.apache.org/)

Central showcase repository for a production-style e-commerce backend built with Spring Boot microservices.  
This repository is the **entry point** for recruiters, portfolio visitors, and developers to understand the full system landscape across services.

---

## Project Overview

This system is split into focused backend services:
- **user-service** for signup/login/JWT validation
- **product-service** for product and category management
- **order-service** for order lifecycle and product-enriched order retrieval
- **payment-service** for Stripe payment link generation based on order data

The services are independently deployable and communicate over HTTP.

---

## Features

Combined capabilities across all repositories:

- 🔐 User registration, login, and JWT token validation
- 🔒 BCrypt password hashing and stateless auth flow
- 🛍️ Product CRUD operations with category filtering
- 🧾 Soft-delete support for products
- 📦 Order creation with multiple order items
- 🚚 Order status lifecycle management (`PENDING`, `CONFIRMED`, `SHIPPED`, `DELIVERED`, `CANCELLED`)
- 🔄 Inter-service integration:
  - `order-service` enriches order items using `product-service`
  - `payment-service` fetches order details from `order-service`
- 💳 Stripe payment link generation from order data
- ⚠️ Global exception handling with structured error responses

---

## Architecture

Each service owns its business capability and data concerns, with synchronous service-to-service communication.

- **Client Layer**: Web/mobile clients
- **Gateway Layer**: API Gateway (recommended entry point for routing, auth, and rate limiting)
- **Service Layer**: user, product, order, payment microservices
- **Data / Integration Layer**: MySQL databases + external payment provider (Stripe)

### System Flow

```text
User / Client
   |
   v
API Gateway (recommended)
   |
   +--> user-service ------> MySQL
   |
   +--> product-service ---> MySQL
   |
   +--> order-service -----> MySQL
   |         |
   |         +--> product-service (product enrichment)
   |
   +--> payment-service ---> order-service (order fetch)
             |
             +--> Stripe API (payment link)

(Planned extension) Services --> Kafka (event-driven communication)
```

> **Architecture diagram placeholder:** add system diagram image at `docs/architecture.png` when available.

---

## Microservices

| Service | Responsibility | Repository | Key Technologies |
|---|---|---|---|
| User Service | User signup/login, JWT token validation, role-aware auth flow | [ariffurmani/user-service](https://github.com/ariffurmani/user-service) | Java 17, Spring Boot 4.0.6, Spring Security, JJWT, MySQL, JPA |
| Product Service | Product CRUD, category-based retrieval, soft-delete behavior | [ariffurmani/product-service](https://github.com/ariffurmani/product-service) | Java 17, Spring Boot 3.5.5, Spring Web, Spring Data JPA, MySQL |
| Order Service | Order management, customer order retrieval, status updates, product-detail enrichment | [ariffurmani/order-service](https://github.com/ariffurmani/order-service) | Java 17, Spring Boot 4.0.6, Spring Web, Validation, JPA, MySQL, RestTemplate |
| Payment Service | Payment link generation for orders via Stripe integration | [ariffurmani/payment-service](https://github.com/ariffurmani/payment-service) | Java 17, Spring Boot 4.0.6, Stripe Java SDK, Spring Web MVC, RestTemplate |

---

## Tech Stack

| Category | Technologies Used Across Services |
|---|---|
| Backend | Java 17, Spring Boot (3.5.5 / 4.0.6), Spring Web, Spring Web MVC |
| Databases | MySQL, Spring Data JPA / Hibernate |
| Authentication | Spring Security, JWT (JJWT), BCrypt |
| Messaging | Not implemented yet in current services (Kafka mentioned as future enhancement) |
| Infrastructure | Microservices architecture, REST-based inter-service communication, API Gateway pattern (recommended) |
| DevOps Tools | Maven / Maven Wrapper, Postman collection (product-service), shell quick-start scripts |

---

## Getting Started

### Prerequisites

- Java 17+
- Maven 3.8+
- MySQL 8+
- Stripe test key (for payment-service)

### Setup Steps

1. Clone each service repository:
   - `https://github.com/ariffurmani/user-service`
   - `https://github.com/ariffurmani/product-service`
   - `https://github.com/ariffurmani/order-service`
   - `https://github.com/ariffurmani/payment-service`
2. Create required MySQL databases (as defined in each service `application.properties`).
3. Update datasource credentials and service URLs in each service config.
4. Run each service using Maven wrapper:
   ```bash
   ./mvnw spring-boot:run
   ```
5. Verify API endpoints from each service README.

### Suggested Local Run Order

1. `user-service`
2. `product-service`
3. `order-service`
4. `payment-service`

---

## Docker Compose (Starter Template)

A compose baseline can be used for local orchestration. Update ports/env vars to match your final deployment setup.

```yaml
version: "3.9"

services:
  mysql:
    image: mysql:8
    container_name: ecommerce-mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  user-service:
    image: ariffurmani/user-service:latest
    depends_on:
      - mysql
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/user-service

  product-service:
    image: ariffurmani/product-service:latest
    depends_on:
      - mysql
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/ecom-backend

  order-service:
    image: ariffurmani/order-service:latest
    depends_on:
      - mysql
      - product-service
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/order_service_db
      PRODUCT_SERVICE_BASE_URL: http://product-service:8080/product-service

  payment-service:
    image: ariffurmani/payment-service:latest
    depends_on:
      - order-service
    environment:
      ORDER_SERVICE_URL: http://order-service:8080/order-service/orders/{orderId}
      STRIPE_KEY: ${STRIPE_KEY}

volumes:
  mysql_data:
```

> Note: this is a **starter template**. Current repositories do not provide a unified compose setup in this showcase repo.

---

## API Documentation

No centralized Swagger hub is currently published. Use service-level docs/endpoints:

- **Order Service**
  - Endpoints: `POST/GET/PUT/DELETE /api/orders...`
  - Docs: [order-service/README.md](https://github.com/ariffurmani/order-service/blob/master/README.md)

- **Product Service**
  - Base: `/products`
  - Docs: [product-service/README.md](https://github.com/ariffurmani/product-service/blob/master/README.md)
  - Quick Reference: [API_QUICK_REFERENCE.md](https://github.com/ariffurmani/product-service/blob/master/API_QUICK_REFERENCE.md)
  - Postman: [POSTMAN_COLLECTION.json](https://github.com/ariffurmani/product-service/blob/master/POSTMAN_COLLECTION.json)

- **User Service**
  - Endpoints: `/user/signup`, `/user/login`, `/user/validateToken`
  - Docs: [user-service/README.md](https://github.com/ariffurmani/user-service/blob/master/README.md)

- **Payment Service**
  - Endpoint: `/payments/generatePaymentLink?orderId={id}`
  - Controller reference: [PaymentController.java](https://github.com/ariffurmani/payment-service/blob/master/src/main/java/org/furmani/paymentservice/controller/PaymentController.java)

---

## Project Structure

```text
ariffurmani-ecommerce-microservices-system/
└── README.md                      # Central system showcase (this file)
```

Related repositories:

```text
ariffurmani/
├── user-service
├── product-service
├── order-service
└── payment-service
```

---

## Future Improvements

- Add API Gateway implementation repo and route documentation
- Add centralized configuration management (Spring Cloud Config / env strategy)
- Add service discovery and resilience patterns
- Add Kafka-based async communication for order/payment events
- Add unified Docker Compose and deployment manifests
- Add centralized observability (logs, metrics, tracing)
- Add CI/CD overview with environment promotion flow
- Publish OpenAPI/Swagger docs for each service and aggregate links

---

## Author Repositories

- [https://github.com/ariffurmani](https://github.com/ariffurmani)

