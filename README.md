# 📈 Realtime Stock Market Data Pipeline

[![Java](https://img.shields.io/badge/Java-17+-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/Spring_Boot-3+-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![Apache Kafka](https://img.shields.io/badge/Apache_Kafka-231F20?style=for-the-badge&logo=apache-kafka&logoColor=white)](https://kafka.apache.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)](https://redis.io/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)

A high-throughput, event-driven stock market data pipeline built with **Spring Boot**, **Apache Kafka**, **PostgreSQL**, and **Redis**. Designed for real-time ingestion, serialization, consumer aggregation, and sub-millisecond retrieval of financial ticker streams.

---

## 🏛️ Architecture Overview

```
[ Financial API / Stock Producer ]
              │
              ▼  (JSON Serialized Events)
     [ Apache Kafka Broker ]  (Topic: stock-prices)
              │
              ▼  (Consumer Service)
   ┌──────────┴──────────┐
   ▼                     ▼
[ PostgreSQL ]       [ Redis Cache ]
(Historical Audit)   (Realtime Cache: <5ms)
```

---

## 🚀 Quickstart & Setup

### 1. Launch Infrastructure
```bash
docker-compose up -d
```
This spins up:
- Apache Kafka & Zookeeper (`localhost:9092`)
- PostgreSQL (`localhost:5432`, db: `pipelinedb`)
- Redis (`localhost:6379`)

### 2. Run the Application
```bash
./mvnw clean spring-boot:run
```

---

## 📄 License
This project is open-source under the [MIT License](LICENSE).
