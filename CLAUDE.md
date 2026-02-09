# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

This is a **Maven parent POM** — it contains no application source code. It provides centralized build configuration, dependency management, and plugin setup for the Bankzy microservices ecosystem. Child projects (e.g., ClinicalsApi, Rewards) inherit this parent.

## Prerequisites

- **Java 25+**
- **Maven 3.9.12+** (provided via Maven Wrapper)

## Build Commands

```bash
# Build and install locally
./mvnw clean install

# Build skipping tests
./mvnw clean install -Dskip.unit.tests=true -Dskip.integration.tests=true

# Deploy to local Nexus
./mvnw clean deploy -Pnexus

# Deploy to PackageCloud (CI only, requires auth)
./mvnw clean deploy -Ppackagecloud -s .circleci/settings.xml

# Run with a custom version
./mvnw clean install -Drevision=2.0.0
```

Always use `./mvnw` (the wrapper), not a system-installed `mvn`.

## Architecture

### Versioning
Uses CI-friendly versioning via the `${revision}` property (default: `1.0.0-SNAPSHOT`). The `flatten-maven-plugin` resolves this at build time so published POMs have concrete versions.

### Dependency Management
- Imports `com.bankzy:bankzy-dependencies` BOM for shared dependency versions across the ecosystem
- Manages MapStruct and JUnit Jupiter versions directly

### Compiler Setup
Annotation processors are configured in order: **MapStruct** then **Lombok**. MapStruct uses `spring` as its default component model.

### Test Strategy
- **Unit tests**: Run via Surefire. Files matching `**/IT*.java` are excluded.
- **Integration tests**: Run via Failsafe. Files matching `IT*` pattern.
- **Coverage**: JaCoCo collects separate reports for UT (`target/coverage-reports/jacoco-ut.exec`) and IT (`target/coverage-reports/jacoco-it.exec`).
- **Static analysis**: SpotBugs with FindSecBugs runs during `verify` phase.

### Build Profiles
| Profile | Purpose | Activation |
|---------|---------|------------|
| `packagecloud` | Deploy to Buildkite-hosted PackageCloud repos | `-Ppackagecloud` (used in CI) |
| `nexus` | Deploy to local Nexus at `localhost:8081` | `-Pnexus` |

### CI/CD
CircleCI pipeline (`.circleci/config.yml`) builds on the `main` branch only, using `cimg/openjdk:25.0`. Auth tokens for PackageCloud are passed via `PCLOUD_RELEASE` and `PCLOUD_SNAPSHOT` environment variables.

## Key Properties

| Property | Default | Purpose |
|----------|---------|---------|
| `revision` | `1.0.0-SNAPSHOT` | CI-friendly project version |
| `skip.unit.tests` | `false` | Skip unit tests |
| `skip.integration.tests` | `false` | Skip integration tests |
| `java.version` | `25` | Java compiler target |

## Editing This POM

When modifying plugin versions or adding dependencies:
- Put version numbers in `<properties>` and reference them — don't hardcode versions in `<dependencyManagement>` or `<pluginManagement>`.
- New plugins go in `<pluginManagement>` for configuration, then must be activated in the `<plugins>` section below it.
- The enforcer plugin requires no duplicate dependency versions and minimum Java 25 / Maven 3.9.12.
