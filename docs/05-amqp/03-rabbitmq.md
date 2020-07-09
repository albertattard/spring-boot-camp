---
layout: default
title: RabbitMQ
description: Setup RabbitMQ and RabbitMQ Management using docker compose
parent: Advanced Message Queuing Protocol
nav_order: 3
permalink: docs/amqp/rabbit-mq/
lang: en
social:
  name: Albert Attard
  links:
    - https://www.linkedin.com/in/javacreed/
---

# RabbitMQ
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## RabbitMQ

{% include custom/pending.html %}

[https://www.rabbitmq.com/](https://www.rabbitmq.com/)

## RabbitMQ management

{% include custom/pending.html %}

[https://www.rabbitmq.com/management.html](https://www.rabbitmq.com/management.html)

## Docker Compose (setup RabbitMQ management)

We can take advantage of [docker](https://docs.docker.com/) and [docker compose](https://docs.docker.com/compose/) to setup third party dependencies, such as RabbitMQ and the management API.

1. Update the `.env` file

   Update file: `.env`

   ```properties
   # Rabbit MQ
   RABBITMQ_HOST=localhost
   RABBITMQ_PORT=5672
   RABBITMQ_USERNAME=tw-data
   RABBITMQ_PASSWORD=SomeRandomPassword
   ```

1. Configure the Queues

   Create file: `docker/rabbitmq/definitions.json`

   ```json
   {
     "queues": [
       {
         "name": "event",
         "vhost": "/",
         "durable": true,
         "auto_delete": false,
         "arguments": {}
       },
       {
         "name": "food",
         "vhost": "/",
         "durable": true,
         "auto_delete": false,
         "arguments": {}
       }
     ]
   }
   ```

1. Add RabbitMQ to the docker compose file

   Update file: `docker-compose.yml`

   ```yaml
     rabbitmq:
       container_name: ${APPLICATION_NAME}-rabbitmq
       image: rabbitmq:3.8.5-management-alpine
       restart: unless-stopped
       networks:
         - app-net
       ports:
         - ${MESSAGE_QUEUE_PORT}:5672
         - 15672:15672
       volumes:
         - ./docker/rabbitmq/definitions.json:/etc/rabbitmq/definitions.json:ro
       environment:
         RABBITMQ_DEFAULT_USER: ${MESSAGE_QUEUE_USERNAME}
         RABBITMQ_DEFAULT_PASS: ${MESSAGE_QUEUE_PASSWORD}
       healthcheck:
         test: [ "CMD", "nc", "-z", "localhost", "5672" ]
         interval: 30s
         timeout: 5s
         retries: 5
         start_period: 30s
   ```

1. Start the services defined in the docker compose file

   {% include custom/proceed_with_caution.html details="The following command will delete <strong>all</strong> stopped containers.  If you do not want to delete old containers, please do not run the <code>docker system prune -f</code> command." %}

   ```bash
   $ docker-compose stop && docker system prune -f && docker-compose up -d
   ```

   The `-d` flag will start docker compose in the background.

   ```bash
   ...
   Creating network "contact-us_app-net" with driver "bridge"
   Creating contact-us-pgadmin4   ... done
   Creating contact-us-rabbitmq   ... done
   Creating contact-us-prometheus ... done
   Creating contact-us-pg         ... done
   ```

1. Access RabbitMQ Management: [http://localhost:15672/#/queues](http://localhost:15672/#/queues)

   The RabbitMQ management service is using the message queue credentials (`MESSAGE_QUEUE_USERNAME` and `MESSAGE_QUEUE_PASSWORD`) defined in the `.env` file.

   | Property | Environment Variable      | Value                |
   | -------- | ------------------------- | -------------------- |
   | Username | `MESSAGE_QUEUE_USERNAME`  | `tw-data`            |
   | Password | `MESSAGE_QUEUE_PASSWORD`  | `SomeRandomPassword` |

   ![RabbitMQ-Login.png]({{ '/assets/images/RabbitMQ-Login.png' | absolute_url }})

   RabbitMQ will show the queues, configured before (in file `docker/rabbitmq/definitions.json`).

   ![RabbitMQ-Queues-Empty.png]({{ '/assets/images/RabbitMQ-Queues-Empty.png' | absolute_url }})
