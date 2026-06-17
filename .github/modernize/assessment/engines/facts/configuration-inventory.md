# Configuration & Externalized Settings Inventory

The application uses a single YAML configuration file (`application.yml`) with one embedded runtime profile (`test`). There are no external config servers, secret stores, or environment variable references.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|--------|------|--------------|-------|
| application.yml | Spring Boot YAML | src/main/resources/application.yml | Primary config file; contains default and `test` profile blocks |
| pom.xml | Maven build descriptor | pom.xml | Declares Spring Boot parent BOM 1.5.9, Java 1.8 source/target, plugins |

No `bootstrap.yml`, `bootstrap.properties`, Spring Cloud Config server, Vault, KeyVault, AWS Secrets Manager, Consul KV, Kubernetes ConfigMaps, or Docker Compose files are present.

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---------|-----------|---------|--------------------------|
| (default) | Always active | Standard build, WAR packaging | spring-boot-maven-plugin, maven-compiler-plugin (Java 1.8) |

No Maven `<profiles>` blocks are declared in `pom.xml`. There is only one build configuration. No `buildDocker`, `dev`, `cloud`, or other conditional Maven profiles exist. The `<packaging>war</packaging>` is always active, and the Spring Boot Maven plugin always executes.

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---------|-----------------|-------------|---------------|
| default | Active when no profile is set | application.yml (top block) | H2 in-memory DB, server port 8090, security disabled, management port 8091 |
| test | `spring.profiles: test` in application.yml | application.yml (embedded `---` block) | H2 console enabled, JPA DDL `create-drop`, logging to file, hotel.service.name override |

The `test` profile is embedded in the same `application.yml` using YAML document separators (`---`). No separate `application-test.yml`, `application-prod.yml`, or `application-dev.yml` files exist. Profile activation is expected to be set via `spring.profiles.active=test` at runtime (e.g., passed as a system property).

## Properties Inventory

### spring-boot-rest-example

| Property Key | Default | Profile | Source |
|-------------|---------|---------|--------|
| spring.jmx.enabled | false | all | application.yml |
| spring.datasource.driverClassName | org.h2.Driver | all | application.yml |
| spring.datasource.url | jdbc:h2:mem:bootexample;MODE=MySQL | all | application.yml |
| server.port | 8090 | all | application.yml |
| security.basic.enabled | false | all | application.yml |
| management.port | 8091 | all | application.yml |
| management.security.enabled | false | all | application.yml |
| project.name | spring-boot-rest-example | all | application.yml |
| project.version | 0.1 | all | application.yml |
| project.description | boot-example default description | all | application.yml |
| info.build.artifact | ${project.artifactId} | all | application.yml (pom placeholder) |
| info.build.name | ${project.name} | all | application.yml (pom placeholder) |
| info.build.description | ${project.description} | all | application.yml (pom placeholder) |
| info.build.version | ${project.version} | all | application.yml (pom placeholder) |
| hotel.service.name | 'default profile:' | default | application.yml |
| spring.h2.console.enabled | true | test | application.yml |
| spring.jpa.hibernate.ddl-auto | create-drop | test | application.yml |
| hotel.service.name | 'test profile:' | test | application.yml |
| logging.file | boot_example.log | test | application.yml |
| logging.org.hibernate | INFO | test | application.yml |

No environment variable references (`${ENV_VAR}`) are used. All values are hardcoded in the YAML file.

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---------|---------------------|--------|---------------|
| spring-boot-rest-example | None declared | Not specified | 1 (no scaling config) |

No Dockerfile, `docker-compose.yml`, Kubernetes manifests, or JVM heap settings (`-Xms`, `-Xmx`) are present in the repository. There are no container `mem_limit`, CPU quotas, or cloud deployment configurations. The application is intended to run as a standalone Spring Boot WAR or embedded Tomcat process with default JVM settings.

## Startup Dependency Chain

The application is a standalone single-service monolith with no external service dependencies at startup.

1. **spring-boot-rest-example** starts directly — no upstream config server, discovery server, or database server to wait for (H2 is embedded and starts in-process).

No `dockerize` wait-for-TCP scripts, Kubernetes readiness probes, Docker Compose `depends_on`, or Spring Cloud Config retry mechanisms are configured.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage |
|-----------------|------|---------|
| spring.datasource.url | Database connection string | application.yml (hardcoded, no credentials; H2 in-memory) |
| security.basic.enabled | Security toggle | application.yml (set to `false`; no password configured) |

No actual secrets are present. The H2 in-memory connection URL requires no username/password (H2 defaults to user `sa` with empty password). No API keys, OAuth2 client secrets, JWT signing keys, or external service credentials exist anywhere in the codebase or configuration files.

### Secrets Provisioning Workflow

No secrets provisioning workflow exists. The application contains no sensitive credentials — the in-memory H2 database uses the H2 default anonymous access, and HTTP Basic authentication is explicitly disabled. There is no integration with HashiCorp Vault, Azure KeyVault, AWS Secrets Manager, Jasypt encrypted properties, or GitHub Actions secrets. All configuration is committed to source control in plaintext.

## Feature Flags

No feature flag framework is used. No LaunchDarkly, Unleash, Spring `@ConditionalOnProperty`, or custom toggle configuration is present. The closest analog is the `security.basic.enabled=false` property, which acts as a manual toggle to disable HTTP Basic authentication — but this is a standard Spring Boot security property, not a feature flag system.

| Flag Name | Default | Controlled By |
|-----------|---------|--------------|
| security.basic.enabled | false | application.yml (hardcoded) |
| spring.h2.console.enabled | false (true in test) | application.yml test profile |
| management.security.enabled | false | application.yml (hardcoded) |

## Framework & Runtime Versions

| Component | Version | Source |
|-----------|---------|--------|
| Java (source/target) | 1.8 | pom.xml maven-compiler-plugin |
| Spring Boot | 1.5.9.RELEASE | pom.xml parent BOM |
| Spring Framework | 4.3.x (via Boot BOM) | Transitive via spring-boot-starter-parent |
| Spring Security | 4.2.x (via Boot BOM) | Transitive via spring-boot-starter-security |
| Spring Data JPA | 1.11.x (via Boot BOM) | Transitive via spring-boot-starter-data-jpa |
| Hibernate ORM | 5.0.x (via Boot BOM) | Transitive via spring-boot-starter-data-jpa |
| Embedded Tomcat | 8.5.x (via Boot BOM) | Transitive via spring-boot-starter-tomcat |
| H2 Database | 1.4.193 | pom.xml (explicit) |
| HSQLDB | Boot-managed (runtime) | pom.xml (explicit, runtime scope) |
| Springfox Swagger2 | 2.5.0 | pom.xml (explicit) |
| Springfox Swagger UI | 2.5.0 | pom.xml (explicit) |
| Jackson Core ASL (1.x) | 1.9.13 | pom.xml (explicit) |
| JsonPath | 2.4.0 | pom.xml (explicit) |
| JAXB API | 2.3.0 | pom.xml (explicit) |
| Maven | 3.x (assumed) | Build environment |
| Spring Boot Maven Plugin | 1.5.9.RELEASE (via BOM) | pom.xml |
| Maven Compiler Plugin | 3.1 | pom.xml |
