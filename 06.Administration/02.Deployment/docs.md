---
title: Deploying Mender backend
taxonomy:
    category: docs
---

Mender backend services can be deployed to production using a skeleton
provided in integration repository.

# Prerequisites

Before a Docker based setup can be made available, the services need to be
provided with required certificate, keys or configuration.

!! It is recommended to use a separate `docker-compose` file for injecting
deployment specific overrides and configuration.
Consult
[Using Compose in production](https://docs.docker.com/compose/production/) guide
for details.

A `docker-compose.demo.yml` file provides example overrides and configuration
for demo backend setup. This file can be used as reference for preparing a
production deployment.

