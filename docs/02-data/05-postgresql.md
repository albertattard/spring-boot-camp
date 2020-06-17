---
layout: default
title: PostgreSQL
parent: Spring Data
nav_order: 99
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

## Docker Compose (PostgreSQL and PgAdmin)

We can take advantage of [docker](https://docs.docker.com/) and [docker composed](https://docs.docker.com/compose/) to setup third party dependencies, such as PostgreSQL and [PgAdmin](https://www.pgadmin.org/).

1. Create the `docker-compose.yml` file

   {% include custom/proceed_with_caution.html details="Do not reuse credentials between different services" %}

   For conveniance, the following example uses the same credentials (`DATABASE_USERNAME` and `DATABASE_PASSWORD`) for both services.  **Do not do this in production**.

   Create file: `docker-compose.yml`

   ```yaml
   version: "3"
   services:
     postgres:
       container_name: ${DATABASE_NAME}
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
     pgadmin4:
       container_name: pgadmin4
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
          container_name: ${DATABASE_NAME}
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
          container_name: pgadmin4
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

   Update file `.env`

   ```properties
   DATABASE_NAME=contact-us
   DATABASE_PORT=5432
   DATABASE_URL=jdbc:postgresql://localhost:5432/contact-us
   DATABASE_DRIVER=org.postgresql.Driver
   DATABASE_USERNAME=tw-data
   DATABASE_PASSWORD=SomeRandomPassword
   ```

   {% include custom/note.html details="The application will fail the integration tests until we include the new dependency" %}

   Note that the database driver, `DATABASE_DRIVER`, now points to the PostgreSQL driver `org.postgresql.Driver` not to H2 anymore.  Running the integration tests now will fail as still we need to update the application.  Furthermore, the application will not start as we are missing the driver.

1. Start the services defined by docker composed

   ```bash
   $ docker-compose up -d
   ```

   The `-d` flag will start docker compose in the background.

   ```bash
   Creating network "contact-us_app-net" with driver "bridge"
   Creating pgadmin4   ... done
   Creating contact-us ... done
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

   Note that PgAdmin is not yet connected to our database.  We can configure the new connection from here, but that's discouraged.

1. Setup database connection

   The `pgadmin4` service, defined in `docker-compose.yml`, defines [`PGADMIN_SERVER_JSON_FILE`](https://www.pgadmin.org/docs/pgadmin4/development/container_deployment.html#environment-variables) environment variable.  The _server json file_ points to a volume that is mounted from the `./docker/pgadmin4` directory.  This allows us to manage the database connection settings as part of the code.

      ```yaml
        pgadmin4:
          container_name: pgadmin4
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

   The above file does not make use of environment variables and you need to make sure that the database name
    (`DATABASE_NAME`) and the database username (`DATABASE_USERNAME`) match those defined in the `.env` file.

1. Delete the existing containers

   {% include custom/proceed_with_caution.html details="The following commad will delete all stopped containers" %}

   If you do not want to delete old containers, please do not run the `docker system prune -f` command.

   PgAdmin is stateful and we need to delete the container first.

   ```bash
   $ docker-compose stop
   $ docker system prune -f
   ```

   The above commands can be grouped as one

   ```bash
   $ docker-compose stop && docker system prune -f
   ```

   The docker containers are stopped and then deleted.

   ```bash
   Stopping contact-us ... done
   Stopping pgadmin4   ... done
   Deleted Containers:
   7fd865ee5aced2b8265611fb15e00feaea33679cf8c5f234efd300287ba15b39
   a694df37731836953b9c272ab529c26852bd92f78f317a31172c15ee4fd914b3

   Deleted Networks:
   contact-us_app-net

   Total reclaimed space: 154B
   ```

1. Start the services defined by docker composed

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

1. Connected

   ![PgAdmin Connected]({{ '/assets/images/PgAdmin-Connected.png' | absolute_url }})


## Others

docker-compose stop && docker system prune -f && docker-compose up -d && ./gradlew clean integrationTest




```bash
$ docker-compose up -d

Creating network "contact-us_game-net" with driver "bridge"
Creating contact-us ... done
Creating pgadmin4   ... done
```

```bash
docker-compose stop && docker system prune -f
Stopping contact-us ... done
Stopping pgadmin4   ... done
Deleted Containers:
68e15b698c143816149f0463399a394dcf97b6529478d65ae2d1bb1c10a11685
7c3325978fbf79d1e41f0c60c4901aac7b65a443802d8a7bd7605743f5678396

Deleted Networks:
contact-us_app-net

Total reclaimed space: 154B
```

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

```groovy
  runtimeOnly 'com.h2database:h2'
```

```groovy
  runtimeOnly 'org.postgresql:postgresql'
```

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
  implementation 'org.flywaydb:flyway-core'
  runtimeOnly 'org.postgresql:postgresql'

  /* OpenApi/Swagger */
  implementation 'org.springdoc:springdoc-openapi-ui:1.3.9'
}
```

```bash
$ ./gradlew integrationTest
```

![PgAdmin Tables Created]({{ '/assets/images/PgAdmin-Tables-Created.png' | absolute_url }})


```bash
$ ./gradlew bootRun

...
BUILD FAILED in 4s
3 actionable tasks: 1 executed, 2 up-to-date
```

Update file `build.gradle`

```groovy
bootRun {
  doFirst {
    file("$rootDir/.env").readLines().each() {
      def (key, value) = it.split('=', 2)
      environment key, value
    }
  }
}
```



docker-compose stop && docker system prune -f && docker-compose up -d && ./gradlew clean integrationTest
