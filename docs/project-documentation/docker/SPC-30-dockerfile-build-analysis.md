# SPC-30 : Dockerfile and Build Process Analysis

## 1. Objective

Inspect the existing Spring PetClinic Dockerfiles and understand how application services are packaged into Docker images.

This task focuses on understanding the existing implementation. No unnecessary Dockerfile modifications are required.

## 2. Dockerfile Inventory

The repository contains three Dockerfiles:

| Dockerfile | Purpose |
|---|---|
| `docker/Dockerfile` | Shared Dockerfile for Spring PetClinic application services |
| `docker/grafana/Dockerfile` | Customises the Grafana monitoring image |
| `docker/prometheus/Dockerfile` | Customises the Prometheus monitoring image |

The eight application modules are:

- spring-petclinic-config-server
- spring-petclinic-discovery-server
- spring-petclinic-customers-service
- spring-petclinic-visits-service
- spring-petclinic-vets-service
- spring-petclinic-api-gateway
- spring-petclinic-admin-server
- spring-petclinic-genai-service

All eight modules configure the shared application Dockerfile through their Maven POM files.

## 3. Application Dockerfile Analysis

Location: `docker/Dockerfile`

### Builder stage

| Instruction | Purpose |
|---|---|
| `FROM eclipse-temurin:17 AS builder` | Starts the builder stage using Java 17 |
| `WORKDIR application` | Sets the working directory |
| `ARG ARTIFACT_NAME` | Accepts the application JAR name at build time |
| `COPY ${ARTIFACT_NAME}.jar application.jar` | Copies the JAR from the Docker build context |
| `RUN java -Djarmode=layertools -jar application.jar extract` | Extracts Spring Boot application layers |

### Runtime stage

| Instruction | Purpose |
|---|---|
| `FROM eclipse-temurin:17` | Starts a separate runtime image stage |
| `WORKDIR application` | Sets the runtime working directory |
| `ARG EXPOSED_PORT` | Accepts the port as a build argument |
| `EXPOSE ${EXPOSED_PORT}` | Documents the container port |
| `ENV SPRING_PROFILES_ACTIVE=docker` | Activates the Docker Spring profile |
| `COPY --from=builder application/dependencies/ ./` | Copies dependency layers |
| `RUN true` | Preserves a separate image layer |
| `COPY --from=builder application/spring-boot-loader/ ./` | Copies the Spring Boot loader |
| `RUN true` | Preserves a separate image layer |
| `COPY --from=builder application/snapshot-dependencies/ ./` | Copies snapshot dependency layers |
| `RUN true` | Preserves a separate image layer |
| `COPY --from=builder application/application/ ./` | Copies application classes and resources |
| `ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]` | Starts the Spring Boot application |

The Dockerfile uses a multi-stage build. The builder stage extracts the application layers, while the runtime stage receives the extracted layers and starts the application.

The final image does not need the original builder-stage JAR extraction environment.

## 4. Maven Integration

The root `pom.xml` configures the `exec-maven-plugin`.

The execution is named `docker-build` and is bound to the Maven `install` phase.

The inspected configuration establishes the following:

| Maven property | Configured value |
|---|---|
| `container.executable` | `docker` |
| `docker.image.prefix` | `springcommunity` |
| `docker.image.exposed.port` | `9090` |
| `container.platform` | `linux/amd64` |
| `container.build.extraarg` | `--load` |

The plugin invokes the Docker CLI with the following important arguments:

- `build`
- `-f Dockerfile`
- `--build-arg ARTIFACT_NAME=${project.build.finalName}`
- `--build-arg EXPOSED_PORT=${docker.image.exposed.port}`
- `--platform ${container.platform}`
- `${container.build.extraarg}`
- `-t ${docker.image.prefix}/${project.artifactId}`
- `${project.build.directory}`

The image tag is therefore constructed from the `springcommunity` prefix and the current Maven module's artifact ID.

The plugin uses the Maven module's build directory as the Docker build context.

## 5. Dockerfile Location and Build Context

Each of the eight application module POM files sets:

`${basedir}/../docker`

as the Dockerfile working directory.

This points to the shared `docker/` directory containing the application Dockerfile.

The Maven plugin separately passes:

`${project.build.directory}`

