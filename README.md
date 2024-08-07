# ChirpStack Setup with Portainer

This repository provides the configuration files necessary to set up a basic instance of ChirpStack using Docker and Portainer. ChirpStack is an open-source LoRaWAN Network Server stack that provides all necessary components for running a LoRaWAN network. This setup uses configuration files and settings referenced from the [ChirpStack GitHub repository](https://github.com/chirpstack).

## Table of Contents

- [Overview](#overview)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)

## Overview

This setup uses Docker Compose to run the following services:

- **ChirpStack**: The main LoRaWAN network server.
- **PostgreSQL**: Database to store ChirpStack data.
- **Redis**: In-memory data structure store used for caching.
- **Mosquitto**: MQTT broker for message queuing.

## Requirements

- Docker
- Docker Compose
- Portainer (optional, for managing Docker containers via a web interface)

## Installation

### Step 1: Clone the Repository

Clone this repository to your local machine:

```bash
git clone https://github.com/yourusername/chirpstack-portainer-setup.git
cd chirpstack-portainer-setup
```

### Step 2: Configure External Configurations

Ensure that the following external configurations are available on your Docker host. These files should be located in a directory that is accessible to Docker.

- **ChirpStack Configuration:** `chirpstack_configV1`
- **PostgreSQL Extensions:** `postgres_extensions_007`
- **Mosquitto Configuration:** `chirpstack_mqtt_config`

### Step 3: Start the Services

Run the following command to start all services using Docker Compose:

```bash
docker-compose up -d
```

This command will start all services in detached mode.

## Configuration

### Docker Compose

The `docker-compose.yml` file defines the service configurations:

```yaml
version: '3.8'

services:
  chirpstack:
    image: chirpstack/chirpstack
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: postgres://postgres:postgres@postgres/postgres?sslmode=disable
      REDIS_HOST: redis
      POSTGRESQL_HOST: postgres
      MQTT_BROKER_HOST: mosquitto
    volumes:
      - chirpstack_data:/data
      - chirpstack_config:/etc/chirpstack
    depends_on:
      - postgres
      - redis
      - mosquitto
    configs:
      - source: chirpstack_configV1
        target: /etc/chirpstack/chirpstack.toml
    networks:
      - chirpstack_net

  postgres:
    image: postgres:latest
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - chirpstack_postgres_entrypoint:/docker-entrypoint-initdb.d
    configs:
      - source: postgres_extensions_007
        target: /docker-entrypoint-initdb.d/postgres_extensions_007.sh
    networks:
      - chirpstack_net

  redis:
    image: redis:latest
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data
    networks:
      - chirpstack_net

  mosquitto:
    image: eclipse-mosquitto:latest
    ports:
      - "1883:1883"
    volumes:
      - mosquitto_data:/mosquitto/data
      - mosquitto_config:/mosquitto/config
      - mosquitto_log:/mosquitto/log
    configs:
      - source: chirpstack_mqtt_config
        target: /mosquitto/config/mosquitto.conf
    networks:
      - chirpstack_net

volumes:
  chirpstack_data:
  chirpstack_config:
  postgres_data:
  chirpstack_postgres_entrypoint:
  mosquitto_data:
  mosquitto_config:
  mosquitto_log:
  redisdata:

configs:
  chirpstack_configV1:
    external: true  
  postgres_extensions_007:
    external: true
  chirpstack_mqtt_config:
    external: true

networks:
  chirpstack_net:
    external: true
```

### ChirpStack Configuration

The `chirpstack_configV1` file contains the configuration for ChirpStack, including logging, database, and network settings.

```toml
# Logging.
[logging]
level="info"

# PostgreSQL configuration.
[postgresql]
dsn="postgres://postgres:postgres@$POSTGRESQL_HOST/postgres?sslmode=disable"
max_open_connections=10
min_idle_connections=0

# Redis configuration.
[redis]
servers=["redis://$REDIS_HOST/"]
tls_enabled=false
cluster=false

# Network related configuration.
[network]
net_id="000000"
enabled_regions=[
  "as923",
  "as923_2",
  "as923_3",
  "as923_4",
  "au915_0",
  "cn470_10",
  "cn779",
  "eu433",
  "eu868",
  "in865",
  "ism2400",
  "kr920",
  "ru864",
  "us915_0",
  "us915_1",
]

# API interface configuration.
[api]
bind="0.0.0.0:8080"
secret="you-must-replace-this"

[integration]
enabled=["mqtt"]

  [integration.mqtt]
    server="tcp://$MQTT_BROKER_HOST:1883/"
    json=true
```

### PostgreSQL Initialization

The `postgres_extensions_007` script initializes the PostgreSQL database with necessary extensions.

```bash
#!/bin/bash
set -e

psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname="postgres" <<-EOSQL
    create extension pg_trgm;
    create extension hstore;
EOSQL
```

### Mosquitto Configuration

The `chirpstack_mqtt_config` file contains the configuration for the Mosquitto MQTT broker.

```bash
listener 1883
allow_anonymous true
```

## Usage

### Accessing ChirpStack

After starting the services, ChirpStack can be accessed via a web browser at `http://localhost:8080`.

### Managing with Portainer

If you have Portainer installed, you can manage these Docker containers through its web interface, typically accessible at `http://localhost:9000`.

## References

This setup and configuration are based on the [ChirpStack GitHub repository](https://github.com/chirpstack). Please refer to their documentation for more detailed information about the ChirpStack components and configurations.

