# Helios Solar Monitoring System

Helios is a Java web application for monitoring residential solar installations. It provides homeowner and administrator dashboards for solar production, household consumption, battery storage, registered devices, and system alerts. A background simulator supplies changing telemetry for local demonstrations.

## Features

- Homeowner dashboards for production, consumption, battery status, alerts, and profile settings.
- Administrator tools for users, devices, and system-wide settings.
- Session-based authentication with role-based access control and CSRF protection.
- A JSON telemetry endpoint used by the live dashboard charts.
- MySQL persistence through JDBC and HikariCP, with an in-memory H2 fallback for local development.
- Responsive JSP interface with vanilla JavaScript and Chart.js.

## Technology

| Component | Technology |
| --- | --- |
| Language | Java 17 |
| Web layer | Jakarta Servlets 6, JSP 3.1, JSTL 3 |
| Local web server | Embedded Apache Tomcat 10.1 |
| Database | MySQL 8+ or the local H2 fallback |
| Connection pool | HikariCP |
| Build | Apache Maven 3.8+ |

## Requirements

- JDK 17 or newer
- Apache Maven 3.8 or newer
- MySQL 8 or newer for persistent storage (optional for a local H2-backed demo)

Check that Java and Maven are available:

```sh
java -version
mvn -version
```

## Run locally

From the project root, compile and start the embedded Tomcat server:

```sh
mvn clean compile
mvn exec:java
```

Open [http://localhost:8080](http://localhost:8080). To use another port, set the `PORT` environment variable before starting the application.

The application first attempts to connect to the MySQL database configured in `src/main/resources/db.properties`. If MySQL is unavailable, it attempts to use an in-memory H2 database. The H2 database is temporary and is cleared when the application stops.

## MySQL configuration

1. Create the database and tables, then load the sample records. From a MySQL client, run:

   ```sql
   source database/schema.sql;
   source database/seed.sql;
   ```

2. Configure `src/main/resources/db.properties` for your local MySQL instance. For example:

   ```properties
   db.driver=com.mysql.cj.jdbc.Driver
   db.url=jdbc:mysql://localhost:3306/solar_db?createDatabaseIfNotExist=true&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
   db.username=root
   db.password=your-local-mysql-password
   ```

   Do not commit real database passwords. Set `db.password` to match your local MySQL setup.

The application can bootstrap an empty database from the included SQL scripts when they are available to the running application. If automatic setup is unavailable, apply `database/schema.sql` and `database/seed.sql` manually as shown above.

## Demo accounts

The seed script creates the following accounts for local demonstration:

| Role | Email | Password |
| --- | --- | --- |
| Administrator | `admin@solar.local` | `admin123` |
| Homeowner | `alice@solar.local` | `user123` |
| Homeowner | `bob@solar.local` | `user123` |
| Homeowner | `charlie@solar.local` | `user123` |

These are public sample credentials, not suitable for a deployed or production system. Change or remove them before exposing an instance to other users.

## Build a WAR

Create a deployable Web Application Archive:

```sh
mvn clean package
```

The artifact is written to `target/solar-monitoring-system.war`. Deploy it to a compatible Jakarta EE 10 / Tomcat 10.1 server. Configure its database and ensure the schema and seed data have been installed in MySQL before use.

## Telemetry API

The application exposes one JSON endpoint:

| Method | Path | Description |
| --- | --- | --- |
| `GET` | `/api/telemetry` | Returns dashboard summary metrics, today's hourly readings, unread-alert count, and battery status when available. |

Optional query parameters:

| Parameter | Description |
| --- | --- |
| `ownerId` | Select a homeowner's dashboard data when no homeowner session determines the owner. |
| `deviceId` | Limit hourly telemetry points to a device. |

Example:

```sh
curl http://localhost:8080/api/telemetry
```

## Project layout

```text
database/
  schema.sql                 MySQL schema
  seed.sql                   Demo users, devices, settings, and readings
src/main/java/com/solar/
  dao/                        Data access interfaces and implementations
  enums/                      Roles, device types/statuses, and alert severity
  filter/                     Authentication, role, CSRF, and encoding filters
  listener/                   Application lifecycle and simulator startup
  model/                      Domain models
  service/                    Business logic
  servlet/                    MVC controllers and telemetry API
  simulator/                  Background telemetry simulator
  util/                       Database, JSON, validation, and utility classes
src/main/resources/            Database and logging configuration
src/main/webapp/
  WEB-INF/views/               JSP views for authentication, homeowners, and admins
  assets/                      CSS and JavaScript
  index.jsp                    Application entry point
pom.xml                         Maven configuration
```

## License

This project is distributed under the MIT License. See [LICENSE](LICENSE).
