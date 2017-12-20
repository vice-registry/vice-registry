# ViCE-Registry Documentation

## Component Overview

ViCE-Registry is build as Docker images and published on Docker hub. We
recommend to use Docker for installations & deployments. Nevertheless the
binaries can be deployed manually as well.


The ViCE-Registry consists of multiple components, which are mostly required for
a successful deployment:

 * vice-api
 * vice-store
 * vice-worker with type export
 * vice-worker with type import
 * vice-webui (optional)

Further, a database (RethinkDB) to store metadata and credentials, and a message
queue (RabbitMQ) for inter-component communications are required.

### ViCE API

This component is the RESTful API endpoint for communications from clients to the ViCE Registry.

```
Usage:
  vice-api [OPTIONS]

A RESTful API server for the ViCE Image Registry

Application Options:
      --scheme=             the listeners to enable, this can be repeated and
                            defaults to the schemes in the swagger spec
      --cleanup-timeout=    grace period for which to wait before shutting down
                            the server (default: 10s)
      --max-header-size=    controls the maximum number of bytes the server
                            will read parsing the request header's keys and
                            values, including the request line. It does not
                            limit the size of the request body. (default: 1MiB)
      --socket-path=        the unix socket to listen on (default:
                            /var/run/vice.sock)
      --host=               the IP to listen on (default: localhost) [$HOST]
      --port=               the port to listen on for insecure connections,
                            defaults to a random value [$PORT]
      --listen-limit=       limit the number of outstanding requests
      --keep-alive=         sets the TCP keep-alive timeouts on accepted
                            connections. It prunes dead TCP connections ( e.g.
                            closing laptop mid-download) (default: 3m)
      --read-timeout=       maximum duration before timing out read of the
                            request (default: 30s)
      --write-timeout=      maximum duration before timing out write of the
                            response (default: 60s)
      --tls-host=           the IP to listen on for tls, when not specified
                            it's the same as --host [$TLS_HOST]
      --tls-port=           the port to listen on for secure connections,
                            defaults to a random value [$TLS_PORT]
      --tls-certificate=    the certificate to use for secure connections
                            [$TLS_CERTIFICATE]
      --tls-key=            the private key to use for secure conections
                            [$TLS_PRIVATE_KEY]
      --tls-ca=             the certificate authority file to be used with
                            mutual tls auth [$TLS_CA_CERTIFICATE]
      --tls-listen-limit=   limit the number of outstanding requests
      --tls-keep-alive=     sets the TCP keep-alive timeouts on accepted
                            connections. It prunes dead TCP connections ( e.g.
                            closing laptop mid-download)
      --tls-read-timeout=   maximum duration before timing out read of the
                            request
      --tls-write-timeout=  maximum duration before timing out write of the
                            response

RethinkDB Connection:
  -c, --rethinkdb-location= Location of the RethinkDB cluster to connect to
                            (e.g. localhost)
  -d, --rethinkdb-database= Database Name of RethinkDB cluster to use (e.g.
                            vice)

RabbitMQ Connection:
  -r, --rabbitmq-location=  Location of the RabbitMQ to connect to (e.g.
                            localhost)
      --rabbitmq-user=      Username to log in to RabbitMQ
      --rabbitmq-pass=      Password to log in to RabbitMQ

Help Options:
  -h, --help                Show this help message
```

### ViCE Worker

This component is responsible for importing and exporting virtual images from and to execution environments.

```
Usage:
  vice-worker [OPTIONS]

Image Import component of the ViCE Image Registry

Application Options:
  -i, --import              Define instance as import worker
  -e, --export              Define instance as export worker

RethinkDB Connection:
  -c, --rethinkdb-location= Location of the RethinkDB cluster to connect to
                            (e.g. localhost)
  -d, --rethinkdb-database= Database Name of RethinkDB cluster to use (e.g.
                            vice)

RabbitMQ Connection:
      --rabbitmq-location=  Location of the RabbitMQ to connect to (e.g.
                            localhost)
      --rabbitmq-user=      Username to log in to RabbitMQ
      --rabbitmq-pass=      Password to log in to RabbitMQ

Help Options:
  -h, --help                Show this help message
```

### ViCE Store

This component stores and retrieves the imported virtual images. The ViCE worker communicates with the ViCE store when importing / exporting images.

```
Usage:
  vice-store [OPTIONS]

Store component of the ViCE Image Registry

RethinkDB Connection:
  -c, --rethinkdb-location= Location of the RethinkDB cluster to connect to
                            (e.g. localhost)
  -d, --rethinkdb-database= Database Name of RethinkDB cluster to use (e.g.
                            vice)

RabbitMQ Connection:
      --rabbitmq-location=  Location of the RabbitMQ to connect to (e.g.
                            localhost)
      --rabbitmq-user=      Username to log in to RabbitMQ
      --rabbitmq-pass=      Password to log in to RabbitMQ

Storage Connection:
      --storage-basepath=   Basepath to store the imported images

Help Options:
  -h, --help                Show this help message
```

