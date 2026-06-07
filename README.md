# E-Commerce Microservices System

[![Java](https://img.shields.io/badge/Java-17-blue)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x%20%7C%204.x-6DB33F)](https://spring.io/projects/spring-boot)
[![Architecture](https://img.shields.io/badge/Architecture-Microservices-0A66C2)](https://microservices.io/)
[![Database](https://img.shields.io/badge/Database-MySQL-4479A1)](https://www.mysql.com/)
[![Build](https://img.shields.io/badge/Build-Maven-C71A36)](https://maven.apache.org/)
[![Payment](https://img.shields.io/badge/Payment-Stripe-635BFF)](https://stripe.com/)

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

## System Architecture

```
                    ┌──────────────────────────────────┐
                    │           API Clients            │
                    │   Web / Mobile / Postman / etc.  │
                    └────────┬─────────────────┬───────┘
                             │                 │
                        Bearer JWT        Bearer JWT
                             │                 │
          ┌──────────────────▼──┐    ┌─────────▼──────────────────┐
          │    user-service     │    │      product-service       │
          │       :8082         │    │          :8080             │
          │                     │    │                            │
          │  Signup / Login     │    │  Product CRUD              │
          │  JWT issuance       │    │  Category filtering        │
          │  Token validation   │    │  Stock management          │
          │  MySQL              │    │  Soft-delete               │
          │  (users, roles,     │    │  MySQL                     │
          │   tokens)           │    │  (products, categories)    │
          └─────────────────────┘    └────────────┬───────────────┘
                                                  │
                                       REST — stock decrement
                                       REST — product details
                                                  │
          ┌───────────────────────────────────────▼───────────────┐
          │                    order-service                      │
          │                       :8083                           │
          │                                                       │
          │  Order lifecycle: PENDING → CONFIRMED → DELIVERED     │
          │  Calls product-service to fetch details + sync stock  │
          │  MySQL (orders, order_items)                          │
          └───────────────────────────┬───────────────────────────┘
                                      │
                                   orderId
                                      │
          ┌───────────────────────────▼───────────────────────────┐
          │                  payment-service                      │
          │                      :8084                            │
          │                                                       │
          │  Accepts orderId                                      │
          │  Fetches order details from order-service             │
          │  Creates Stripe payment link via stripe-java SDK      │
          │  Stateless — no database                              │
          └───────────────────────────┬───────────────────────────┘
                                      │ HTTPS
                                      ▼
                                 Stripe API
```

---

## Microservices

| Service | Responsibility | Repository | Key Technologies |
|---|---|---|---|
| User Service | User signup/login, JWT token validation, role-aware auth flow | [ariffurmani/user-service](https://github.com/ariffurmani/user-service) | Java 17, Spring Boot 4.0.6, Spring Security, JJWT, MySQL, JPA |
| Product Service | Product CRUD, category-based retrieval, soft-delete behavior | [ariffurmani/product-service](https://github.com/ariffurmani/product-service) | Java 17, Spring Boot 3.5.5, Spring Web, Spring Data JPA, MySQL |
| Order Service | Order management, customer order retrieval, status updates, product-detail enrichment | [ariffurmani/order-service](https://github.com/ariffurmani/order-service) | Java 17, Spring Boot 4.0.6, Spring Web, Validation, JPA, MySQL, RestTemplate |
| Payment Service | Payment link generation for orders via Stripe integration | [ariffurmani/payment-service](https://github.com/ariffurmani/payment-service) | Java 17, Spring Boot 4.0.6, Stripe Java SDK, Spring Web MVC, RestTemplate |

---

## Key Engineering Highlights

**Decentralized JWT validation** — `user-service` issues tokens; `product-service` and `order-service` each validate JWTs locally using a shared HMAC secret via a custom `HandlerInterceptor`. No central auth gateway, no extra network call per request.

**Role-based access control** — All write operations (`POST`, `PUT`, `DELETE`) across product and order services require the `ADMIN` role embedded in JWT claims, enforced at the interceptor level before reaching any controller.

**Synchronous stock management** — When an order is created, `order-service` calls `product-service` to fetch product details and decrement stock in the same request flow. On cancellation or deletion, stock is incremented back. Inventory stays consistent without a message broker.

**Stateless payment service** — `payment-service` owns no database. It accepts an `orderId`, fetches order data from `order-service` at runtime, maps it to Stripe line items, and returns a Stripe-hosted payment URL. Fully stateless and horizontally scalable.

**Bounded context ownership** — Each service owns its schema entirely. No shared databases, no cross-service JPA joins. `order-service` stores only `productId` as a reference — product names and prices are fetched at order creation time and stored as a snapshot in `order_items`.

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

## End-to-End Flow: Placing an Order

```
Step 1 — Authenticate
  POST /user/login
  └── user-service validates credentials
  └── Returns signed JWT (contains email + roles)

Step 2 — Create Order  [Authorization: Bearer <token>]
  POST /orders
  └── order-service validates JWT locally using shared HMAC secret
  └── Calls GET /product/{id} on product-service
        └── Fetches name, price, available stock
  └── Calls POST /product/{id}/stock/decrement on product-service
        └── Reduces stock quantity
  └── Persists Order + OrderItems snapshot to order_service_db
  └── Returns orderId

Step 3 — Generate Payment Link
  GET /payments/generatePaymentLink?orderId={orderId}
  └── payment-service calls GET /orders/{orderId} on order-service
  └── Maps order items to Stripe line item format
  └── Calls Stripe API via stripe-java SDK
  └── Returns Stripe-hosted payment URL to client

Step 4 — Customer pays
  └── Customer completes checkout on Stripe-hosted page
  └── Stripe handles payment processing
```

---

## Docker Compose

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

---

## API Documentation

No centralized Swagger hub is published yet. Use service-level READMEs:

| Service | Base Path | Docs |
|---|---|---|
| user-service | `/user` | [README](https://github.com/ariffurmani/user-service/blob/master/README.md) |
| product-service | `/product` | [README](https://github.com/ariffurmani/product-service/blob/master/README.md) |
| order-service | `/orders` | [README](https://github.com/ariffurmani/order-service/blob/master/README.md) |
| payment-service | `/payments` | [README](https://github.com/ariffurmani/payment-service/blob/master/README.md) |

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
