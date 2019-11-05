# Vicoders Docker Traefik

Vicoders docker stack with Traefik as proxy service

- [Vicoders Docker Traefik](#vicoders-docker-traefik)
  - [Overview](#overview)
  - [How it works](#how-it-works)
  - [Installation Guide](#installation-guide)
    - [Pre Install](#pre-install)
    - [Optional configuration](#optional-configuration)
    - [Install](#install)

## Overview

This guide will show you how to set up Traefik as a load balancer/proxy and Consul to store configurations and HTTPS certificates.

## How it works

The idea is to have a main load balancer/proxy that covers all the Docker Swarm cluster and handles HTTPS certificates and requests for each domain.

But doing it in a way that allows you to have other Traefik services inside each stack without interfering with each other, to redirect based on path in the same stack (e.g. one container handles / for a web frontend and another handles /api for an API under the same domain), or to redirect from HTTP to HTTPS selectively.

## Installation Guide

### Pre Install

Create a network that will be shared with Traefik and the containers that should be accessible from the outside

```
docker network create --driver=overlay --attachable vcrobot
```

> You will access the Traefik UI at <your domain>:8080

Create an environment variable with a VC_USER (you will use it for the HTTP Basic Auth for Traefik and Portainer), for example:

```
export VC_USER=admin
```

Create an environment variable with the password, e.g.

```
export VC_PASSWORD=changethis
```

Use openssl to generate the "hashed" version of the password and store it in an environment variable

```
export VC_BASIC_AUTH=$(htpasswd -nb $VC_USER $VC_PASSWORD)
```

### Optional configuration

Enable portainer

```
export PORTAINER_DOMAIN=portainer.domain.com
```

Create an environment variable with your email, to be used for the generation of Let's Encrypt certificates

```
export ACME_EMAIL=acme@domain.com
```

Enable traefik debug

```
export TRAEFIK_DEBUGE=true
```

If you want to enable traefik API

> Enable API in production is not recommendation https://docs.traefik.io/operations/api/#security and you have to remove comment of deploy section in `docker-compose.yaml`

```
export ENABLE_TRAEFIK_API=true
```

If you want to enable traefik API dashboard

```
export ENABLE_TRAEFIK_API_DASHBOARD=true
```

Set insecure mode for api dashboard https://docs.traefik.io/operations/api/#insecure

```
export ENABLE_TRAEFIK_API_INSECURE=true
```

Set name for cert resolver https://docs.traefik.io/routing/routers/#certresolver

```
export ENABLE_TRAEFIK_CERT_RESOLVER=staging
```

Enable access log https://docs.traefik.io/observability/access-logs/

```
export ENABLE_TRAEFIK_ACCESS_LOG=true
```

### Install

Clone repository

```
https://github.com/vcdocker/docker-traefik
```

Checkout 2.0 branch

```
git checkout 2.0
```

Create acme.json file to store letsencrypt certificate and make sure it has permission is 600

```
touch letsencrypt/acme.json && chmod 600 letsencrypt/acme.json
```

Create access.log file if you enable access log

```
touch logs/access.log
```

Deploy the stack with

```
docker stack deploy -c docker-compose.yaml vcrobot
```

Check if the stack was deployed with

```
docker stack ps vcrobot
```
