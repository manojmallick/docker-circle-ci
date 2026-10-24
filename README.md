# docker-circle-ci

A 2020 sample of a Docker-based Node.js app with a CircleCI pipeline that deploys to a server over SSH. The app is a page-view counter backed by Redis.

> **Status: historical sample.** It uses `node:latest`, `docker-compose` v1 file syntax, CircleCI 2.0 `machine` executors and an old Express and Redis client. Treat it as a reference for the idea, not a template to deploy as it is.

## What it does

- `app/index.js`: an Express server on port 3000. Each request to `/` increments a `visits` counter in Redis and replies `You have viewed this page N times!`.
- `docker-compose.yml`: runs the `web` service (built from `./app`, published on port 80) and a `redis` service (port 6379).
- `.circleci/config.yml`: on each push it checks out the code, starts the `web` container, then runs `deploy.sh` on a remote server over SSH.
- `deploy.sh`: runs on the remote host; clones this repo, installs docker-compose and runs `docker-compose up -d`.

## Run it locally

Requires Docker and Docker Compose.

```bash
docker-compose up --build
# open http://localhost  (the counter increases on each refresh)
```

## Configure the pipeline

The CircleCI job expects:

| Setting | Where | Purpose |
|---|---|---|
| `DEPLOY` | CircleCI project environment variable | A command run during the build |
| SSH key for the deploy user | CircleCI project SSH keys | Lets the job connect to the server |
| Deploy host | `.circleci/config.yml` | **Hard-coded** `ec2-user@ec2-...compute-1.amazonaws.com`. Replace it with your own host (ideally from an environment variable) before using the config |

## Known limitations

- The deploy host is hard-coded in the CI config.
- `deploy.sh` downloads docker-compose 1.4.2 and runs as a script on the server without any verification.
- `app/package.json` lists `mocha` but there are no tests, and the pipeline does not run any.
- Dependencies are several years old and unpinned in the Dockerfile (`node:latest`).
