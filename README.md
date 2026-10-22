# 📈 Realtime Stock Market Data Pipeline

[![Java](https://img.shields.io/badge/Java-17+-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring_Boot-3+-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![Apache Kafka](https://img.shields.io/badge/Apache_Kafka-231F20?style=for-the-badge&logo=apache-kafka&logoColor=white)](https://kafka.apache.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)](https://redis.io/)
[![Docker Compose](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)

A high-throughput, event-driven stock market data pipeline engineered with **Spring Boot 3**, **Apache Kafka**, **PostgreSQL**, and **Redis**. Designed for real-time ingestion, partitioned consumer distribution, sub-millisecond in-memory caching, and reliable persistent auditing of volatile financial streams.

---

## 🏛️ System Architecture

```mermaid
flowchart TD
    subgraph Ingestion ["Ingestion Layer"]
        A["External Market API / Producer"] -->|Poll / Stream| B["StockDataProducer"]
        B -->|JSON Events| C["Kafka Topic: stock-prices<br/>(3 Partitions)"]
    end

    subgraph Processing ["Event Streaming & Consumer Group"]
        C -->|Concurrent Listeners| D["StockConsumerService<br/>(Concurrency: 3)"]
    end

    subgraph Storage ["Hybrid Storage Tier"]
        D -->|Key-Value Store<br/>Sub-millisecond TTL| E[("Redis Cache<br/>(Port: 6379)")]
        D -->|Relational Audit<br/>Batch JDBC Writes (size=50)| F[("PostgreSQL DB<br/>(pipelinedb:5432)")]
    end

    subgraph Serving ["Serving & Observability Layer"]
        E --> G["StockController (Port: 8000)"]
        F --> G
        G -->|REST API| H["Client Applications / Dashboards"]
        G -->|Telemetry & Metrics| I["Grafana Dashboard (Port: 3000)"]
    end
```

---

## ⚡ Key Architectural Highlights

1. **Partitioned Ingestion with Apache Kafka**:
   - Topic `stock-prices` partitioned across multiple brokers to handle high ingestion bursts without backpressure.
   - Decoupled producers and consumers ensuring non-blocking operations.
2. **Dual-Tier Storage Strategy**:
   - **Hot Cache (Redis)**: Stores the latest ticker prices in-memory for instant, sub-5ms retrieval by consumer dashboards and trading bots.
   - **Cold / Persistent Audit (PostgreSQL)**: Persists full transaction history with Hibernate batch inserts (`hibernate.jdbc.batch_size: 50`) and Hikari connection pooling.
3. **Concurrent Consumer Service**:
   - Spring Kafka listener configured with `concurrency = 3` to match topic partition distribution, processing stream batches in parallel.
4. **Resilient Error Handling**:
   - Centralized `GlobalExceptionHandler` and structured `ApiResponse<T>` envelope for uniform client responses.

---

## 🔌 REST API Reference

Base URL: `http://localhost:8000/prices` (CORS enabled for local dashboards and Grafana).

| Method | Endpoint | Description | Sample Response |
| :--- | :--- | :--- | :--- |
| `GET` | `/latest` | Retrieves latest cached prices for all active tickers | `{"status": "ok", "data": {"AAPL": 182.45, "MSFT": 420.10}}` |
| `GET` | `/history/{symbol}` | Fetches chronological price records for a ticker | `{"status": "ok", "data": [{"symbol": "AAPL", "price": 182.45, ...}]}` |
| `GET` | `/anomalies` | Returns 50 most recent price ticks for volatility inspection | `{"status": "ok", "data": [...]}` |
| `GET` | `/health` | Liveness health check | `{"status": "ok", "data": "pipeline running"}` |

---

## 🛠️ Tech Stack & Dependencies

- **Framework**: Spring Boot 3.x, Spring Data JPA, Spring Kafka, Spring Data Redis
- **Language**: Java 17+
- **Message Broker**: Apache Kafka 7.4 (Confluent) + Zookeeper
- **Databases**: PostgreSQL 15, Redis 7 (Alpine)
- **Containerization**: Docker Compose
- **Build Tool**: Apache Maven (`./mvnw`)

---

## 🚀 Getting Started

### Prerequisites
- Docker & Docker Compose
- Java JDK 17+
- Maven 3.8+ (or use included `./mvnw`)

### 1. Launch Supporting Infrastructure
Start Kafka, Zookeeper, Redis, PostgreSQL, and Grafana containers:
```bash
docker-compose up -d
```

Verify running containers:
```bash
docker-compose ps
```

### 2. Configure Environment (Optional)
Defaults are pre-configured in `src/main/resources/application.yml`:
- Kafka: `localhost:9092`
- Redis: `localhost:6379`
- PostgreSQL: `localhost:5432/pipelinedb` (user: `admin`, password: `admin`)

### 3. Build & Run Application
```bash
./mvnw clean spring-boot:run
```

The pipeline will boot on port `8000`, connect to the Kafka broker, initialize the database schema via JPA Hibernate, and begin consuming stock price streams.

---

## 📊 Monitoring with Grafana

Open `http://localhost:3000` in your browser:
- **Default Credentials**: `admin` / `admin`
- Add PostgreSQL as a datasource (`host: postgres:5432`, db: `pipelinedb`) or query the `/prices/anomalies` endpoint to visualize ticker volatility in real-time.

---

## 📄 License
This project is open-source and available under the [MIT License](LICENSE).
