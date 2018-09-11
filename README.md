# Traefik

Traefik is a reverse proxy.

In a docker development environment it's usefull in order to access to web containers with a complete url instead of playing with port number (http://my-domain.docker.local vs http://localhost:8123)

It will listen to default http port number (80) so if you have a container already on this port or a local web server please, update your container configuration and/or stop your local webserver.

# Prerequisites

## Create a docker network to let communicate the reverse proxy with the containers in the docker land

`docker network create proxy`

## Copy the env-dist into a .env file

`$/workspace/traefik: cp env-dist .env`

# Edit /etc/hosts and docker-compose.yml

Assuming we want all our domain with the following pattern `*.docker.local` (you could set the value of your choice in `.env` file)

## Each domain must be know by your local machine

`echo "127.0.0.1    my-domain.docker.local" >> /etc/hosts`

## Each domain must be affected to a container. Edit the docker-compose.yml file (or create a docker-compose.override.yml file instead)

### Add the traefik labels and the proxy network on the container
Example:
```
  httpd:
    image: httpd:2.4
    networks:
      - akeneo
      - proxy #Add this network to the container and the following labels
    labels:
      - "traefik.docker.network=proxy"
      - "traefik.frontend.rule=Host:my-domain.docker.local"
      - "traefik.enable=true"
```

### Add the network configuration at the end of the docker-compose.yml file

```
networks:
    proxy:
        external: true
```

### Restart you docker compose configuration

`$/workspace/myProject: docker-compose up -d`

# Launch traefik service

`$/workspace/traefik: docker-compose up -d`


# HTTPS

Traefik comes with the https support using https://letsencrypt.org/ service.

Here are the three steps configuration:

- Provide your email in the `.env` file
- Create an `acme.json` file with low rights `$/workspace/traefik: touch acme.json && chmod 600 acme.json`
- Launch traefik container with the good configuration.

`$/workspace/traefik: docker-compose -f docker-composer.yml -f docker-compose.https.yml up -d`

More information about https with traefik here https://docs.traefik.io/user-guide/docker-and-lets-encrypt/