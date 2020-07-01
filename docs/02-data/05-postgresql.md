---
layout: default
title: PostgreSQL
parent: Spring Data
nav_order: 5
permalink: docs/data/postgresql/
---

# PostgreSQL
{: .no_toc }

So far we used an [H2](https://www.h2database.com/) in-memory database.  While this database is good for development and prototyping, it's rarely used for production.  In this page we will configure our application to make use of the [PostgreSQL](https://www.postgresql.org/) instead.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## PostgreSQL

"_PostgreSQL is a very powerful, open source object-relational database system with over 30 years of active development that has earned it a strong reputation for reliability, feature robustness, and performance._"<br />
([reference](https://www.postgresql.org/about/))

## PgAdmin

[PgAdmin](https://www.pgadmin.org/) is a web application that can be used to manage and access PostgreSQL databases.

![PgAdmin](https://www.pgadmin.org/static/COMPILED/assets/img/screenshots/pgadmin4-welcome.png)

This is a great tool to query our PostgreSQL database.

## Docker Compose (setup PostgreSQL and PgAdmin)

We can take advantage of [docker](https://docs.docker.com/) and [docker compose](https://docs.docker.com/compose/) to setup third party dependencies, such as PostgreSQL and PgAdmin.

1. Create the docker compose file

   Create file: `docker-compose.yml`

   {% include custom/proceed_with_caution.html details="Do not reuse credentials between different services.<br/>For convenience, the following example uses the same credentials (<code>DATABASE_USERNAME</code> and <code>DATABASE_PASSWORD</code>) for both the PostgreSQL and PgAdmin services.  <strong>Do not do this in production!!</strong>" %}

   ```yaml
   version: "3.8"
   services:
     postgres:
       container_name: ${DATABASE_NAME}-pg
       image: postgres:11.1
       restart: unless-stopped
       networks:
         - app-net
       env_file:
         - .env
       ports:
         - ${DATABASE_PORT}:5432
       environment:
         PGUSER: ${DATABASE_USERNAME}
         POSTGRES_USER: ${DATABASE_USERNAME}
         POSTGRES_PASSWORD: ${DATABASE_PASSWORD}
         POSTGRES_DB: ${DATABASE_NAME}
       healthcheck:
         test: ["CMD", "pg_isready", "-U", "postgres", "-d", "${DATABASE_NAME}"]
         interval: 10s
         timeout: 5s
         retries: 5
         start_period: 30s
     pgadmin4:
       container_name: ${DATABASE_NAME}-pgadmin4
       image: dpage/pgadmin4:4.22
       restart: unless-stopped
       networks:
         - app-net
       env_file:
         - .env
       volumes:
         - ./docker/pgadmin4:/pgadmin4/external
       environment:
         PGADMIN_DEFAULT_EMAIL: ${DATABASE_USERNAME}
         PGADMIN_DEFAULT_PASSWORD: ${DATABASE_PASSWORD}
         PGADMIN_SERVER_JSON_FILE: /pgadmin4/external/servers.json
       ports:
         - "8000:80"
   networks:
     app-net:
       driver: bridge
   ```

   The above defines two services

   1. The `postgres` service

      ```yaml
        postgres:
          container_name: ${DATABASE_NAME}-pg
          image: postgres:11.1
          restart: unless-stopped
          networks:
            - app-net
          env_file:
            - .env
          ports:
            - ${DATABASE_PORT}:5432
          environment:
            PGUSER: ${DATABASE_USERNAME}
            POSTGRES_USER: ${DATABASE_USERNAME}
            POSTGRES_PASSWORD: ${DATABASE_PASSWORD}
            POSTGRES_DB: ${DATABASE_NAME}
          healthcheck:
            test: ["CMD", "pg_isready", "-U", "postgres", "-d", "${DATABASE_NAME}"]
            interval: 10s
            timeout: 5s
            retries: 5
      ```

   1. The `pgadmin4` service

      ```yaml
        pgadmin4:
          container_name: ${DATABASE_NAME}-pgadmin4
          image: dpage/pgadmin4:4.22
          restart: unless-stopped
          networks:
            - app-net
          env_file:
            - .env
          volumes:
            - ./docker/pgadmin4:/pgadmin4/external
          environment:
            PGADMIN_DEFAULT_EMAIL: ${DATABASE_USERNAME}
            PGADMIN_DEFAULT_PASSWORD: ${DATABASE_PASSWORD}
            PGADMIN_SERVER_JSON_FILE: /pgadmin4/external/servers.json
          ports:
            - "8000:80"
      ```

   Both services make use of the same `.env` file.  This ensures that the database and our application refer to the same credentials and saves us the hassle of syncing all parts.

1. Update the `.env` file

   Update file: `.env`

   {% include custom/note.html details="The application will fail the integration tests until we include the new PostgreSQL database driver dependency." %}

   ```properties
   DATABASE_NAME=contact-us
   DATABASE_PORT=5432
   DATABASE_URL=jdbc:postgresql://localhost:5432/contact-us
   DATABASE_DRIVER=org.postgresql.Driver
   DATABASE_USERNAME=tw-data
   DATABASE_PASSWORD=SomeRandomPassword
   ```

   The database driver, `DATABASE_DRIVER`, now points to the PostgreSQL driver `org.postgresql.Driver` not to H2 anymore.  Running the integration tests now will fail as still we need to update the application.  Furthermore, the application will not start as we are missing the driver.

1. Start the services defined in the docker compose file

   ```bash
   $ docker-compose up -d
   ```

   The `-d` flag will start docker compose in the background.

   ```bash
   Creating network "contact-us_app-net" with driver "bridge"
   Creating contact-us-pgadmin4 ... done
   Creating contact-us-pg       ... done
   ```

1. Access PgAdmin; [http://localhost:8000/](http://localhost:8000/)

   PgAmin is using the database credentials (`DATABASE_USERNAME` and `DATABASE_PASSWORD`) defined in the `.env` file.

   | Property | Environment Variable | Value                |
   | -------- | -------------------- | -------------------- |
   | Username | `DATABASE_USERNAME`  | `tw-data`            |
   | Password | `DATABASE_PASSWORD`  | `SomeRandomPassword` |

   ![PgAdmin-Login.png]({{ '/assets/images/PgAdmin-Login.png' | absolute_url }})

1. No database servers defined

   ![PgAdmin No Servers]({{ '/assets/images/PgAdmin-No-Servers.png' | absolute_url}})

   Note that PgAdmin is not yet connected to our PostgreSQL database.  We can configure the new connection from here, but that's discouraged.

1. Setup database connection

   The `pgadmin4` service, defined in `docker-compose.yml`, defines [`PGADMIN_SERVER_JSON_FILE`](https://www.pgadmin.org/docs/pgadmin4/development/container_deployment.html#environment-variables) environment variable.  The _server json file_ points to a volume that is mounted from the `./docker/pgadmin4` directory.  This allows us to manage the database connection settings as part of the code.

      ```yaml
        pgadmin4:
          container_name: ${DATABASE_NAME}-pgadmin4
          image: dpage/pgadmin4:4.22
          restart: unless-stopped
          networks:
            - app-net
          env_file:
            - .env
          volumes:
            - ./docker/pgadmin4:/pgadmin4/external
          environment:
            PGADMIN_DEFAULT_EMAIL: ${DATABASE_USERNAME}
            PGADMIN_DEFAULT_PASSWORD: ${DATABASE_PASSWORD}
            PGADMIN_SERVER_JSON_FILE: /pgadmin4/external/servers.json
          ports:
            - "8000:80"
      ```

   This file can be used to preconfigure the server connections.

   Create file `docker/pgadmin4/servers.json`

   ```json
   {
     "Servers": {
       "1": {
         "Name": "Contact US DB",
         "Group": "Servers",
         "Host": "postgres",
         "Port": 5432,
         "MaintenanceDB": "contact-us",
         "Username": "tw-data",
         "SSLMode": "prefer",
         "SSLCert": "<STORAGE_DIR>/.postgresql/postgresql.crt",
         "SSLKey": "<STORAGE_DIR>/.postgresql/postgresql.key",
         "SSLCompression": 0,
         "Timeout": 10,
         "UseSSHTunnel": 0,
         "TunnelPort": "22",
         "TunnelAuthentication": 0
       }
     }
   }
   ```

   {% include custom/note.html details="The above file does not make use of environment variables and you need to make sure that the database name (<code>DATABASE_NAME</code>) and the database username (<code>DATABASE_USERNAME</code>) match those defined in the <code>.env</code> file." %}

1. Delete the existing containers

   {% include custom/proceed_with_caution.html details="The following command will delete <strong>all</strong> stopped containers" %}

   If you do not want to delete old containers, please do not run the `docker system prune -f` command.

   PgAdmin is stateful and we need to delete the container first to delete any existing state.

   ```bash
   $ docker-compose stop
   $ docker system prune -f
   ```

   The above commands can be grouped as one

   ```bash
   $ docker-compose stop && docker system prune -f
   ```

   The docker containers are first stopped and then deleted.

   ```bash
   Stopping contact-us-pg         ... done
   Stopping contact-us-pgadmin4   ... done
   Deleted Containers:
   7fd865ee5aced2b8265611fb15e00feaea33679cf8c5f234efd300287ba15b39
   a694df37731836953b9c272ab529c26852bd92f78f317a31172c15ee4fd914b3

   Deleted Networks:
   contact-us_app-net

   Total reclaimed space: 154B
   ```

1. Start the services defined in the docker compose file

   ```bash
   $ docker-compose up -d
   ```

1. Login to PgAdmin; [http://localhost:8000/](http://localhost:8000/)

1. Expand the servers and login to the PostgreSQL database

   The database credentials (`DATABASE_PASSWORD`) is defined in the `.env` file.

   | Property | Environment Variable | Value                |
   | -------- | -------------------- | -------------------- |
   | Password | `DATABASE_PASSWORD`  | `SomeRandomPassword` |

   ![PgAdmin Connect]({{ '/assets/images/PgAdmin-Connect.png' | absolute_url }})

1. Expand the databases and navigate the available objects

   ![PgAdmin Connected]({{ '/assets/images/PgAdmin-Connected.png' | absolute_url }})

   Note that we have no tables created for now.  These are automatically created by [Flyway](https://flywaydb.org/) when the integration tests are executed or the application is started.

The database is setup and ready to be used by our application.

## Use PostgreSQL

1. Replace the database dependency

   Update file `build.gradle`, delete the H2 database driver

   ```groovy
     runtimeOnly 'com.h2database:h2'
   ```

   then add the PostgreSQL database driver

   ```groovy
     runtimeOnly 'org.postgresql:postgresql'
   ```

   The following fragment shows the dependencies following the update.

   ```groovy
   dependencies {
     /* Lombok */
     compileOnly 'org.projectlombok:lombok'
     annotationProcessor 'org.projectlombok:lombok'

     /* Spring */
     implementation 'org.springframework.boot:spring-boot-starter-web'
     implementation 'org.springframework.boot:spring-boot-starter-actuator'
     testImplementation('org.springframework.boot:spring-boot-starter-test') {
       exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
     }

     /* Data */
     implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
     runtimeOnly 'org.flywaydb:flyway-core'
     runtimeOnly 'org.postgresql:postgresql'

     /* OpenApi/Swagger */
     implementation 'org.springdoc:springdoc-openapi-ui:1.3.9'
   }
   ```

1. Restart the services

   {% include custom/proceed_with_caution.html details="The following command will delete <strong>all</strong> stopped containers." %}

   If you do not want to delete old containers, please do not run the `docker system prune -f` command.

   ```bash
   $ docker-compose stop
   $ docker system prune -f
   $ docker-compose up -d
   ```

   Alternatively, as a single command

   ```bash
   $ docker-compose stop && docker system prune -f && docker-compose up -d
   ```

1. Run the integration tests

   ```bash
   $ ./gradlew clean integrationTest

   ...
   BUILD SUCCESSFUL in 11s
   6 actionable tasks: 6 executed
   ```

   The integration test should pass

1. Expands the tables node

   There should be two tables:

   1. `flyway_schema_history`: used by Flyway to manage the database migration state.  This is how Flyway keeps track of which scripts are executed and which scripts are pending.
   1. `offices`: our table

1. Query the database

   Select the `offices` table and then click _Query Tool_ icon and then run the query

   ```sql
   SELECT * FROM "offices";
   ```

   ![PgAdmin Tables Created]({{ '/assets/images/PgAdmin-Tables-Created.png' | absolute_url }})

   This will display all offices that were populated by Flyway during the migration.

## Remove H2 references

We have deleted the H2 dependency from the `build.gradle` file.  We still may have H2 related configuration in our `application.yaml` properties file.

1. Remove the H2 configuration

   Update file: `src/main/resources/application.yaml`

   Remove the `h2` configuration by deleting the following.

   ```yaml
     h2:
       console:
         enabled: true
         path: /h2
   ```