## Deploy with Docker (recommended)

We recommend to use [docker-compose](https://docs.docker.com/compose/) or
compatible container cluster tools like [Rancher](https://rancher.com/rancher/). 

An example `docker-compose.yaml` file looks like the following (make sure to change the password accordingly):

```
version: '2'
services:
    vice-db:
        image: kallqvist/rethinkdb-cluster
        environment:
         - JOIN=vice-db
        volumes:
         - /srv/vice/vice-db:/data
    vice-mq:
        image: rabbitmq:management
        environment:
         - RABBITMQ_DEFAULT_USER=vice
         - RABBITMQ_DEFAULT_PASS=secret
    vice-api:
        image: viceregistry/vice-api
        restart: on-failure
        ports:
         - 8080:8080
        environment:
         - PORT=8080
         - RETHINKDB_LOCATION=vice-db
         - RETHINKDB_DATABASE=vice
         - RABBITMQ_LOCATION=vice-mq
         - RABBITMQ_USER=vice
         - RABBITMQ_PASS=secret
        depends_on:
         - vice-db
         - vice-mq
    vice-import:
        image: viceregistry/vice-worker
        restart: on-failure
        environment:
         - WORKERTYPE=import
         - RETHINKDB_LOCATION=vice-db
         - RETHINKDB_DATABASE=vice
         - RABBITMQ_LOCATION=vice-mq
         - RABBITMQ_USER=vice
         - RABBITMQ_PASS=secret
        depends_on:
         - vice-db
         - vice-mq
         - vice-store
    vice-export:
        image: viceregistry/vice-worker
        restart: on-failure
        environment:
         - WORKERTYPE=export
         - RETHINKDB_LOCATION=vice-db
         - RETHINKDB_DATABASE=vice
         - RABBITMQ_LOCATION=vice-mq
         - RABBITMQ_USER=vice
         - RABBITMQ_PASS=secret
        depends_on:
         - vice-db
         - vice-mq
         - vice-store
    vice-store:
        image: viceregistry/vice-store
        restart: on-failure
        environment:
         - RETHINKDB_LOCATION=vice-db
         - RETHINKDB_DATABASE=vice
         - RABBITMQ_LOCATION=vice-mq
         - RABBITMQ_USER=vice
         - RABBITMQ_PASS=secret
         - STORAGE_BASEPATH=/opt/images/
        volumes:
         - /srv/vice/images:/opt/images
        depends_on:
         - vice-db
         - vice-mq
    vice-webui:
        image: viceregistry/vice-webui
        restart: on-failure
        ports:
         - 8081:80
        depends_on:
         - vice-api
```

With this `docker-compose.yml` file, a deployment is as simple as calling `docker-compose up` in a terminal in the folder containing the file.
An example `rancher-compose.yml` for deployments with rancher [can be found here](../rancher-compose.yml).

## Manual deployment

The components can be installed manually by downloading the binaries from Github Releases. Make sure to deploy RethinkDB and RabbitMQ before starting the ViCE components.

* [download most recent release of vice-api](https://github.com/vice-registry/vice-api/releases)
* [download most recent release of vice-worker](https://github.com/vice-registry/vice-worker/releases)
* [download most recent release of vice-store](https://github.com/vice-registry/vice-store/releases)
* [download most recent release of vice-webui](https://github.com/vice-registry/vice-webui/releases)

Having the binaries, the application can be deployed by starting with the necessary configuration parameters:

```
./vice-api \
    --port 8080 \
    --rethinkdb-location 172.18.0.3 \
    --rethinkdb-database vice \
    --rabbitmq-location 172.18.0.2 \
    --rabbitmq-user vice \
    --rabbitmq-pass secret &

./vice-worker \
    --import \
    --rethinkdb-location 172.18.0.3 \
    --rethinkdb-database vice \
    --rabbitmq-location 172.18.0.2 \
    --rabbitmq-user vice \
    --rabbitmq-pass secret &

./vice-worker \
    --export \
    --rethinkdb-location 172.18.0.3 \
    --rethinkdb-database vice \
    --rabbitmq-location 172.18.0.2 \
    --rabbitmq-user vice \
    --rabbitmq-pass secret &

./vice-store \
    --storage-basepath /srv/vice/images \
    --rethinkdb-location 172.18.0.3 \
    --rethinkdb-database vice \
    --rabbitmq-location 172.18.0.2 \
    --rabbitmq-user vice \
    --rabbitmq-pass secret &
```

The ViCE WebUI needs to be placed in the documents root of any webserver.