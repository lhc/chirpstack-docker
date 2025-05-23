# ChirpStack Docker Environment for LHC Hackerspace

This repository contains the LoRaWAN infrastructure setup used at LHC (Laboratório Hacker de Campinas), a hackerspace in Campinas, Brazil. It provides a complete IoT development environment using [ChirpStack](https://www.chirpstack.io) as the LoRaWAN Network Server (v4) and [Docker Compose](https://docs.docker.com/compose/) for easy deployment.

This setup is specifically designed for IoT development and experimentation at LHC, providing:
* A complete LoRaWAN network server
* MQTT broker for device communication
* RabbitMQ for message queuing
* PostgreSQL for data persistence
* REST API for integration

The deployment of this environment is automated using Ansible scripts, which can be found in our [Ansible Scripts repository](https://github.com/lhc/Ansilbe_Scripts/tree/master/Chirpstack). These scripts handle the server setup, environment variables configuration, and service deployment.

**Note:** While this setup is optimized for LHC's IoT development needs, it can be used as a starting point for other LoRaWAN deployments. For production usage, additional security and performance considerations might be needed.

**Note:** Please use this `docker-compose.yml` file as a starting point for testing
but keep in mind that for production usage it might need modifications. 

## Directory layout

* `docker-compose.yml`: the docker-compose file containing the services
* `configuration/chirpstack`: directory containing the ChirpStack configuration files
* `configuration/chirpstack-gateway-bridge`: directory containing the ChirpStack Gateway Bridge configuration
* `configuration/mosquitto`: directory containing the Mosquitto (MQTT broker) configuration
* `configuration/postgresql/initdb/`: directory containing PostgreSQL initialization scripts
* `configuration/rabbitmq`: directory containing RabbitMQ configuration files

## Configuration

This setup is pre-configured for all regions. You can either connect a ChirpStack Gateway Bridge
instance (v3.14.0+) to the MQTT broker (port 1883) or connect a Semtech UDP Packet Forwarder.
Please note that:

* You must prefix the MQTT topic with the region.
  Please see the region configuration files in the `configuration/chirpstack` for a list
  of topic prefixes (e.g. eu868, us915_0, au915_0, as923_2, ...).
* The protobuf marshaler is configured.

This setup also comes with two instances of the ChirpStack Gateway Bridge. One
is configured to handle the Semtech UDP Packet Forwarder data (port 1700), the
other is configured to handle the Basics Station protocol (port 3001). Both
instances are by default configured for EU868 (using the `eu868` MQTT topic
prefix).

### Environment Variables

The following environment variables can be configured:

* `LOG_LEVEL`: ChirpStack log level (default: "info", options: "trace", "debug", "info", "warn", "error")
* `RABBITMQ_ADMIN_USER`: RabbitMQ admin username
* `RABBITMQ_ADMIN_PASSWORD`: RabbitMQ admin password
* `SERVER_BROKER_HOST`: MQTT broker host
* `SERVER_BROKER_USER`: MQTT broker username
* `SERVER_BROKER_PASS`: MQTT broker password

### Reconfigure regions

ChirpStack has at least one configuration of each region enabled. You will find
the list of `enabled_regions` in `configuration/chirpstack/chirpstack.toml`.
Each entry in `enabled_regions` refers to the `id` that can be found in the
`region_XXX.toml` file. This `region_XXX.toml` also contains a `topic_prefix`
configuration which you need to configure the ChirpStack Gateway Bridge
UDP instance (see below).

#### ChirpStack Gateway Bridge (UDP)

Within the `docker-compose.yml` file, you must replace the `eu868` prefix in the
`INTEGRATION__..._TOPIC_TEMPLATE` configuration with the MQTT `topic_prefix` of
the region you would like to use (e.g. `us915_0`, `au915_0`, `in865`, ...).

#### ChirpStack Gateway Bridge (Basics Station)

Within the `docker-compose.yml` file, you must update the configuration file
that the ChirpStack Gateway Bridge instance must used. The default is
`chirpstack-gateway-bridge-basicstation-eu868.toml`. For available
configuration files, please see the `configuration/chirpstack-gateway-bridge`
directory.

# Data persistence

PostgreSQL and Redis data is persisted in Docker volumes, see the `docker-compose.yml`
`volumes` definition.

## Requirements

Before using this `docker-compose.yml` file, make sure you have [Docker](https://www.docker.com/community-edition)
installed.

## Importing device repository

To import the [lorawan-devices](https://github.com/TheThingsNetwork/lorawan-devices)
repository (optional step), run the following command:

```bash
make import-lorawan-devices
```

This will clone the `lorawan-devices` repository and execute the import command of ChirpStack.
Please note that for this step you need to have the `make` command installed.

**Note:** an older snapshot of the `lorawan-devices` repository is cloned as the
latest revision no longer contains a `LICENSE` file.

## Usage

To start the ChirpStack simply run:

```bash
$ docker-compose up
```

After all the components have been initialized and started, you should be able
to open http://localhost:8080/ in your browser.

The example includes:
* [ChirpStack REST API](https://github.com/chirpstack/chirpstack-rest-api) at http://localhost:8090
* RabbitMQ Management Interface at http://localhost:15672

**Note:** It is recommended to use the [gRPC](https://www.chirpstack.io/docs/chirpstack/api/grpc.html)
interface over the [REST](https://www.chirpstack.io/docs/chirpstack/api/rest.html) interface.
