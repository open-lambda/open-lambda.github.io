---
myst:
  html_meta:
    description: "The OpenLambda worker — the core node component that runs lambdas in SOCK sandboxes and serves HTTP, Kafka, and cron triggers."
    keywords: "OpenLambda, worker, serverless, SOCK, sandbox, HTTP, triggers, ol"
---

# Worker

The OpenLambda **worker** is the core server-side component of a node. It listens for
incoming requests, runs lambdas inside isolated sandboxes, and returns responses to callers.
The worker is part of the `ol` binary.

```{note}
This page is a high-level summary. The authoritative, version-tracked documentation lives in
the main repository under
[`docs/worker`](https://github.com/open-lambda/open-lambda/tree/main/docs/worker).
```

## Architecture

A worker is built from three layers, from the bottom up:

1. **Sandbox** — isolates lambdas from one another. This layer is pluggable; the main
   implementation today is **SOCK** (serverless-optimized containers), described in
   [Oakes et al., ATC '18](https://www.usenix.org/conference/atc18/presentation/oakes).
   Early versions used Docker.
2. **Lambda** — a *lambda instance* is a robust virtual container backed by zero or one
   sandboxes; a *lambda function* routes incoming requests to instances and decides how many
   instances to provision based on load.
3. **Event** — the trigger sources that invoke lambdas.

## Triggers

The worker supports three kinds of triggers:

- **HTTP** — a request to `http(s)://<worker-addr>:<port>/run/<lambda-name>` invokes a lambda
  directly.
- **Kafka** — a lambda can be configured to consume messages from a Kafka topic.
- **Cron** — a lambda can be invoked on a schedule.

## Building

OpenLambda is actively tested on **Ubuntu 24.04 LTS**, requires **cgroups v2**, and relies on
operations that need root privilege. After installing the
dependencies listed in the
[getting-started guide](https://github.com/open-lambda/open-lambda/blob/main/docs/worker/getting-started.md),
build the Python-only ("min") deployment:

```bash
make ol imgs/ol-min
```

## Running a worker

The `ol worker` subcommands manage a worker's lifecycle (run `./ol worker --help` for the
full list):

```bash
# Create a worker directory with a config.json and base image
./ol worker init -i ol-min

# Start the worker (use -d to run in the background)
./ol worker up -d

# Check status
./ol worker status

# Stop the worker
./ol worker down
```

`init` creates a worker directory (named `default-ol` by default) containing the worker
configuration (`config.json`), the read-only base image shared by all lambda instances, and
other resources. Per-worker settings such as memory limits are edited in `config.json`:

```json
"limits": {
    "mem_mb": 512
}
```

## Deploying a lambda

Functions can be installed directly from a Git repository, with dependencies pinned via a
`requirements.txt` and behavior configured via an `ol.yaml` file:

```bash
./ol admin install -c ol.yaml -r requirements.txt https://github.com/<org>/<repo>.git

# Invoke it
curl http://localhost:5000/run/<lambda-name>/
```

See the [Applications](applications/index.md) section and the
[Ag Forecasting case study](blog/post/2026-05-18-ag-forecasting-case-study.md) for a complete
worked example, including `ol.yaml` configuration and ASGI entry points.

## Scaling out

Workers are independent and require no coordination, so you can scale horizontally by running
one worker per node behind a standard HTTP load balancer (Nginx, HAProxy, or similar).

```{note}
A centralized **boss** component for cluster-wide management is under development. Until then,
manual deployment behind a load balancer is the recommended approach for multi-node setups.
```

## Further reading

- [Getting started](https://github.com/open-lambda/open-lambda/blob/main/docs/worker/getting-started.md) — build, deploy, and run your first lambda
- [Lambda configuration](https://github.com/open-lambda/open-lambda/blob/main/docs/worker/lambda-config.md) — the `ol.yaml` format
- [Deploying applications](https://github.com/open-lambda/open-lambda/blob/main/docs/worker/apps.md) — example app walkthroughs
- [SOCK: Rapid Task Provisioning with Serverless-Optimized Containers](https://www.usenix.org/conference/atc18/presentation/oakes) — the sandbox research paper
