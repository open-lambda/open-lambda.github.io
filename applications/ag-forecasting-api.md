---
myst:
  html_meta:
    description: "A FastAPI/ASGI application used as a case study for porting real workloads to OpenLambda."
    keywords: "OpenLambda, serverless, FastAPI, ASGI, case study"
---

# Agricultural Forecasting API

A FastAPI-based ASGI service from the [UW–Madison Data Science Institute](https://dsi.wisc.edu/),
used here as a **case study for OpenLambda** — an existing, real-world application that was
never designed for serverless, ported to the platform to find out what OpenLambda needs.

```{note}
This page focuses on what the port taught us about OpenLambda. For the application itself,
see its [source repository](https://github.com/UW-Madison-DSI/ag_forecasting_api).
```

## Why this application

It's a good stress test for OpenLambda because it is non-trivial and ordinary: a FastAPI
(ASGI) backend with real package dependencies, weather-data caching, and parallel compute —
the kind of app teams actually run, not a serverless demo.

## What it drove in OpenLambda

Porting it surfaced five concrete challenges that drove four new OpenLambda features:

- **Per-function environment variables** — to redirect writes away from read-only directories.
- **`pip-compile` as a lambda** — to pin dependencies inside the OpenLambda environment.
- **Direct GitHub deployment** — `ol admin install` straight from a repo URL.
- **Built-in ASGI support** — OpenLambda acts as the ASGI server for FastAPI/Starlette/etc.

It also pointed to a future feature: `/dev/shm` support so process pools work under OpenLambda.

The full porting write-up is in the
[blog case study](../blog/post/2026-05-18-ag-forecasting-case-study.md).
