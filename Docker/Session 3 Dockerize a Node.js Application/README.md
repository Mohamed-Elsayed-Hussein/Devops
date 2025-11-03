# Session 3 — Dockerize a Node.js Application

This folder contains a small Single Page Application (SPA) served by Express, static assets, a Jest test suite, and a multi-stage `Dockerfile` that packages the app using Bun.

The repository root contains a `Jenkinsfile` which demonstrates a CI/CD pipeline that builds the Docker image from this folder and runs it on the Jenkins agent. This README explains how to build/run locally and how the Jenkins pipeline interacts with this folder (including a common image-tag mismatch and simple fixes).

## Contents

1. `Dockerfile` — multi-stage build using Bun (builder + runtime).
2. `package.json` — scripts and dependencies (`start` runs the app via Bun, `test` runs Jest).
3. `src/` — server and front-end JS (`app.js`, `imageDisplay.js`).
4. `public/` — static assets served by Express (HTML/CSS/images).
5. `tests/` — Jest tests for the front-end helpers.
6. `Jenkinsfile` — example pipeline in the repository root that references this folder.

## Prerequisites

1. Docker installed and running (required to build and run the Docker image).
2. (Optional) Bun installed locally if you want to run the app without Docker.

## Build and run locally with Docker

1. Change to this directory:

```bash
cd Docker/Session\ 3\ Dockerize\ a\ Node.js\ Application
```

2. Build the Docker image from this directory:

```bash
docker build -t my-nodejs-spa:latest .
```

3. Run the container and map port 3000:

```bash
docker run -d --name nodejs-container -p 3000:3000 my-nodejs-spa:latest
```

4. Open your browser at [http://localhost:3000](http://localhost:3000).

Notes:

- The `Dockerfile` uses Bun base images (`oven/bun`). The container exposes port `3000` and starts the app with `bun run start`.
- If you prefer per-build image tags (as used by CI), replace `:latest` with the tag you built (for example `my-nodejs-spa:123`).

## Run locally without Docker

If you have Bun installed:

```bash
bun install
```

Or with npm/Node:

```bash
npm install
npm start
```

Then open [http://localhost:3000](http://localhost:3000).

## Run tests

The project includes Jest tests in `tests/`. Run them with:

```bash
npm install
npm test
```

If you prefer Bun for development, install dev dependencies with Bun and run the `test` script defined in `package.json`.

## Jenkins CI (repository root `Jenkinsfile`)

Location: `Jenkinsfile` (the pipeline in the repository root).

What it does (relevant parts):

1. Checks out the repository.
2. Changes directory to this folder and runs:

```groovy
sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
```

3. Attempts to run the container later with `docker run ... ${IMAGE_NAME}:latest`.

Common issue: the pipeline builds the image and tags it with `${BUILD_NUMBER}`, but the `Run Container` stage tries to run `:latest`. That causes the agent to fail to find the `:latest` image unless an explicit `docker tag` or `docker push`/`pull` step was performed.

Simple fixes (pick one):

- Tag the built image as `latest` before running it (add this after the build step):

```groovy
sh "docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest"
```

- Or run the container using the build tag instead of `latest`:

```groovy
sh "docker run -d --name ${CONTAINER_NAME} -p 3000:3000 ${IMAGE_NAME}:${BUILD_NUMBER}"
```

Notes for CI agents:

- The Jenkins agent that runs this pipeline must have Docker CLI access and permission to talk to the Docker daemon (or use Docker-in-Docker). If the agent cannot run Docker commands, the pipeline will fail.
- For multi-node setups or production workflows, push the image to a registry (Docker Hub, GitHub Container Registry, etc.) and pull it on the target host rather than running containers directly on the Jenkins agent.
