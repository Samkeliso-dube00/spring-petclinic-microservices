# Local Environment Setup

**Jira Task:** SPC-28
**Scope:** Prepare and document the local development environment.

## Operating System

- Operating system: Ubuntu 24.04.5 LTS.
- Architecture: x86_64.

## Java

- Project Java target: Java 17 (configured through `java.version` in the root `pom.xml`).
- Installed version: OpenJDK 21.0.12.1.
- Verification commands: `java --version` and `javac --version`.
- Build verification: `./mvnw clean install` completed successfully using Java 21.

## Maven

- Maven Wrapper is included in the repository.
- Maven Wrapper distribution: 3.9.8.
- Verification command: `./mvnw --version`.
- Build command: `./mvnw clean install`.
- Result: BUILD SUCCESS across all nine Maven modules.
- The Maven Wrapper is preferred for consistent project builds across development environments.

## Docker

- Docker Engine: 29.6.0.
- Docker Compose: 5.1.4.
- Docker service: active and enabled through systemd.
- Docker daemon is accessible by the current user without requiring `sudo`.
- Verification commands:
  - `docker --version`
  - `docker compose version`
  - `systemctl is-active docker`
  - `systemctl is-enabled docker`
  - `docker info`

## Kubernetes Tools

- kubectl: v1.36.2.
- Kustomize: v5.8.1.
- Kind: v0.33.0 (stable release).
- Kind provides local Kubernetes clusters using Docker containers as nodes.
- The previously installed Kind alpha binary was replaced with the stable release.
- The previous alpha binary is retained at `/usr/local/bin/kind.alpha-backup`.
- SHA-256 of the downloaded Kind binary matched the installed binary.
- The upstream checksum or signature was not independently verified.
- No Kind clusters have been created yet.

## Environment Configuration Decisions

### Database

- HSQLDB is the default database.
- HSQLDB runs in-memory and uses sample data.
- MySQL is optional and can be enabled using the MySQL Spring profile.
- No separate database installation is required for the default HSQLDB setup.

### Optional Services

The following supporting services are available:

- Zipkin: distributed tracing.
- Spring Boot Admin: application administration.
- Prometheus: application metrics collection.
- Grafana: metrics visualisation and dashboards.

These services are supporting components and are not all required for the basic application deployment.

### Credentials and Secrets

- GenAI API credentials are required only when enabling the AI integration.
- Credentials and API keys must be supplied through environment variables or an appropriate secrets-management mechanism.
- Credentials, API keys and passwords must never be committed to Git.

## Outstanding Work

- Kind is installed, but no Kind cluster has been created yet.
- Application deployment and runtime verification remain outstanding.
- Kubernetes deployment and application testing will be addressed in their relevant project tasks.
- Final environment verification and evidence will be recorded under SPC-29.

## Scope and Verification Notes

This document records the local environment preparation and configuration decisions for SPC-28.

A successful Maven build confirms that the project builds in the current environment. It does not, by itself, confirm that the application has started successfully or that all microservices communicate correctly.

Application runtime and deployment verification remain separate activities.
