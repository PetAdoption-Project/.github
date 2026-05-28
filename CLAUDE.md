# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

**Build a single service JAR for Docker:**
Each Dockerfile does `COPY target/*.jar app.jar`, so run Maven locally before `docker compose up --build`:
```bash
mvn -f <service>/pom.xml package -DskipTests
```

**Start infrastructure (Postgres, Keycloak, Vault, Kafka, observability stack):**
```bash
cd pet-adoption-infrastructure
docker compose -f docker-compose.infra.yml -f docker-compose.infra.local.yml --env-file .env.local up -d
# Wait for init containers:
docker compose -f docker-compose.infra.yml -f docker-compose.infra.local.yml logs -f vault-init keycloak-init
```

**Start all services:**
```bash
docker compose -f docker-compose.services.yml -f docker-compose.services.local.yml --env-file .env up -d --build
```

**Run tests (before committing):**
```bash
mvn -f <service>/pom.xml test
# Single test class:
mvn -f user-service/pom.xml test -Dtest=AuthServiceTest
```

**When changing `pet-adoption-commons`, install it locally first:**
```bash
mvn -f pet-adoption-commons/pom.xml install -DskipTests
```

**Stop everything:**
```bash
docker compose -f docker-compose.services.yml -f docker-compose.services.local.yml --env-file .env down
cd pet-adoption-infrastructure && docker compose -f docker-compose.infra.yml -f docker-compose.infra.local.yml --env-file .env.local down
```

## Architecture

```
Client → API Gateway (8080)  ←  single entry point, JWT validation for all traffic
              ├── /api/auth/*, /api/users/*  →  user-service  (8081)
              └── /graphql                  →  graphql-service (8082, federation gateway)
                                                  └── user subgraph → user-service

Config Server (8888)  ←  reads from pet-adoption-config/ (separate git repo)
Eureka (8761)         ←  all services register on startup
```

**Request flow for authenticated endpoints:**
1. API Gateway validates JWT with Keycloak, strips client-supplied `X-Auth-Subject` / `X-User-Roles`
2. Gateway injects `X-Auth-Subject` (keycloakId) and `X-User-Roles` from the validated token
3. Downstream services read identity from these headers — they do **not** re-validate the JWT

**Auth flow:**
- `POST /api/auth/register` → creates Keycloak user + local `User` record
- `POST /api/auth/login` → returns access token in body + `refresh_token` HttpOnly cookie (`Path=/api/auth/refresh`, `SameSite=Lax`)
- `POST /api/auth/complete-registration` → used after OAuth2 social login to finalise local profile

## API Design: REST + GraphQL Federation

The platform intentionally mixes REST and GraphQL Federation — a pattern used by companies like GitHub and Shopify.

**REST handles:**
- Auth flows (`/api/auth/*`) — token issuance, refresh cookies, OAuth redirects
- `GET /api/users/me` — simple, cacheable, well-known endpoint for current user profile

**GraphQL Federation handles:**
- Cross-service aggregations (e.g. user + their pets + adoption requests)
- Client-driven queries where the client controls which fields to fetch

**Federation architecture:**
```
Client
  │
  └── API Gateway  (single entry point — auth handled here for all traffic)
        ├── REST    → user-service             (auth + /me)
        └── GraphQL → graphql-service          (federation gateway)
                          ├── user subgraph    (user-service exposes DGS subgraph)
                          └── [future subgraphs: pet-service, adoption-service, ...]
```

API Gateway is the only entry point — all traffic (REST and GraphQL) passes through it for JWT validation. `user-service` hosts both REST endpoints and a DGS GraphQL subgraph. `graphql-service` acts as the federation gateway that stitches subgraphs together. New services should expose a subgraph rather than REST endpoints unless the use case is auth-related.

## Commons Library (`pet-adoption-commons`)

Multi-module Maven library consumed by all services. Key packages:

| Module | Package | Contents |
|--------|---------|----------|
| `service-spring-boot-starter` | `commons.exception` | `ServiceException`, `ErrorCode`, `GlobalExceptionHandler` (auto-configured) |
| `service-spring-boot-starter` | `commons.security` | `UserHeaders` — constants `X-Auth-Subject`, `X-User-Roles` |
| `user-service-commons` | `commons.user` | `UserDTO`, `Role` enum |

**Error handling pattern:**
1. Define error codes as an enum implementing `ErrorCode` (see `UserErrorCode`)
2. Add message keys to `messages_en.properties` and `messages_uk.properties`
3. Throw `ServiceException(HttpStatus, ErrorCode)` — `GlobalExceptionHandler` renders RFC 9457 `ProblemDetail`

## Configuration

Service configs live in `pet-adoption-config/config/` (served by Config Server):
- `application.yml` — shared defaults
- `<service-name>.yml` — per-service overrides
- `application-aws.yml` — AWS profile overrides

Secrets (DB credentials, Keycloak client secret, etc.) come from **Vault** — never hard-code them. Config Server pulls them at startup via Vault integration.

Each service's `application.yaml` bootstraps with:
```yaml
spring.config.import: "optional:configserver:${CONFIG_SERVER_URL}"
```

## Coding Conventions

- **MapStruct** for all DTO ↔ entity mapping — never map fields manually
- **Lombok** everywhere — `@RequiredArgsConstructor`, `@Slf4j`, `@Data`, etc.
- **Flyway** migrations: `src/main/resources/db/migration/V{n}__{Description}.sql`
- Services discover each other via Eureka — use service names, not hardcoded URLs
- i18n: error messages resolved from `messages_en.properties` / `messages_uk.properties` via `Accept-Language` header

## Testing

- **Unit (controller):** extend `BaseControllerTest` — uses `@WebMvcTest` + `MockMvc`, services are `@MockitoBean`
- **Unit (service):** plain JUnit 5 + Mockito, no Spring context
- **Repository:** `@DataJpaTest` + `@Import(JpaConfig.class)` + `@AutoConfigureTestDatabase(replace = NONE)` — requires real Postgres (Testcontainers or local)
- Naming: `*Test` for unit, `*IT` for integration

## Task Management

See `AGENTS.md` for Notion task rules (naming convention, required metadata, approval flow).
