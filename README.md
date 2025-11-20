# Spring Security OAuth2 Authorities Company Structure

A Spring Boot 3.5.6 application demonstrating OAuth2 authentication with company structure hierarchy and role-based
access control.

## Features

- Spring Security OAuth2 implementation
- Company structure hierarchy (Company → Department → Office → Employee)
- Role-based access control with authorities
- JPA/Hibernate with Flyway migrations
- H2 in-memory database (default) with PostgreSQL support
- RESTful API endpoints
- H2 Console for database management

## Technologies

- Java 25
- Spring Boot 3.5.6
- Spring Security OAuth2
- Spring Data JPA
- Hibernate 6
- Flyway Database Migrations
- H2 Database (in-memory)
- PostgreSQL (optional)
- Maven
- Lombok

## Prerequisites

- JDK 25 or compatible version
- Maven 3.x
- (Optional) Docker for PostgreSQL

## Quick Start

### 1. Clone the repository

```bash
git clone <repository-url>
cd spring-security-oauth2-authorities-company-structure
```

### 2. Build the project

```bash
mvn clean package
```

### 3. Run the application

```bash
mvn spring-boot:run
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

The application uses **Spring Security 6** (Spring Boot 3 compatible) with OAuth2 and modern security patterns.

### Security Features

- **Spring Security 6**: Component-based configuration (SecurityFilterChain)
- **OAuth2 Authentication**: Token-based authentication support
- **Role-Based Access Control**: Authority-based permissions
- **HTTP Basic Authentication**: Enabled for API testing
- **Password Encoders**:
    - **userPasswordEncoder** (Primary): BCrypt with strength 8 for user passwords
    - **oauthClientPasswordEncoder**: BCrypt with strength 4 for OAuth clients

### Security Configuration Classes

- `ServerSecurityConfig`: Main security filter chain configuration
- `Encoders`: Password encoder beans configuration
- `@Dev`: Development profile annotation

### Default Credentials

Spring Security generates a random password on startup. Check the console output:

```
Using generated security password: <random-password>
```

**Username**: `user`
**Password**: `<check console logs for generated password>`

### Predefined Users

The application seeds test users via Flyway migrations. Check:

- `src/main/resources/db/migration/h2/V14__insert_user.sql`
- `src/main/resources/db/migration/h2/V13__insert_authorities.sql`
- `src/main/resources/db/migration/h2/V12__insert_authentication.sql`

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
- **Users & Authorities**: OAuth2 authentication

## API Endpoints

All endpoints require authentication.

Example endpoints (based on the project structure):

- `GET /api/companies` - List all companies
- `GET /api/departments` - List all departments
- `GET /api/offices` - List all offices
- `GET /api/employees` - List all employees

## Development

### Enabling DevTools

Spring Boot DevTools is included for hot reload during development.

### H2 Console Access

When running with H2 profile, access the database console at:

`http://localhost:8080/h2-console`

### Running Tests

```bash
mvn test
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
│       ├── application-dev.properties
│       ├── application-h2.properties
│       └── application-postgres.properties
└── test/
```

## Troubleshooting

### Application fails to start

1. Ensure JDK 25 is installed: `java -version`
2. Clean and rebuild: `mvn clean package`
3. Check port 8080 is available: `lsof -i :8080`

### Database connection issues

- For H2: Database is in-memory; no external setup needed
- For PostgreSQL: Ensure PostgreSQL is running on port 5433

### Migration conflicts

If you encounter Flyway migration conflicts:

```bash
mvn clean
mvn flyway:clean
mvn spring-boot:run
```

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

**Spring Boot 3 Migration** - Fully updated and tested

#### Major Updates:

- ✅ **Spring Boot 3.5.6**: Complete upgrade from Spring Boot 2.x
- ✅ **Spring Security 6**: Migrated to component-based configuration
    - Removed deprecated `WebSecurityConfigurerAdapter`
    - Implemented `SecurityFilterChain` pattern
    - Updated `AuthenticationManager` configuration
- ✅ **Hibernate 6**: Full compatibility
    - Updated to `Hibernate6Module` for Jackson
    - Changed to `org.hibernate.orm` group ID
    - Updated PostgreSQL dialect
- ✅ **Jakarta EE**: Migrated from javax.* to jakarta.* packages
- ✅ **H2 Database**: In-memory database as default (easily switchable to PostgreSQL)
- ✅ **Flyway Migrations**: Updated to Spring Boot 3 property format
- ✅ **OAuth2 Authentication**: Token-based authentication and authorization
- ✅ **Company Structure Hierarchy**: Multi-level organizational structure
- ✅ **Password Encoders**: Dual BCrypt encoders for users and OAuth clients

#### Bug Fixes:

- Fixed missing package declarations
- Fixed missing import statements
- Fixed duplicate dependencies
- Fixed main method visibility
- Fixed SQL syntax for H2 compatibility (removed MySQL-specific syntax)
- Fixed Flyway migration path conflicts
- Fixed PasswordEncoder bean conflicts with `@Primary` annotation
- Fixed Docker Compose integration issues

#### Configuration Updates:

- Application properties updated to Spring Boot 3 format
- Security configuration modernized for Spring Security 6
- Database configurations separated by profile (H2/PostgreSQL)
- DevTools auto-reload enabled for development
