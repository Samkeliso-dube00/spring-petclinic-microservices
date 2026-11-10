# Local Environment Setup

**Jira Task:** SPC-28
**Scope:** Prepare and document the local development environment.

## Operating System

- Ubuntu 24.04.5 LTS
- Architecture: x86_64

## Java

- Project requirement: Java 17 or higher.
- Installed version: OpenJDK 21.0.12.1.
- Verification: `java --version` and `javac --version`.
- Build verification: `./mvnw clean install` completed successfully.

## Maven

- Maven Wrapper is included in the repository.
- Installed Maven version: 3.9.8.
- Verification: `./mvnw --version`.
- Build command: `./mvnw clean install`.
- Result: BUILD SUCCESS across all nine Maven modules.
- The Maven Wrapper is preferred for consistent project builds.

## Docker

- Docker Engine: 29.6.0.
- Docker Compose: 5.1.4.
- Docker service: active and enabled through systemd.
- Docker daemon accessible by the current user without sudo.
- Verification: `docker --version`, `docker compose version`,
  `systemctl is-active docker`, and `docker info`.

## Kubernetes Tools

- kubectl: v1.36.2.
- Kustomize: v5.8.1.
- Kind: v0.33.0 (stable).
- Kind is used to create local Kubernetes clusters using Docker containers.
- The previously installed Kind alpha binary was replaced with the stable release.
- The previous binary is retained at `/usr/local/bin/kind.alpha-backup`.
- The installed Kind binary SHA-256 was compared with the downloaded binary and matched.
- No Kind clusters have been created yet.

## Environment Configuration Decisions

### Database

- HSQLDB is the default database.
- HSQLDB runs in-memory and uses sample data.
- MySQL is optional and can be enabled using the MySQL Spring profile.
- No separate database installation is required for the default setup.

### Optional Services

- Zipkin provides distributed tracing.
- Spring Boot Admin provides application administration.
- Prometheus collects application metrics.
- Grafana provides metrics dashboards.
- These services are optional for the basic application deployment.

### Credentials and Secrets

- GenAI API credentials are required only when enabling the AI integration.
- Credentials and API keys must be supplied through environment variables
  or an appropriate secrets-management mechanism.
- Never commit credentials, API keys or passwords to Git.

## Outstanding Work

- No Kind cluster has been created yet.
- Application deployment and runtime verification remain outstanding.
- Final environment verification and evidence will be recorded under SPC-29.