as the Docker build context. For these Maven modules, this is normally the module's `target/` directory.

This distinction is important:

- Dockerfile location: shared `docker/Dockerfile`.
- Docker build context: individual module's `target/` directory.
- JAR source: the JAR produced by Maven inside that module's build directory.

The `COPY` instruction can therefore find the JAR because it is inside the supplied build context.

## 6. Maven JAR Artifact Flow

The application packaging and image build process follows this sequence:

1. Maven builds an application module.
2. Maven produces the module's executable JAR in its `target/` directory.
3. The Docker build execution runs during the Maven `install` phase.
4. Maven passes the final JAR name through `ARTIFACT_NAME`.
5. Docker receives the module's `target/` directory as its build context.
6. The Dockerfile copies the JAR into the builder stage.
7. Spring Boot layers are extracted.
8. The runtime stage copies the extracted layers.
9. Docker creates an image tagged with the `springcommunity` prefix and module artifact ID.

The repository inspection identified built JARs for the eight application modules under their respective `target/` directories.

## 7. Grafana Dockerfile Analysis

Location: `docker/grafana/Dockerfile`

| Instruction | Purpose |
|---|---|
| `FROM grafana/grafana:11.4.0` | Uses the Grafana 11.4.0 base image |
| `ADD ./provisioning /etc/grafana/provisioning` | Adds Grafana provisioning configuration |
| `ADD ./grafana.ini /etc/grafana/grafana.ini` | Adds Grafana configuration |
| `ADD ./dashboards /var/lib/grafana/dashboards` | Adds dashboard definitions |

Compose specifies `./docker/grafana` as this service's build context.

The context contains the Dockerfile, provisioning directory, Grafana configuration and dashboard files.

Unlike the application Dockerfile, this Dockerfile does not package a Maven-built JAR.

## 8. Prometheus Dockerfile Analysis

Location: `docker/prometheus/Dockerfile`

| Instruction | Purpose |
|---|---|
| `FROM prom/prometheus:v3.1.0` | Uses the Prometheus 3.1.0 base image |
| `ADD prometheus.yml /etc/prometheus/` | Adds the Prometheus configuration |

Compose specifies `./docker/prometheus` as its build context.

The build context contains the Dockerfile and `prometheus.yml`.

This image customises an existing Prometheus image rather than packaging a Spring application.

## 9. Shared Patterns and Differences

### Shared patterns

- Dockerfiles use existing base images.
- Application services share one Dockerfile.
- The monitoring components have separate Dockerfiles and build contexts.
- Image creation is configured through Docker build instructions.

### Differences

| Area | Application services | Grafana | Prometheus |
|---|---|---|---|
| Dockerfile | Shared `docker/Dockerfile` | Dedicated | Dedicated |
| Build context | Module `target/` | `docker/grafana/` | `docker/prometheus/` |
| Maven JAR | Yes | No | No |
| Build stages | Multi-stage | Single stage | Single stage |
| Main purpose | Package and run Spring Boot services | Add dashboards and provisioning | Add monitoring configuration |

## 10. Findings and Verification Boundaries

The repository inspection confirms:

- Three Dockerfiles exist.
- Eight application modules use the shared application Dockerfile.
- Maven configuration defines the Docker build invocation.
- Application JARs exist in the respective Maven target directories.
- Grafana and Prometheus use separate Compose build contexts.
- The Docker build context and Dockerfile location are configured separately.

This task documents the existing build design. Inspection alone does not establish that a fresh Maven-driven image build has succeeded.

No Dockerfile changes were made as part of this analysis.

## 11. Evidence

Relevant screenshots are stored in:

`docs/project-documentation/images/docker/`

Evidence should show:

1. Dockerfile inventory.
2. Application Dockerfile and its two stages.
3. Maven Docker build plugin configuration.
4. Application module Dockerfile directory overrides.
5. Maven-built JAR inventory.
6. Grafana and Prometheus Dockerfiles and build-context contents.

Screenshots must show genuine repository or terminal output.

## 12. Jira Reference

**SPC-30 : Inspect and understand existing Dockerfiles**

Acceptance criteria:

- Relevant Dockerfiles and build contexts are documented.
- Image build approach and artifact flow are understood and recorded.
- No unnecessary Dockerfile changes are made without evidence.
