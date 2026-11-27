# SPC-51: Repository Structure and Build Overview

**Jira:** SPC-51 - Explore Repository Structure and Build  
**Parent:** SPC-36 - Discovery and Architecture  
**Scope:** Repository structure and Maven build organisation  
**Source:** Spring PetClinic Microservices repository  
**Status:** In Progress

## 1. Objective

Understand the repository structure and root Maven build configuration before making implementation or deployment changes.

No application code or deployment configuration was changed.

## 2. Repository overview

The repository is a multi-module Spring Boot project containing application services, shared Maven configuration, container files, documentation, and scripts.

Important top-level files and directories:

| Path | Purpose |
| --- | --- |
| `pom.xml` | Root Maven project and shared build configuration |
| `mvnw`, `mvnw.cmd` | Maven Wrapper scripts |
| `.mvn/` | Maven Wrapper configuration |
| `docker/` | Container-related files |
| `docker-compose.yml` | Docker Compose configuration |
| `docs/` | Project documentation |
| `scripts/` | Supporting scripts |
| `README.md` | Project overview and usage |
| `CONTRIBUTING.md` | Contribution guidance |

## 3. Application modules

The root POM declares eight application modules:

- `spring-petclinic-admin-server`
- `spring-petclinic-api-gateway`
- `spring-petclinic-config-server`
- `spring-petclinic-customers-service`
- `spring-petclinic-discovery-server`
- `spring-petclinic-genai-service`
- `spring-petclinic-vets-service`
- `spring-petclinic-visits-service`

Their detailed responsibilities and runtime relationships are covered separately in architecture discovery.

## 4. Maven build overview

The root `pom.xml` is a Maven aggregator with `pom` packaging. It declares the application modules and centralises shared properties and dependency management.

Key findings:

- Java 17 is specified in the root properties.
- Spring Cloud dependency versions are managed through the Spring Cloud BOM.
- The Maven Wrapper provides the repository's Maven entry point.
- The `springboot` profile activates when a module contains `src/main/resources/application.yml`.
- The `buildDocker` profile configures an external container image build command.

### Plugins reviewed

| Plugin | Purpose |
| --- | --- |
| Spring Boot Maven Plugin | Generates build information |
| Git Commit ID Maven Plugin | Records Git revision information |
| Maven Enforcer Plugin | Enforces configured build requirements |
| Exec Maven Plugin | Invokes the configured container build command |

The `buildDocker` profile configures image building; Docker Compose starts services defined in `docker-compose.yml`. These are separate operations.

### Configuration observation

The Enforcer configuration contains `<version>${java.version})</version>`, including a trailing closing parenthesis. This was noted for verification. The upstream POM was preserved without modification, and practical behaviour has not yet been confirmed.

## 5. Repository inspection and evidence

The following Linux commands were used during repository inspection:

| Command | Purpose |
| --- | --- |
| `pwd` | Displays the current working directory to identify the repository location. |
| `ls -la` | Lists files and directories, including hidden entries, with details such as permissions and ownership. |
| `tree -L 1` | Displays the repository directory structure one level deep, helping identify top-level directories and files. |
| `find . -maxdepth 2 -type d` | Inspects the directory structure to identify top-level directories and module locations. |
| `grep -n '<module>' pom.xml` | Displays the module declarations in the root POM with line numbers. |

Evidence planned for SPC-51:

1. Repository location
2. Top-level listing
3. Directory structure
4. Root Maven modules and parent configuration
5. Java and build requirements

Screenshots must be checked and attached to Jira before considering the evidence complete.

## 6. Outcome

The repository structure and root Maven build configuration have been reviewed. The project uses a multi-module Maven structure with shared build configuration.

The POM analysis is complete. Practical build verification and evidence review remain before SPC-51 can be closed.
