---
title: Storage
taxonomy:
    category: docs
---

Deployments service stores mender artifact in a S3 compatible object store. This
gives the end use a flexibility in using either an own storage (default setup)
or 3rd party services like Amazon's S3.

# Minio object store and storage proxy

The default setup described in compose file uses [Minio](https://www.minio.io/)
object storage along with a Storage Proxy service. The proxy service provides
HTTPS and traffic limiting services.

## Certificates

Storage proxy certificate needs to be mounted into the container. This can be
implemented using a `docker-compose` file with the following entry:

```
    storage-proxy:
        volumes:
            - ./ssl/storage-proxy/s3.docker.mender.io.crt:/var/www/storage-proxy/cert/cert.crt
            - ./ssl/storage-proxy/s3.docker.mender.io.key:/var/www/storage-proxy/cert/key.pem
```

Replace path to demo certificate and key with paths to your certificate and key.

Deployments service communicates with Minio object storage via storage proxy.
For this reason, `mender-deployments` service must be provisioned with a
certificate of a storage proxy for host verification purpose. This can be
implemented by adding the following entry to compose file:

```
    mender-deployments:
        volumes:
            - ./ssl/storage-proxy/s3.docker.mender.io.crt:/etc/ssl/certs/s3.docker.mender.io.crt
        environment:
            - STORAGE_BACKEND_CERT=/etc/ssl/certs/s3.docker.mender.io.crt
```

`STORAGE_BACKEND_CERT` defines a path to the certificate of storage proxy within
the container's filesystem. Deployments service will automatically load the
certificate into its trust store.

!! Make sure that the certificate matches the domain name used by other services
accessing storage proxy.

## Storage location

Minio service is configured to use `/export` directory as its storage location.
It is possible to define a volume that mounts a local directory into the service
container:

```
    mender-deployments:
        volumes:
            - /my/storage/location:/export
```

## Bandwidth and connection limiting

When a large number of devices is being updated, the uplink connection of the
backend can easily get saturated. For this reason `storage-proxy` container is
aware of 2 environment variables:

* `MAX_CONNECTIONS` - limits the number of concurrent GET requests. Integer,
  defaults to 100.

* `DOWNLOAD_SPEED` - limits the download speed of proxy response. String,
  defaults to `1m` (1MByte/s). Detailed format description is provided in nginx
  documentation
  of
  [limit_rate](https://nginx.org/en/docs/http/ngx_http_core_module.html#limit_rate) variable

These options can be adjusted using a separate compose file with the following
entry (example limiting download speed to 512kB/s, max 5 concurrent transfers):

```
    storage-proxy:
        environment:
            MAX_CONNECTIONS: 5
            DOWNLOAD_SPEED: 512k
```

# S3 storage backend

It is possible to use S3 as a storage backend in place of Minio object storage.
This can be achieved using a separate compose file with the following entry:

```
    mender-deployments:
        environment:
            AWS_ACCESS_KEY_ID: <your-aws-access-key-id>
            AWS_SECRET_ACCESS_KEY: <your-aws-secret-access-key>
            AWS_URI: https://s3.amazonaws.com
```
