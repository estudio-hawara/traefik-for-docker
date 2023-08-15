# Traefik for Docker

## Important Notes

- This project is in early development and it's published without any warrant.
- It's intended to work with web applications and it won't necessarily work in every possible Docker context.

## Install

As you can see, this project consists on a tiny amount of files that can help you setting up a Traefik instance running in Docker.

```bash
git clone https://github.com/hawara-es/traefik-for-docker.git
cd traefik-for-docker
```

To run this Traefik router you need to have Docker in your system. If you are using an Ubuntu machine in a development environment, you may get advantage of the convenience deployment script:

```bash
./deploy/install-docker-in-ubuntu
```

Otherwise, please check the official documentation: [Install Docker Engine](https://docs.docker.com/engine/install/).

### Choose the Environment Type

To deploy the Traefik service you need to create a `.env` file. To help you doing it quickly, there are two templates you can choose from, depending on the type of environment you are setting up:

```bash
# Production
cp .env.prod .env

# Development
cp .env.dev .env
```

The main difference between production and development environments is that production environments require the services to use Let's Encrypt certificates.

This means that, apart from making your public domain reach the public IP address that's pointing to Traefik, you'll also need to configure an environment variable with your email:

```sh
TRAEFIK_CERTIFICATESRESOLVERS_TLSSOLVER_ACME_EMAIL
```

The email will be used to negociate the certificates with Let's Encrypt.

### Start the Router

You can now run your router by using the standard Docker command:

```bash
# Production
docker compose up -d

# Development
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d
```

Keep in mind that you won't use both types of environments at once.

## Deploy Services

### Routing Labels

You'll need to add some labels to the `docker-compose.yml` file of the services that you want Traefik to listen. As an example, these labels would make your `app` work via HTTP and HTTPs.

```yml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.app.rule=Host(`app.localhost`)"
  - "traefik.http.routers.app.entrypoints=web"
  - "traefik.http.routers.app.service=app"
```

Check the [Official Traefik Documentation for Docker Routing Rules](https://doc.traefik.io/traefik/routing/providers/docker/#routers).

### Setting the `traefik` Network

This Traefik container defines a `traefik` network. To get a service to work along with it, you'll need to set the labels described above and also add the service to this network.

```yml
version: '3.8'

services:
  whoami:
    image: containous/whoami
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik"
      - "traefik.http.routers.whoami.entrypoints=web"
      - "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)"
      - "traefik.http.routers.whoami.service=whoami"
      - "traefik.http.services.whoami.loadbalancer.server.port=80"
    networks:
      - traefik

networks:
  traefik:
    name: "traefik"
    external: true
```

Note that you can still add your service to additional networks.