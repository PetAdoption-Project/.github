# Pet Adoption Platform

A microservices-based backend platform for pet adoption management, built with Spring Boot 3 and Spring Cloud.

## Architecture

```
Client
  │
  ▼
API Gateway  ──────────────────────────────────────┐
  │                                                │
  ├──▶  User Service  (REST, Auth, PostgreSQL)     │
  │                                                │
  └──▶  GraphQL Service  (DGS, aggregates data)   │
                                                   │
  Eureka Server  (service discovery)               │
  Config Server  (centralized configuration)       │
                                                   │
  Infrastructure:  PostgreSQL · Keycloak · Vault · Kafka
  Observability:   Prometheus · Grafana · Loki · Tempo
```

## Services

| Service | Description |
|---|---|
| `api-gateway` | Single entry point for all traffic (REST + GraphQL), JWT validation, forwards auth headers |
| `user-service` | User registration, login, JWT auth via Keycloak; also exposes a DGS GraphQL subgraph |
| `graphql-service` | GraphQL Federation gateway (Netflix DGS), stitches subgraphs from all services |
| `config-server` | Centralized config via Spring Cloud Config |
| `eureka-server` | Service discovery via Netflix Eureka |

## Tech Stack

**Backend**
- Java 21, Spring Boot 3.5, Spring Cloud
- Spring Cloud Gateway, Netflix Eureka, Spring Cloud Config
- Spring Data JPA, Flyway, PostgreSQL
- Netflix DGS (GraphQL Federation)
- Apache Kafka
- MapStruct, Lombok

**Security**
- Keycloak (OAuth2 / OIDC)
- HashiCorp Vault (secrets management)
- JWT access tokens + HttpOnly refresh cookies

**Observability**
- Prometheus + Grafana (metrics)
- Loki + Promtail (log aggregation)
- Tempo + Micrometer Tracing (distributed tracing)

**Infrastructure**
- Docker, Docker Compose
- AWS-ready compose profiles

## API Overview

### Auth — `POST /api/auth/*`

| Endpoint | Description |
|---|---|
| `POST /api/auth/register` | Register a new user (creates Keycloak account) |
| `POST /api/auth/login` | Login — returns access token + sets HttpOnly refresh cookie |
| `POST /api/auth/refresh` | Refresh access token using cookie |
| `POST /api/auth/complete-registration` | Complete profile setup after OAuth flow |

### Users — `GET /api/users/*`

| Endpoint | Description |
|---|---|
| `GET /api/users/me` | Get current user profile (authenticated) |

### GraphQL — `POST /graphql`

Queries and mutations for user data, routed through the GraphQL service via service discovery.

## Running Locally

**Prerequisites:** Docker Desktop, Java 21, Maven

**1. Start infrastructure**
```bash
cd pet-adoption-infrastructure
docker compose -f docker-compose.infra.yml -f docker-compose.infra.local.yml --env-file .env.local up -d
```

**2. Build services**
```bash
mvn -f eureka-server/pom.xml package -DskipTests
mvn -f config-server/pom.xml package -DskipTests
mvn -f user-service/pom.xml package -DskipTests
mvn -f graphql-service/pom.xml package -DskipTests
mvn -f api-gateway/pom.xml package -DskipTests
```

**3. Start services**
```bash
docker compose -f docker-compose.services.yml -f docker-compose.services.local.yml --env-file .env up -d --build
```

## Service URLs

| Service | URL |
|---|---|
| API Gateway | http://localhost:8080 |
| GraphQL | http://localhost:8082/graphql |
| Eureka | http://localhost:8761 |
| Config Server | http://localhost:8888 |
| Keycloak | http://localhost:8180 |
| Vault | http://localhost:8200 |
| Kafka UI | http://localhost:8090 |
| Prometheus | http://localhost:9090 |
| Grafana | http://localhost:3000 |

## Project Structure

```
PetAdoption/
├── api-gateway/              # Spring Cloud Gateway
├── user-service/             # Auth + user management
├── graphql-service/          # GraphQL API (Netflix DGS)
├── config-server/            # Centralized config
├── eureka-server/            # Service registry
├── pet-adoption-commons/     # Shared DTOs, security utils
├── pet-adoption-config/      # Config files per service/profile
└── pet-adoption-infrastructure/  # Docker infra (Postgres, Keycloak, Vault, Kafka, Grafana stack)
```
