# Configuration & Externalized Settings Inventory

The configuration surface is compact and centered on one Spring Boot YAML file, Maven metadata filtering, and profile-based startup arguments. Profiles are simple, secrets are manual rather than managed through a vault, and there are no remote configuration services.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Main application config | YAML | `src/main/resources/application.yml` | Defines datasource, ports, security, project info, management port, and profile blocks |
| Maven build descriptor | XML | `pom.xml` | Supplies dependency versions and filtered project metadata consumed by `info.build.*` |
| Configuration properties binding | Annotation-driven bean | `src/main/java/com/khoubyari/example/service/ServiceProperties.java` | Binds `hotel.service.*` values into a Spring bean |
| README operational examples | Documentation | `README.md` | Documents optional `mysql` profile and supported startup arguments |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| default Maven build | implicit | Compiles Java 8 source and packages the application as a WAR | `maven-compiler-plugin 3.1`, `spring-boot-maven-plugin` |

No Maven `<profiles>` are declared in `pom.xml`.

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| default | implicit when no profile is passed | `application.yml` | H2 datasource, API port 8090, management port 8091, security disabled |
| test | `-Dspring.profiles.active=test` | `application.yml` test document | Enables H2 console, `spring.jpa.hibernate.ddl-auto=create-drop`, `logging.file=boot_example.log`, `hotel.service.name=test profile:` |
| mysql | `-Dspring.profiles.active=mysql` documented in README | documented append-only example in `README.md` | Switches datasource to MySQL and documents Hibernate `ddl-auto=update` |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `spring.jmx.enabled` | `false` | default, test | `application.yml` |
| `spring.datasource.driverClassName` | `org.h2.Driver` | default, test | `application.yml` |
| `spring.datasource.url` | `jdbc:h2:mem:bootexample;MODE=MySQL` | default, test | `application.yml` |
| `server.port` | `8090` | default, test | `application.yml` |
| `security.basic.enabled` | `false` | default, test | `application.yml` |
| `management.port` | `8091` | default, test | `application.yml` |
| `management.security.enabled` | `false` | default, test | `application.yml` |
| `project.name` | `spring-boot-rest-example` | default, test | `application.yml` with Maven filtering placeholders for info output |
| `project.version` | `0.1` | default, test | `application.yml` |
| `project.description` | `boot-example default description` | default, test | `application.yml` |
| `info.build.artifact` | `${project.artifactId}` | default, test | `application.yml` placeholder |
| `info.build.name` | `${project.name}` | default, test | `application.yml` placeholder |
| `info.build.description` | `${project.description}` | default, test | `application.yml` placeholder |
| `info.build.version` | `${project.version}` | default, test | `application.yml` placeholder |
| `hotel.service.name` | `default profile:` | default | `application.yml` |
| `spring.h2.console.enabled` | `true` | test | `application.yml` |
| `spring.jpa.hibernate.ddl-auto` | `create-drop` | test | `application.yml` |
| `logging.file` | `boot_example.log` | test | `application.yml` |
| `logging.org.hibernate` | `INFO` | test | `application.yml` |
| `spring.datasource.driverClassName` | `com.mysql.jdbc.Driver` | mysql documented in README | `README.md` |
| `spring.datasource.url` | `jdbc:mysql://<your_mysql_host_or_ip>/bootexample` | mysql documented in README | `README.md` |
| `spring.datasource.username` | `<your_mysql_username>` | mysql documented in README | `README.md` |
| `spring.datasource.password` | `[MASKED]` | mysql documented in README | `README.md` |
| `spring.jpa.hibernate.dialect` | `org.hibernate.dialect.MySQLInnoDBDialect` | mysql documented in README | `README.md` |
| `spring.jpa.hibernate.ddl-auto` | `update` | mysql documented in README | `README.md` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| spring-boot-rest-example | `-Dspring.profiles.active=test` or `-Dspring.profiles.active=mysql`; optional remote debug `-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005` | not specified | 1 |

No container memory, CPU, or autoscaling settings are defined in the repository.

## Startup Dependency Chain

1. `spring-boot-rest-example` starts directly with its embedded H2 datasource in the default and test profiles, so no separate infrastructure bootstrap sequence is required.
2. If the documented `mysql` profile is used, the application requires the target MySQL instance and credentials to be available before startup.
3. Actuator readiness is implicit through the running application; no custom wait scripts, Compose dependencies, or Kubernetes probes are defined in the repository.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `spring.datasource.username` | Database username | README placeholder |
| `spring.datasource.password` | Database password | `[MASKED]` placeholder in README example |

### Secrets Provisioning Workflow

The repository does not implement a secrets manager, vault integration, or encrypted property store. Secrets are expected to be supplied manually when using the documented MySQL profile, either by editing configuration values or providing equivalent runtime parameters outside the repository; the default H2-based profiles do not require any secret material.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | N/A | N/A |

No `@ConditionalOnProperty`, external feature-flag framework, or custom toggle configuration was found.

## Framework & Runtime Versions

| Component | Version | Source |
|---|---:|---|
| Spring Boot parent | `1.5.9.RELEASE` | `pom.xml` |
| Java source and target | `1.8` | `pom.xml` |
| Maven compiler plugin | `3.1` | `pom.xml` |
| Springfox Swagger | `2.5.0` | `pom.xml` |
| H2 Database | `1.4.193` | `pom.xml` |
| Jackson Core ASL | `1.9.13` | `pom.xml` |
| JSON Path | `2.4.0` | `pom.xml` |
| Build tool expectation | `Maven 3.x` | `README.md` |
| Runtime packaging | executable WAR with embedded Tomcat 8 | `README.md` |
