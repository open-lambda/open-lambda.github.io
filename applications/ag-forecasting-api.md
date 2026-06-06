---
myst:
  html_meta:
    description: "FastAPI/ASGI crop disease forecasting service from UW–Madison DSI, ported to OpenLambda."
    keywords: "AgForecast, FastAPI, ASGI, OpenLambda, crop disease, Wisconet, serverless"
---

# Agricultural Forecasting API

A FastAPI-based ASGI service for crop disease risk forecasting using multi-source weather
data, developed by the [University of Wisconsin–Madison Data Science Institute](https://dsi.wisc.edu/)
and ported to OpenLambda as a case study in deploying real-world applications.

---

## Overview

The Ag Forecasting API provides geospatial agricultural intelligence for Wisconsin,
combining weather data with peer-reviewed crop disease forecasting models. It exposes two
parallel data pipelines through a unified interface:

- **Wisconet** — the public mesonet of weather stations across the state.
- **IBM Environmental Intelligence** — point-location queries by latitude and longitude.

The core logic lives in the `ag_models_wrappers` module, which dynamically pulls the daily
and hourly weather variables each model requires for a given forecasting date, runs the risk
calculations, and returns localized predictions.

## Supported models

All models are based on peer-reviewed plant pathology research from UW–Madison:

- **Sporecaster** — white mold in soybean (dry and irrigated row-spacing variants)
- **Tarspotter** — tar spot of corn
- **Gray leaf spot** (corn)
- **Frogeye leaf spot** (soybean)

## Architecture

The service is built on FastAPI (ASGI), with a Starlette `WSGIMiddleware` wrapper so it can
also be served behind WSGI servers for legacy or mixed environments. A companion sub-package,
`pywisconet`, provides a thin REST wrapper over the Wisconet v1 API for active-station
discovery, station field metadata, and bulk measurement retrieval. IBM credentials are
supplied via environment variables (`IBM_API_KEY`, `TENANT_ID`, `ORG_ID`).

## Running on OpenLambda

Porting AgForecast surfaced five concrete challenges — writable directories, package version
pinning, GitHub deployment, asynchronous execution, and parallel pools — that drove four new
OpenLambda features: per-function environment variables, `pip-compile` as a lambda, direct
GitHub deployment, and built-in ASGI support. The full write-up is in the
[blog case study](../blog/post/2026-05-18-ag-forecasting-case-study.md).

## References

- Source: [UW-Madison-DSI/ag_forecasting_api](https://github.com/UW-Madison-DSI/ag_forecasting_api)
- Live deployment: [connect.doit.wisc.edu/ag_forecasting_api](https://connect.doit.wisc.edu/ag_forecasting_api)
- License: MIT
