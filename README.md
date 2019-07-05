# Vicoders Docker Traefik

Vicoders docker stack with Traefik as proxy service

- [Vicoders Docker Traefik](#Vicoders-Docker-Traefik)
  - [Overview](#Overview)
  - [How it works](#How-it-works)
  - [Installation Guide](#Installation-Guide)
    - [Pre Install](#Pre-Install)
    - [Install](#Install)

## Overview

This guide will show you how to set up Traefik as a load balancer/proxy and Consul to store configurations and HTTPS certificates.

## How it works

The idea is to have a main load balancer/proxy that covers all the Docker Swarm cluster and handles HTTPS certificates and requests for each domain.

But doing it in a way that allows you to have other Traefik services inside each stack without interfering with each other, to redirect based on path in the same stack (e.g. one container handles / for a web frontend and another handles /api for an API under the same domain), or to redirect from HTTP to HTTPS selectively.

## Installation Guide

### Pre Install

Create a network that will be shared with Traefik and the containers that should be accessible from the outside

```
docker network create --driver=overlay traefik-public
```

Create an environment variable with your email, to be used for the generation of Let's Encrypt certificates

```
export EMAIL=admin@example.com
```

Create an environment variable with the domain you want to use for the Traefik UI (user interface) and the Consul UI of the host, e.g.:

```
export DOMAIN=example.com
```

> You will access the Traefik UI at traefik.<your domain>, e.g. traefik.example.com and the Consul UI at consul.<your domain>, e.g. consul.example.com.
> So, make sure that your DNS records point traefik.<your domain> and consul.<your domain> to one of the IPs of the cluster.

If you have several nodes (several IP addresses), you might want to create the DNS records for multiple of those IP addresses.

That way, you would have redundancy even at the DNS level.

Create an environment variable with a username (you will use it for the HTTP Basic Auth for Traefik and Consul UIs), for example:

```
export USERNAME=admin
```

Create an environment variable with the password, e.g.

```
export PASSWORD=changethis
```

Use openssl to generate the "hashed" version of the password and store it in an environment variable

```
export HASHED_PASSWORD=$(openssl passwd -apr1 $PASSWORD)
```

Create an environment variable with the number of replicas for the Consul service (if you don't set it, by default it will be 3)

```
export CONSUL_REPLICAS=3
```

> If you have a single node, you can set CONSUL_REPLICAS to 0, that way you will only have the Consul "leader", you don't need the replicas if you don't have other nodes yet:

So, for single node

```
export CONSUL_REPLICAS=0
```

Create an environment variable with the number of replicas for the Traefik service (if you don't set it, by default it will be 3)

```
export TRAEFIK_REPLICAS=3
```

If you have a single node, you can set TRAEFIK_REPLICAS to 1

```
export TRAEFIK_REPLICAS=1
```

### Install

Create the Docker Compose file

With Jenkins

```
curl -L https://raw.githubusercontent.com/hieu-pv/docker-traefik/master/traefik.yml -o traefik.yml
```

Without Jenkins

```
curl -L https://raw.githubusercontent.com/hieu-pv/docker-traefik/without_jenkins/traefik.yml -o traefik.yml
```

Deploy the stack with

```
docker stack deploy -c traefik.yml traefik-consul
```

Check if the stack was deployed with

```
docker stack ps traefik-consul
```
