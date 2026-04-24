# Testing Spring PetClinic

## Prerequisites
- Java 21+ (LTS) installed with `JAVA_HOME` set to the JDK path
- Docker (optional, needed for MySQL/PostgreSQL Testcontainer-based tests)

## Devin Secrets Needed
None for default H2 profile. MySQL/PostgreSQL profiles would need database credentials if testing against external DBs.

## Build & Test Commands

```bash
# Set Java home (adjust path if needed)
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64

# Compile only
./mvnw compile -DskipTests

# Run all tests (59 tests, ~2 min)
./mvnw test

# Full verify (compile + test + checkstyle + format validation)
./mvnw -B verify

# Start the app locally (H2 in-memory DB)
./mvnw spring-boot:run
# App serves on http://localhost:8080/
```

## Key Actuator Endpoints
All actuator endpoints are exposed (`management.endpoints.web.exposure.include=*`):
- `/actuator/info` — Build info including `java.source` and `java.target` versions (from spring-boot-maven-plugin build-info config)
- `/actuator/health` — Health check
- `/actuator/env` — Environment properties including JVM version

## Key UI Test Flows

### Home Page
- URL: `http://localhost:8080/`
- Expect: "Welcome" heading, pet image, nav bar with HOME, FIND OWNERS, VETERINARIANS, ERROR

### Find Owners (JPA + Thymeleaf)
- URL: `http://localhost:8080/owners/find`
- Click "Find Owner" with empty search → lists all seeded owners
- George Franklin: Address "110 W. Liberty St.", City "Madison", Tel "6085551023", Pet "Leo" (cat)
- Owner detail page at `/owners/1` shows Pets and Visits section

### Veterinarians (many-to-many JPA)
- URL: `http://localhost:8080/vets.html`
- James Carter → none (no specialties)
- Helen Leary → radiology
- Linda Douglas → dentistry surgery

## Database Profiles
- Default: H2 in-memory (no setup needed, seeded with sample data)
- MySQL: `./mvnw spring-boot:run -Dspring-boot.run.profiles=mysql` (needs MySQL running)
- PostgreSQL: `./mvnw spring-boot:run -Dspring-boot.run.profiles=postgres` (needs PostgreSQL running)

## CI Workflows
- `maven-build.yml` — Builds and verifies with Maven + Java 21
- `gradle-build.yml` — Builds with Gradle + Java 21
- `deploy-and-test-cluster.yml` — K8s cluster deploy test (only triggers on `k8s/` path changes)

## Tips
- The app uses Spring Boot 4.0.x with Spring Framework 7.x
- Checkstyle v13+ requires JDK 21+ — if downgrading Java, checkstyle must also be downgraded
- The `spring-javaformat-maven-plugin` enforces Spring code formatting during `validate` phase
- Testcontainer-based tests (MySQL) need Docker running; they may be skipped if Docker is unavailable
- The H2 console is available at `http://localhost:8080/h2-console` with JDBC URL printed at startup
