# Microservices Architectural Patterns — SE4010

A hands-on project demonstrating microservices architectural patterns using **Spring Boot**, **Spring Cloud Gateway**, **Apache Kafka**, and **Docker**. The repository is structured into four modules covering an API Gateway pattern, basic REST backend services, an event-driven Kafka-based architecture, and a standalone Spring Boot introductory demo.

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Modules](#modules)
  - [1. API Gateway](#1-api-gateway)
  - [2. Backend Services](#2-backend-services)
  - [3. Kafka Example (Event-Driven)](#3-kafka-example-event-driven)
  - [4. Spring Boot Intro](#4-spring-boot-intro)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [API Reference](#api-reference)

---

## Architecture Overview

```
                        ┌─────────────────┐
                        │   API Gateway   │
                        │   (port 8080)   │
                        └────────┬────────┘
               ┌─────────────────┼──────────────────┐
               ▼                 ▼                  ▼
     ┌──────────────┐  ┌──────────────┐   ┌──────────────────┐
     │ Order Service│  │ Payment Svc  │   │ Inventory Service│
     │  (port 8081) │  │ (port 8082)  │   │   (port 8083)    │
     └──────────────┘  └──────────────┘   └──────────────────┘

        ── Kafka Event-Driven Architecture ──

     ┌──────────────┐   order.created   ┌──────────────────┐
     │ Order Service│ ─────────────────►│ Inventory Service│
     │  (port 8081) │                   │   (port 8082)    │
     └──────────────┘        │          └──────────────────┘
                             │          ┌──────────────────────┐
                             └─────────►│ Notification Service │
                                        │      (port 8083)     │
                                        └──────────────────────┘
```

---

## Modules

### 1. API Gateway

**Path:** `API-Gateway/`

A **Spring Cloud Gateway** instance that acts as the single entry point for all client requests, routing them to the appropriate backend service.

| Property | Value |
|---|---|
| Port | `8080` |
| Spring Boot | `3.2.0` |
| Java | `17` |
| Spring Cloud | `2023.0.0` |

**Routing Table**

| Path Prefix | Target Service |
|---|---|
| `/inventory/**` | `http://host.docker.internal:8083` |
| `/payments/**` | `http://host.docker.internal:8082` |
| `/orders/**` | `http://host.docker.internal:8081` |

**Run with Docker:**
```bash
cd API-Gateway
docker build -t api-gateway .
docker run -p 8080:8080 api-gateway
```

---

### 2. Backend Services

**Path:** `BackEndServices/`

Three independent REST microservices, each with its own H2 in-memory database, demonstrating the **Database per Service** pattern. All services use Spring Boot `3.3.2` and Java `17`.

---

#### Inventory Service — port `8083`

Manages stock items by SKU.

**Model: `InventoryItem`**
| Field | Type |
|---|---|
| `id` | `Long` (auto) |
| `sku` | `String` |
| `quantity` | `Integer` |

**Endpoints**
| Method | Path | Description |
|---|---|---|
| `GET` | `/inventory` | List all inventory items |
| `POST` | `/inventory` | Create a new inventory item |

---

#### Order Service — port `8081`

Manages customer orders.

**Model: `Order`**
| Field | Type |
|---|---|
| `id` | `Long` (auto) |
| `customerName` | `String` |
| `totalAmount` | `BigDecimal` |

**Endpoints**
| Method | Path | Description |
|---|---|---|
| `GET` | `/orders` | List all orders |
| `POST` | `/orders` | Create a new order |

---

#### Payment Service — port `8082`

Tracks payments linked to orders.

**Model: `Payment`**
| Field | Type |
|---|---|
| `id` | `Long` (auto) |
| `orderId` | `Long` |
| `amount` | `BigDecimal` |
| `status` | `String` |

**Endpoints**
| Method | Path | Description |
|---|---|---|
| `GET` | `/payments` | List all payments |
| `POST` | `/payments` | Create a new payment |

**H2 console** is available at `http://localhost:<port>/h2-console` for each service.

**Build & Run (Maven):**
```bash
cd BackEndServices/<service-name>
./mvnw spring-boot:run
```

---

### 3. Kafka Example (Event-Driven)

**Path:** `Kaffka-Example/`

Demonstrates an **event-driven microservices** architecture using Apache Kafka. When an order is placed, an `order.created` event is published to Kafka and consumed asynchronously by the Inventory and Notification services.

#### Services

| Service | Port | Role |
|---|---|---|
| Gateway | `8080` | API Gateway (routes to all services) |
| Order Service | `8081` | REST API — publishes `order.created` events |
| Inventory Service | `8082` | Consumes `order.created` — updates stock |
| Notification Service | `8083` | Consumes `order.created` — sends notifications |
| Kafka Broker | `9092` | Message broker (KRaft mode) |
| Kafka UI | `8088` | Web dashboard for Kafka management |

#### Event Flow

```
Client → Gateway (8080) → Order Service (8081)
                                │
                     Publishes: order.created
                                │
               ┌────────────────┴───────────────┐
               ▼                                ▼
    Inventory Service (8082)       Notification Service (8083)
```

#### Run with Docker Compose

```bash
cd Kaffka-Example
docker compose up --build
```

- **Kafka UI:** http://localhost:8088
- **Gateway:** http://localhost:8080

#### Kafka Configuration

| Setting | Value |
|---|---|
| Bootstrap Servers | `kafka:9092` |
| Order Topic | `order.created` |
| Order Service Consumer Group | `order-service-group` |
| Inventory Consumer Group | `inventory-service-group` |
| Notification Consumer Group | `notification-service-group` |

A Postman collection is included at `Kaffka-Example/postman/` to test the full event-driven flow.

---

### 4. Spring Boot Intro

**Path:** `Spring-boot-intro/`

A standalone introductory Spring Boot application demonstrating CRUD operations, Spring Security, and Swagger/OpenAPI documentation.

| Property | Value |
|---|---|
| Port | (default `8080`) |
| Spring Boot | `3.2.5` |
| Java | `21` |
| Database | H2 in-memory (`laptopstore`) |

**Model: `Student`**
| Field | Type |
|---|---|
| `id` | `Long` (auto, read-only) |
| `name` | `String` |
| `email` | `String` |
| `age` | `int` |

**Endpoints**
| Method | Path | Description |
|---|---|---|
| `GET` | `/api/students` | List all students |
| `GET` | `/api/students/{id}` | Get student by ID |
| `POST` | `/api/students` | Create student |
| `PUT` | `/api/students/{id}` | Update student |
| `DELETE` | `/api/students/{id}` | Delete student |
| `GET` | `/api/students/test` | Health check |

**Swagger UI** is available at `http://localhost:8080/swagger-ui.html` once the app is running.

**Run:**
```bash
cd Spring-boot-intro
./mvnw spring-boot:run
```

---

## Tech Stack

| Technology | Purpose |
|---|---|
| Spring Boot 3.x | Microservice framework |
| Spring Cloud Gateway | API Gateway / routing |
| Spring Data JPA | Data persistence |
| Apache Kafka | Async event-driven messaging |
| H2 Database | In-memory database (dev/demo) |
| Spring Security | Authentication & authorization |
| SpringDoc OpenAPI | Swagger API documentation |
| Docker / Docker Compose | Containerization & orchestration |
| Java 17 / 21 | Runtime |

---

## Getting Started

### Prerequisites

- Java 17 or 21
- Maven 3.8+
- Docker & Docker Compose

### Running the Basic REST Architecture

```bash
# Start backend services (run each in a separate terminal)
cd BackEndServices/inventory-service && ./mvnw spring-boot:run
cd BackEndServices/order-service     && ./mvnw spring-boot:run
cd BackEndServices/payment-service   && ./mvnw spring-boot:run

# Start API Gateway
cd API-Gateway && ./mvnw spring-boot:run
```

All requests can then be routed through the gateway at `http://localhost:8080`.

### Running the Kafka Architecture

```bash
cd Kaffka-Example
docker compose up --build
```

---

## API Reference

All endpoints are accessible through the API Gateway at `http://localhost:8080`:

| Resource | Method | Path |
|---|---|---|
| Orders | `GET` | `http://localhost:8080/orders` |
| Orders | `POST` | `http://localhost:8080/orders` |
| Payments | `GET` | `http://localhost:8080/payments` |
| Payments | `POST` | `http://localhost:8080/payments` |
| Inventory | `GET` | `http://localhost:8080/inventory` |
| Inventory | `POST` | `http://localhost:8080/inventory` |