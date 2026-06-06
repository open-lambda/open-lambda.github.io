---
myst:
  html_meta:
    description: "Real-world applications ported to OpenLambda, and the platform features their needs drove."
    keywords: "OpenLambda, serverless, applications, FastAPI, ASGI, WSGI"
---

# Applications

We are building OpenLambda to support a wide range of applications with minimal changes
required from those applications. Our approach is to port real-world workloads to the
platform and let their needs drive incremental improvements — rather than guessing which
features matter, we discover them by hitting real friction.

## Why applications drive the roadmap

Porting an application that was never designed for serverless surfaces concrete gaps. Each
gap becomes a candidate feature, and shipping that feature makes the next, similar
application easier to deploy. Recent work has been driven this way:

- **WSGI and ASGI support.** OpenLambda removed its dependency on Tornado and added optional
  support for both WSGI (the basis for Flask and Django) and ASGI (its async successor, used
  by FastAPI and Starlette). ASGI enables higher concurrency and better I/O utilization, but
  requires deeper runtime integration.
- **Lower onboarding friction.** Environment-variable configuration for functions and direct
  GitHub-to-OpenLambda deployment reduce the steps needed to get a new workload running.
- **Practical runtime gaps.** Real apps surface constraints such as read-only directories and
  the absence of `/dev/shm` for process pools — each a target for future work.

A recurring design question is whether to consolidate multiple functions into a single
lambda-like unit. We believe co-locating functions is beneficial: it enables opportunistic
state reuse, faster warm starts, and reduced overhead.

Looking ahead, better visibility into execution — such as detecting when an application is
blocked on I/O — opens the door to billing models that distinguish active compute from idle
waiting, addressing a longstanding serverless concern.

## Case studies

- [**Ag Forecasting API**](ag-forecasting-api.md) — a FastAPI/ASGI crop-disease forecasting
  service from the UW–Madison Data Science Institute. See the full porting write-up in the
  [blog case study](../blog/post/2026-05-18-ag-forecasting-case-study.md).
