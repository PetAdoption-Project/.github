# 🐾 Pet Adoption Platform

A microservices-based backend platform for pet adoption management.

## Tech Stack

![Java](https://img.shields.io/badge/Java_21-ED8B00?style=flat&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot_3.5-6DB33F?style=flat&logo=spring&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat&logo=postgresql&logoColor=white)
![Keycloak](https://img.shields.io/badge/Keycloak-4D4D4D?style=flat&logo=keycloak&logoColor=white)
![Kafka](https://img.shields.io/badge/Kafka-231F20?style=flat&logo=apachekafka&logoColor=white)
![GraphQL](https://img.shields.io/badge/GraphQL-E10098?style=flat&logo=graphql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)

## Architecture

REST + GraphQL Federation over a single API Gateway entry point.

| Service | Description |
|---|---|
| `api-gateway` | Single entry point, JWT validation |
| `user-service` | Auth, user management, GraphQL subgraph |
| `graphql-service` | GraphQL Federation gateway (Netflix DGS) |
| `config-server` | Centralized configuration |
| `eureka-server` | Service discovery |
