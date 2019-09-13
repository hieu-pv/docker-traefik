# Vicoders Docker Traefik

Vicoders docker stack with Traefik as proxy service

- [Vicoders Docker Traefik](#vicoders-docker-traefik)
  - [Overview](#overview)
  - [How it works](#how-it-works)
  - [Installation Guide](#installation-guide)
    - [Pre Install](#pre-install)
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

Create an environment variable with your email, to be used for the generation of Let's Encrypt certificates

```
export EMAIL=admin@example.com
```

> You will access the Traefik UI at <your domain>:8080
> You will access the Portainer UI at <your domain>:9000

Create an environment variable with a username (you will use it for the HTTP Basic Auth for Traefik and Portainer), for example:

```
export USERNAME=admin
```

Create an environment variable with the password, e.g.

```
export PASSWORD=changethis
```

Use openssl to generate the "hashed" version of the password and store it in an environment variable

```
export VC_HASHED_PASSWORD=$(openssl passwd -apr1 $PASSWORD)
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

Deploy the stack with

```
docker stack deploy -c docker-compose.yaml vcrobot
```

Check if the stack was deployed with

```
docker stack ps vcrobot
```
