# dbildungs-iam

dbildungs-iam is a customized [`Keycloak`](https://github.com/keycloak/keycloak) docker image for the [dBildungscloud Server](https://github.com/hpi-schul-cloud/schulcloud-server).

Images for local development or production are build for each Git tag via GitHub Actions.

## Manual build for local development image

Following steps are intent to build the container for local development purpose.

### Build steps (development)

You may use a pre-build image from [`GitHub Packages`](https://github.com/orgs/hpi-schul-cloud/packages?repo_name=dbildungs-iam). To build the container on your own execute following command:

```bash
docker build --target development -t schulcloud/dbildungs-iam/dev .
```

To create the container execute following command:

```bash
docker create --name dbildungs-iam -p 8080:8080 -p 8443:8443 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin schulcloud/dbildungs-iam/dev:latest
```

To start (or stop) the container execute following command:

```bash
docker start dbildungs-iam
```

```bash
docker stop dbildungs-iam
```

The Keycloak Admin Console will be available at [`http://localhost:8080`](http://localhost:8080) or [`https://localhost:8443`](https://localhost:8443). You may login into the instance with username `admin` and password `admin`.

### Keycloak configuration

The developer build is configured to start Keycloak in developer mode. It is configured without proxy or clustering capabilities (discovery, replication, fail-over). It'll use a local flat-file database, has self-signed certificates for TLS, and exposes [`metrics`](http://localhost:8080/metrics).

## Manual build for production image

Note, the production image can not be used locally without setting up an TLS termination proxy. Following steps are intent to build the container for production purpose to test it locally. For your development environment you want to make use of the [`development image`](#manual-build-for-local-development-image).

The production image is build automatically. You may use a pre-build image from [`GitHub Packages`](https://github.com/orgs/hpi-schul-cloud/packages?repo_name=dbildungs-iam).

### Build steps (production)

To build the container execute following command:

```bash
docker build --target production -t schulcloud/dbildungs-iam .
```

To use the container, e.g. to test it locally, you'll need a PostgresSQL database up and running. To start a PostgresSQL container execute following commands:

```bash
docker network create dbildungs-iam
docker run --name postgres --network=dbildungs-iam -p 5432:5432 -e POSTGRES_PASSWORD=postgres -d postgres
```

Adjust, and execute following command to start the Keycloak production container for local testing:

```bash
docker create --name dbildungs-iam --network=dbildungs-iam -p 8080:8080 \
    -e KEYCLOAK_ADMIN=admin \
    -e KEYCLOAK_ADMIN_PASSWORD=admin \
    -e KC_DB_URL=jdbc:postgresql://postgres:5432/postgres \
    -e KC_DB_USERNAME=postgres \
    -e KC_DB_PASSWORD=postgres \
    -e KC_HTTP_ENABLED=true \
    -e KC_PROXY=edge \
    -e KC_HOSTNAME=localhost:8080 \
    schulcloud/dbildungs-iam:latest
```

To start (or stop) the container execute following command:

```bash
docker start dbildungs-iam
```

```bash
docker stop dbildungs-iam
```

The Keycloak Admin Console will be available at [`http://localhost:8080`](http://localhost:8080). To make use of the production image locally, you need to configure a TLS termination proxy (setup is beyond the scope of this document).

## Bcrypt Setup

To make use of BCrypt hashed passwords, the [`keycloak-bcrypt`](https://github.com/leroyguillaume/keycloak-bcrypt/) is used.

> The Bcrypt provider can be found [here](./src/providers). During build, it will be copied into the image and is
> available for late use.

- By default, the default Keycloak hashing provider will be used for password hashing
- To change the default behavior, login as admin under <http://localhost:8080>
- Under `Authentication -> Password Policy` create a new `Hashing Algorithm` entry with value `bcrypt`
- Optional: Under `Authentication -> Password Policy` create a new `Hashing Iterations` entry with your desired value, this will be used as cost for bcrypt and the default value is 10

## Structure

- `./src`: Folder containing Keycloak customization (e.g. plug-ins, themes)
- `./Dockerfile`: The multi-staged Dockerfile for develop or production build.
- `./build-dev.sh`: Builds the Keycloak image for local development.
- `./create-dev.sh`: Creates the Keycloak container for local development.
