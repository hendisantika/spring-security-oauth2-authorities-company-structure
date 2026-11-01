# Spring Security OAuth2 Authorities Company Structure

[![Java CI with Maven](https://github.com/hendisantika/spring-security-oauth2-authorities-company-structure/actions/workflows/maven.yml/badge.svg)](https://github.com/hendisantika/spring-security-oauth2-authorities-company-structure/actions/workflows/maven.yml)
![Java](https://img.shields.io/badge/Java-25-orange?logo=openjdk)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.1.2-brightgreen?logo=springboot)
[![Dependabot](https://img.shields.io/badge/Dependabot-enabled-blue?logo=dependabot)](.github/dependabot.yml)

A Spring Boot 4.1 application (running on JDK 25) demonstrating authority-based access control over a company
structure hierarchy.

## Features

- Spring Security 7 with authority-based permissions
- Company structure hierarchy (Company → Department → Office → Employee)
- JPA/Hibernate 7 with Flyway migrations
- H2 in-memory database (default) with PostgreSQL support
- RESTful API endpoints (Spring MVC + HATEOAS links)
- H2 Console for database management

## Technologies

| Component       | Version / Notes                                        |
|-----------------|--------------------------------------------------------|
| Java            | **25** (Temurin in CI)                                 |
| Spring Boot     | **4.1.2**                                              |
| Spring Security | 7.x (`SecurityFilterChain`, HTTP Basic)                |
| Spring Data JPA | Hibernate ORM 7.x (+ `hibernate-processor`)            |
| Jackson         | 3.x (`tools.jackson`) with `jackson-datatype-hibernate7` |
| Flyway          | 12.x via `spring-boot-starter-flyway`                  |
| Database        | H2 2.4 (default), PostgreSQL 42.7 driver (optional)    |
| Build           | Maven Wrapper (Maven 3.9.11), Lombok                   |

## Prerequisites

- **JDK 25** (`java -version` must report 25)
- No local Maven needed — use the bundled Maven Wrapper (`./mvnw`)
- (Optional) Docker for PostgreSQL

## Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/hendisantika/spring-security-oauth2-authorities-company-structure.git
cd spring-security-oauth2-authorities-company-structure
```

### 2. Build & test

```bash
./mvnw clean verify
```

### 3. Run the application

```bash
./mvnw spring-boot:run
# or
java -jar target/company-structure-0.0.1-SNAPSHOT.jar
```

The application will start on `http://localhost:8080`

## Configuration

### Active Profiles

By default, the application runs with `dev` and `h2` profiles.

To change profiles, edit `src/main/resources/application.properties`:

```properties
spring.profiles.active=dev,h2
```

### Database Configuration

#### H2 (Default)

The application uses H2 in-memory database by default. Configuration is in `application-h2.properties`:

```properties
spring.datasource.url=jdbc:h2:mem:company-structure-spring-security-oauth2-authorities
spring.datasource.username=yu71
spring.datasource.password=53cret
```

**H2 Console**: `http://localhost:8080/h2-console`

- JDBC URL: `jdbc:h2:mem:company-structure-spring-security-oauth2-authorities`
- Username: `yu71`
- Password: `53cret`

#### PostgreSQL

To use PostgreSQL:

1. Update `application.properties`:
   ```properties
   spring.profiles.active=dev,postgres
   ```

2. Ensure PostgreSQL is running on port 5433:
   ```bash
   docker run -d \
     --name company-structure-db \
     -e POSTGRES_DB=company-structure-spring-security-oauth2-authorities \
     -e POSTGRES_USER=yu71 \
     -e POSTGRES_PASSWORD=53cret \
     -p 5433:5432 \
     postgres:latest
   ```

## Security Configuration

The application uses **Spring Security 7** (Spring Boot 4) with a component-based `SecurityFilterChain`.

### Security Features

- **Spring Security 7**: Component-based configuration (`SecurityFilterChain`)
- **Database-backed users**: `UserDetailsServiceImpl` loads users & authorities via JPA
- **Role-Based Access Control**: Authority-based permissions
- **HTTP Basic Authentication**: Enabled for API testing
- **Password Encoders**:
    - **userPasswordEncoder** (Primary): BCrypt with strength 8 for user passwords
    - **oauthClientPasswordEncoder**: BCrypt with strength 4 for OAuth clients

### Security Configuration Classes

- `ServerSecurityConfig`: Main security filter chain configuration
- `Encoders`: Password encoder beans configuration
- `@Dev`: Development profile annotation

### Predefined Users

Users are seeded by Flyway (`V11`–`V14`) and authenticate with **HTTP Basic**:

| Username   | Password      | Authorities                                        |
|------------|---------------|----------------------------------------------------|
| `admin`    | `admin1234`   | all `COMPANY_*`, all `DEPARTMENT_*`, `ROLE_COMPANY_READER` |
| `reader`   | `reader1234`  | `COMPANY_READ`, `DEPARTMENT_READ`                  |
| `modifier` | `modifier1234`| `COMPANY_UPDATE`, `DEPARTMENT_UPDATE`              |
| `reader2`  | `reader1234`  | `ROLE_COMPANY_READER`                              |

Legacy OAuth2 client rows (`OAUTH_CLIENT_DETAILS`) are still seeded in `V12`, but the old
`spring-security-oauth2` (2.x) authorization server has been removed because it is end-of-life and incompatible
with Spring Boot 4. The previous configuration is kept as `*.java.bak` files for reference.

### H2 Console Access

The H2 console is configured with:

- **CSRF Protection**: Disabled for H2 console endpoints
- **Frame Options**: Set to `SAMEORIGIN` to allow H2 console frames
- **URL**: `http://localhost:8080/h2-console`

## Database Schema

The application manages the following entities:

- **Company**: Root level organization
- **Department**: Belongs to a Company
- **Office**: Belongs to a Department
- **Employee**: Belongs to an Office
- **Car**: Company assets
- **Address**: Location information
- **Users & Authorities**: authentication & authorization data

## API Endpoints

All endpoints live under `/secured/company` and require authentication.

| Method   | Path                              | Required authority                      |
|----------|-----------------------------------|-----------------------------------------|
| `GET`    | `/secured/company`                | `COMPANY_READ`                          |
| `GET`    | `/secured/company/{id}`           | `COMPANY_READ` and `DEPARTMENT_READ`    |
| `GET`    | `/secured/company/filter?name=..` | `COMPANY_READ` and `DEPARTMENT_READ`    |
| `POST`   | `/secured/company`                | `COMPANY_CREATE`                        |
| `PUT`    | `/secured/company`                | `COMPANY_UPDATE`                        |
| `DELETE` | `/secured/company/{id}`           | `COMPANY_DELETE`                        |

Example:

```bash
curl -u admin:admin1234 http://localhost:8080/secured/company
```

## Development

### Enabling DevTools

Spring Boot DevTools is included for hot reload during development.

### H2 Console Access

When running with H2 profile, access the database console at:

`http://localhost:8080/h2-console`

### Running Tests

```bash
./mvnw test
```

## Database Migrations

Flyway manages database migrations. Migration scripts are located in:

- H2: `src/main/resources/db/migration/h2/`
- PostgreSQL: `src/main/resources/db/migration/postgresql/`

Migration order:

1. V1: Create hibernate sequence
2. V2: Create address table
3. V3: Create company table
4. V4: Create car table
5. V5: Create department table
6. V6: Create employee table
7. V7: Insert car data
8. V8: Create office table
9. V9: Insert office data
10. V10: Create OAuth2 tables
11. V11: Create user table
12. V12: Insert authentication data
13. V13: Insert authorities data
14. V14: Insert user data

## Project Structure

```
src/
├── main/
│   ├── java/
│   │   └── id/my/hendisantika/companystructure/
│   │       ├── config/
│   │       │   ├── encryption/
│   │       │   │   └── Encoders.java
│   │       │   ├── profile/
│   │       │   │   └── annotation/
│   │       │   │       └── Dev.java
│   │       │   └── HibernateConfiguration.java
│   │       ├── controller/
│   │       ├── model/
│   │       ├── repository/
│   │       ├── service/
│   │       └── SpringSecurityOauth2AuthoritiesCompanyStructureApplication.java
│   └── resources/
│       ├── db/migration/
│       │   ├── h2/
│       │   └── postgresql/
│       ├── application.properties
│       ├── application-default.properties
│       ├── application-h2.properties
│       └── application-postgres.properties
└── test/
```

## Troubleshooting

### Application fails to start

1. Ensure JDK 25 is installed: `java -version`
2. Clean and rebuild: `./mvnw clean verify`
3. Check port 8080 is available: `lsof -i :8080`

### Database connection issues

- For H2: Database is in-memory; no external setup needed
- For PostgreSQL: Ensure PostgreSQL is running on port 5433

### Migration conflicts

If you encounter Flyway migration conflicts:

- H2 (default) is in-memory, so simply restart the application.
- For PostgreSQL, drop/recreate the database (or the `flyway_schema_history` table) and restart.

## CI/CD & Dependency Management

### GitHub Actions

[`.github/workflows/maven.yml`](.github/workflows/maven.yml) runs on every push / pull request to `main`
(and manually via *workflow_dispatch*):

- **Build & Test (JDK 25)** — Temurin JDK 25, Maven cache, `./mvnw -B -ntp verify`; Surefire reports are uploaded
  as an artifact when the build fails.
- **Submit dependency graph** — only on pushes to `main`; feeds the Maven dependency graph to GitHub so Dependabot
  alerts are accurate.
- Least-privilege `permissions` and `concurrency` (older runs of the same ref are cancelled).

### Dependabot

[`.github/dependabot.yml`](.github/dependabot.yml) checks **Maven** and **GitHub Actions** every Monday at 05:00
(Asia/Jakarta):

- Grouped PRs: `spring-boot`, `persistence` (Hibernate/Flyway/H2/PostgreSQL), `jackson`, `maven-plugins`,
  `minor-and-patch`, and a single `github-actions` group.
- Conventional commit messages (`build(deps): ...`, `build(deps-dev): ...`, `ci(deps): ...`), labels and assignee.
- Major Spring Boot parent bumps are ignored and handled manually.

## Contributing

1. Fork the repository
2. Create your feature branch: `git checkout -b feature/my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin feature/my-new-feature`
5. Submit a pull request

## License

[Specify your license here]

## Contact

- **Author**: hendisantika
- **Email**: hendisantika@yahoo.co.id
- **Telegram**: @hendisantika34
- **Link**: s.id/hendisantika

## Version History

### Current Version (0.0.1-SNAPSHOT)

**Spring Boot 4.1 on JDK 25**

- ✅ **Spring Boot 4.1.2** with modular starters (`spring-boot-starter-webmvc`, `spring-boot-starter-flyway`,
  `spring-boot-h2console`)
- ✅ **JDK 25** for local builds and CI
- ✅ **Spring Security 7**: `DaoAuthenticationProvider(UserDetailsService)` constructor API
- ✅ **Hibernate 7**: `hibernate-jpamodelgen` replaced by the `hibernate-processor` annotation processor
- ✅ **Jackson 3**: `Hibernate6Module` replaced by `tools.jackson` `Hibernate7Module`
- ✅ Removed end-of-life `spring-security-oauth2` 2.5.2 (only referenced by `.bak` files)
- ✅ Removed obsolete Hibernate properties (`hibernate.dialect`, `hibernate.id.new_generator_mappings`)
- ✅ Hardened GitHub Actions workflow and grouped Dependabot updates

### Previous: Spring Boot 3 Migration

- Spring Boot 3.5.x, Spring Security 6 (`SecurityFilterChain`), Hibernate 6, Jakarta EE (`jakarta.*`)
- H2 as default database, Flyway path fixes, dual BCrypt password encoders
