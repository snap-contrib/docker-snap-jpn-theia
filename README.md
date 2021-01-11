# Docker for a Jupyter and Theia based playground for SNAP, snappy and snapista

This repo contains a Docker image with Jupyter and Theia IDE providing a playground for SNAP, snappy and snapista

## Building the docker image

Use `docker compose` to build this docker image:

```console
docker compose build
```

Note: the build process can take up to around 15 minutes

## Start the docker image

Use `docker compose` to use the built docker image:

```console
docker compose up
```

The open your browser at the localhost, with e.g. `0.0.0.0` or `localhost` or `127.0.0.1`.

When creating new notebooks, use the kernel `Python [conda:env notebook]`

