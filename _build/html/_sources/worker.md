# Worker

The OpenLambda **worker** is the core server-side component of a node. It listens for
incoming HTTP requests, manages container lifecycle, and returns responses to callers.

## Overview

Each worker is a standalone Go binary that exposes a single HTTP endpoint:

```
POST /runLambda/<lambda-name>
```

When a request arrives the worker:

1. Checks whether the lambda's container image is already present on the node; if not, pulls it from the registry.
2. Starts a Linux container from the image.
3. Passes the request payload to the lambda function running inside the container.
4. Waits for the function to return a result, then forwards that result back to the caller.
5. Optionally keeps the container warm for a short period to reduce cold-start latency on subsequent calls.

## Configuration

The worker is configured via a JSON file (default `config.json`) in the working directory.
Key fields:

| Field | Description | Default |
|---|---|---|
| `worker_port` | Port the HTTP server listens on | `8080` |
| `registry` | URL of the lambda registry | `""` |
| `sandbox` | Container backend (`docker` or `sock`) | `docker` |
| `log_output` | Where to write logs (`stdout` or a file path) | `stdout` |

## Starting the Worker

```bash
# From the repo root after building:
./bin/worker --config config.json
```

The worker prints its listening address on startup. You can verify it is running with:

```bash
curl -w "\n" localhost:8080/status
```

## Deploying Multiple Workers

Workers are stateless with respect to routing — each one operates independently. To scale
horizontally, start one worker process per node and place a standard HTTP load balancer
(Nginx, HAProxy, or similar) in front of them. No coordination between workers is required.

```{note}
A centralized **boss** component for cluster-wide management is currently under development.
Until then, manual deployment behind a load balancer is the recommended approach for
multi-node setups.
```

## Further Reading

- [Quickstart guide](doc.htm) — get a single worker running locally in minutes
- [SOCK: Rapid Task Provisioning with Serverless-Optimized Containers](https://www.usenix.org/conference/atc18/presentation/oakes) — the research paper describing the container backend.