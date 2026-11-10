# Copilot instructions for bamboo

## Project shape

This repository is a Gradle multi-project Java/Spring Boot workspace. The root project mostly configures shared Gradle settings, while the runnable application lives in `hr/`.

- Root `build.gradle` sets the shared group/version and applies the Java + Checkstyle settings to subprojects.
- `hr/build.gradle` is the app module and contains the Spring Boot dependencies, JPA, Redis, Kafka, Security, and Testcontainers setup.
- `hr/src/main/java/com/bamboo/hr/` holds the Spring Boot entry point and package-level config.
- `config/checkstyle/checkstyle.xml` is the source of truth for Java style checks; the build enforces it via Gradle.
- The app is still in an early scaffold state; most code is minimal placeholders and configuration, not a large existing domain model.

## Build, test, and lint commands

Run Gradle from the `hr` module, not the repo root. The root project is only a parent config and the root-level `./gradlew test` currently fails because the app plugin configuration is applied in the module instead of the root build.

```bash
cd hr
./gradlew test
./gradlew test --tests com.bamboo.hr.HrApplicationTests
./gradlew checkstyleMain checkstyleTest
./gradlew bootRun
```

Useful notes:

- `test` is configured with JUnit Platform.
- `checkstyleMain` and `checkstyleTest` are the standard lint/style checks enforced by the project.
- The build targets Java 25 via the Gradle toolchain, so local JDK compatibility matters.

## High-level architecture

The application is a single Spring Boot service named `hr`.

- `HrApplication` is the application entry point and enables JPA repositories under `com.bamboo.hr.repository`.
- `SecurityConfiguration` defines the app’s security model: HTTPS basic auth, a public `/public/**` path, and an in-memory `admin/admin` user for authenticated access.
- `application.properties` is intentionally minimal and currently only sets `spring.application.name=hr`.
- `TestcontainersConfiguration` wires PostgreSQL, Kafka, and Redis containers into the test context for `@SpringBootTest` scenarios.
- The project uses Spring Boot starters for web, JPA, Redis, Kafka, security, validation, actuator, and caching, so domain work should usually follow the Spring idioms already present in the scaffold.

## Key conventions in this codebase

- Keep Gradle commands scoped to the `hr` subproject unless you are intentionally changing root-level shared configuration.
- Follow the Spring Boot package layout under `com.bamboo.hr` rather than creating ad hoc top-level packages.
- Prefer package-level configuration classes (for example, `config` and the application entry point) over broad, cross-cutting manual wiring.
- Respect the repo’s Checkstyle rules: no star imports, braces required for blocks, whitespace consistency, and no trailing/unused imports.
- The default Java toolchain is Java 25; new code should remain compatible with that target unless the build is intentionally upgraded.
- Test scaffolding uses `@SpringBootTest` plus Testcontainers, so new integration tests should follow that pattern when they require database/Kafka/Redis-backed behavior.

## Repo-specific notes

- The project has a generated Spring Boot `HELP.md` in `hr/`, but the actual project guidance here is the build and code layout in Gradle and the app sources.
- `README.md` is minimal and does not add project-specific workflow instructions beyond the generated Spring Boot defaults.
- There is no existing `.github/copilot-instructions.md`, `CLAUDE.md`, `AGENTS.md`, or similar repo-level AI instruction file to extend; this file is the starting point for contributors and Copilot sessions.
