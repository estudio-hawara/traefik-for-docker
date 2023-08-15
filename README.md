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

### Start the Router

The `prod` and `dev` scripts allows you to quickly run the service in either production or development mode.

```bash
# ... in production
./prod up -d

# ... in development
./dev up -d
```

Keep in mind that you won't use both types of environments at once.

### Set Up a Production Environment

The main difference between production and development environments is that production environments require the services to use Let's Encrypt certificates.

Apart from making your public domain reach the public IP address that's pointing to Traefik, you'll also need to configure an environment variable:

```sh
TRAEFIK_CERTIFICATESRESOLVERS_LETSSOLVER_ACME_EMAIL
```

The easiest way to achieve this is to copy `.env.example` as `.env` and add your email there.

## Check the Service Status

You can also use the scripts to check the status of the Traefik service.

```bash
./prod ps
```

Of course, this and all the other commands also works with the `dev` script.

### Read the Logs

You can read the logs of your Traefik container with:

```bash
./prod logs
```

Or follow it live as its changed with:

```bash
./prod -f logs
```

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