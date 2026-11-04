# Discovery: Spring PetClinic README Review

**Ticket:** SPC-17 : Review the project assignment and repository documentation
**Scope:** README only. No implementation changes were made.
**Source reviewed:** Spring PetClinic Microservices README (`README.md`)

## Project overview

Spring PetClinic is a sample veterinary clinic application split into microservices. It has a Vue.js frontend, a Spring AI chatbot, and supporting infrastructure for configuration, discovery, routing, and monitoring.

## Services and responsibilities

| Service | Responsibility |
|---|---|
| Customers Service | Manages customers, owners, and pets |
| Vets Service | Manages veterinarians and specialties |
| Visits Service | Records pet visits |
| API Gateway | Single entry point; routes requests and hosts the Vue.js frontend |
| Discovery Server (Eureka) | Service registry |
| Config Server | Centralised configuration |
| GenAI Service | Chatbot interface, using OpenAI (default) or Azure OpenAI |

## Technologies identified

- Spring Boot, Spring Cloud Gateway, Spring Cloud Config, Eureka
- Resilience4j (circuit breaker)
- Micrometer Tracing with Zipkin
- Prometheus and Grafana
- Spring Boot Admin
- Chaos Monkey for Spring Boot
- Vue.js (frontend), Maven packaging (no Node.js or NPM required)
- Spring AI (chatbot)

## How the application runs

**Local with Java**
- Start Config Server and Discovery Server first, then Customers, Vets, Visits, and API Gateway.
- Tracing, Admin, Grafana, and Prometheus are optional.

**Docker Compose**
- Build images: `./mvnw clean install -P buildDocker`
- Start: `docker compose up` (Podman is also supported)
- The gateway may time out until services register with Eureka.

**Hybrid**
- `./scripts/run_all.sh` starts infrastructure in Docker and apps with `java -jar`. Logs go to `target/<app>.log`.
- `--chaos-monkey` enables Chaos Monkey. `./scripts/stop_all.sh` stops everything.

**Access points (local)**

| Component | URL |
|---|---|
| Frontend (API Gateway) | http://localhost:8080 |
| Discovery (Eureka) | http://localhost:8761 |
| Config Server | http://localhost:8888 |
| Zipkin | http://localhost:9411/zipkin/ |
| Spring Boot Admin | http://localhost:9090 |
| Grafana | http://localhost:3030 |
| Prometheus | http://localhost:9091 |

## Database configuration

- **HSQLDB (default):** In-memory, seeded with sample data at startup.
- **MySQL:** Enabled with the `mysql` Spring profile on three services: visits, customers, and vets.
- Schema is auto-created by default. Setting `initialization-mode` to `never` in the config repository disables this.
- A MySQL 8.4.5 container is suggested for testing.

## Monitoring

- Micrometer instruments the Spring Boot applications.
- Prometheus scrapes and stores metrics; Grafana displays them.
- A Petclinic dashboard is included. The Micrometer/Spring Boot dashboard (ID 4701) can be imported.
- Custom timers (`@Timed`): `petclinic.owner` and `petclinic.pet` (Customers), `petclinic.visit` (Visits).
- JMeter load test plan: `petclinic_test_plan.jmx` (location not stated in the README).

## Build and frontend observations

- Maven packages the frontend. Node.js and NPM are not required.
- CSS is compiled from SCSS with `mvn generate-resources -P css` in the API Gateway module.
- Images can be pushed to a registry by setting `REPOSITORY_PREFIX`.
- The `main` branch uses Eclipse Temurin Java 17 as the Docker base image.

## Open questions and items for practical verification

- Which Java version does the assignment require?
- The README does not give the config repository URL or the Dockerfile path used for the MySQL profile change. Confirm both in the repository.
- Confirm that `petclinic_test_plan.jmx` exists and where it is located.
- Verify the documented startup order and ports against a running local setup.
- Decide how the GenAI API key will be supplied, since the demo key is limited.
- Confirm the scope: Docker Compose only, or a later deployment target.

## Evidence

- Source: `README.md` (as supplied)
- Commands for practical verification have not been run yet.