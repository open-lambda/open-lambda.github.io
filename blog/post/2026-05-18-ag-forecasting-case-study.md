---
blogpost: true
date: 2026-05-18
author: maria-oros, tyler-caraza-harter
category: Case Studies
tags: case-study, agforecast, asgi, fastapi, openlambda
language: en
myst:
  html_meta:
    description: "Porting a FastAPI/ASGI application to OpenLambda — five challenges and the four platform features they drove."
    keywords: "OpenLambda, serverless, FastAPI, ASGI, WSGI, pip-compile, case study"
---

# An Application Case Study: Forecasting Crop Disease with OpenLambda

*Maria Oros, Data Science Institute, University of Wisconsin–Madison*  
*Tyler Caraza-Harter, Department of Computer Sciences, University of Wisconsin–Madison*

Our goal is to make an ever-growing set of applications deployable on OpenLambda (OL), with minimal modifications. We believe the best way to work towards this goal is to pick interesting applications that weren't originally designed for serverless deployment, try to port them to OL, and identify pain points. This helps us identify the most useful features to add to OL, to support similar deployments.

Recently, we selected an agricultural forecasting application (AgForecast), developed by the [Data Science Institute at UW–Madison](https://dsi.wisc.edu/), to port to OpenLambda: <https://github.com/UW-Madison-DSI/ag_forecasting_api>. AgForecast is an interesting case study, because it implements its REST API using FastAPI, which in turn uses [ASGI](https://asgi.readthedocs.io/en/latest/), the so-called "spiritual successor" to WSGI, which we recently started supporting in OpenLambda. WSGI is the basis for popular Python web-programming packages such as Django and Flask; new ASGI support opens the door to an even broader range of applications.

In this post, we describe the challenges of porting AgForecast to OL, and four new features we added to OL to make deployment of similar applications in the future simpler. The features are built-in ASGI support, direct GitHub-to-OL deployments, OL function environment variables, and OL-based pip compilation.

## Background: Agricultural Forecasting API

**Motivation.** The agricultural forecasting app is an open-source tool designed to address farmers' needs in a customizable way providing daily crop disease forecasting predictions. We first built a backend infrastructure using FastAPI to serve crop disease forecasting models for multiple crops, focusing on integration with Wisconet weather stations and on-demand model serving. We then developed a custom python front-end interface for farmer use [here](https://github.com/UW-Madison-DSI/ag_forecasting_app_v3).

**Technical overview.** The Ag Forecasting API is a FastAPI-based backend that serves daily crop disease forecasting models across Wisconsin and plan to serve nationwide in the near future. It exposes two parallel data pipelines through a unified interface. The project ships with a `Dockerfile` and `docker-compose.yml` for containerized deployment, and includes a Starlette `WSGIMiddleware` wrapper so the FastAPI app can also be served behind WSGI servers for legacy or mixed environments. For more information, the API is MIT-licensed, fully open source, and currently deployed at [connect.doit.wisc.edu/ag_forecasting_api](https://connect.doit.wisc.edu/ag_forecasting_api).

**Fit with OpenLambda.** We began exploring OpenLambda to leverage the benefits of serverless technology. Hosting our tool on this platform offers significant value not only to the developer community, but also to plant pathology practitioners and scientists who want to keep expanding these innitiatives supported on this foundational robust infrastructure.

## Porting to OpenLambda

When porting AgForecast to OL, we encountered 5 challenges related to: expectations about writable directories, package version selection, deployment from GitHub, asynchronous execution, and parallel pool execution. To overcome these challenges, we introduced four new features to OL and made minor changes to AgForecast itself.

### Challenge 1: File Management

AgForecast is semi-stateless: data files describing stations and measurements are used across requests, but if these files are deleted, it can generate them on-the-fly from upstream data sources, such as the IBM Weather API or Wisconet. This is a good match for FaaS platforms such as OL, where lambda instances frequently persist (along with their state) across multiple invocations, even if an instance can silently be terminated at any time to reclaim memory.

However, most directory locations are read-only for an OL function; the one exception is a single "scratch directory" and various other locations that are symbolic links to the scratch directory (such as `/tmp`). Full-featured sandboxes such as Docker containers use union file systems to make many directories editable, on a copy-on-write basis. OL's limitation is due to its use of bind mounts, a leaner, but less flexible mechanism. AgForecast wasn't originally built for OL, so we modified the code to make the directory location for station and measurement data configurable via environment variables (otherwise AgForecast attempted to write to read-only locations). We added support to OL function configuration files to support the specification of environment variables, like this:

```yaml
triggers:
  http:
    - method: "*"
environment:
  MEASUREMENTS_CACHE_DIR: /host/tmp/cache
  STATIONS_CACHE_FILE: /host/tmp/cache/wisconsin_stations_cache.csv
  ...
```

### Challenge 2: Package Management

Like many Python projects, AgForecast specifies PyPI package requirements in a `requirements.txt` file. Also like most projects, not all version requirements are exact. Here are 3 of the 21 lines in AgForecast's `requirements.txt`:

```
...
matplotlib==3.9.1
fastapi>=0.95.0
pydantic
...
```

Note the different levels of specificity: `matplotlib` must be a specific version, whereas pip can select any version for `pydantic` (probably the latest, barring version conflicts based on other dependencies); `fastapi` specifies a range. Note that some of these might also have indirect dependencies on other packages not explicitly listed.

OL requires exact versions for all packages, direct or indirect. For this purpose, we recommend the use of [`pip-compile`](https://pypi.org/project/pip-tools/) to translate a partially specified `requirements.txt` file to a fully specified one, based on the latest packages at the time of compilation.

One challenge is that pip and pip-compile sometimes select packages based on the host environment. For example, in certain cases, the suitable Python package version for Ubuntu 24.04 might be different than for Ubuntu 26.04. Initially, we encountered this when deploying AgForecast on OL. The `requirements.txt` created by doing pip-compile on the host machine was not quite compatible with the environment inside the lambda function. To address this, we created a new OL function that does pip-compile inside the OL environment. It works like this:

```bash
curl -X POST -d '<some URL>' http://localhost:5000/run/pip-compile/url > requirements.txt
```

### Challenge 3: Deployment

AgForecast lives on a public GitHub repo: <https://github.com/UW-Madison-DSI/ag_forecasting_api/tree/main>. We wanted to make it as easy as possible to deploy directly from GitHub to an OL function. Thus, we added a new `ol admin install` option to point directly to a repo:

```bash
./ol admin install -c ol.yaml -r requirements.txt https://github.com/<org>/<repo>.git
```

### Challenge 4: Asynchronous Execution

AgForecast is built on FastAPI, which in turn is based on ASGI. ASGI is an asynchronous alternative to WSGI (Web Server Gateway Interface). The idea of WSGI is to let you mix and match servers (for example, Gunicorn, uWSGI) with application frameworks (for example, Flask, Django). The server/framework interface is minimalist, a single function signature that the server calls for each incoming HTTP request (GET/POST/etc). The framework implements the function; a common framework pattern is to route the call to a user-written handler function. For example, consider these two functions:

```python
app = Flask(__name__)

@app.route("/")
def home():
    return "Home page"

@app.route("/about")
def about():
    return "About page"
```

The `app` object is a Python callable (meaning it is an object that acts like a function); `app` implements the WSGI interface. So when a server sends a request to `app`, `app` in turn calls the correct user function (`home` or `about`) to execute and obtain a result.

ASGI was introduced as an alternative to WSGI to provide more options for handling concurrent calls. Consider how (in the above Flask/WSGI example) two different users may want to visit the home and about pages at the same time. Can we handle the requests concurrently?

There are a few ways to do this: multiple processes, multiple threads in a process, or [Python's async functionality introduced in Python 3.5 (2015)](https://peps.python.org/pep-0492/). If we want all execution in a single process (useful when there is shared state), we can either use threading (with WSGI) or async (with ASGI).

Threads are a non-cooperative form of scheduling, meaning that a scheduler can switch from running one thread on a CPU to another thread at any time (perhaps a very inconvenient time!). Writing multi-threaded programs is notoriously difficult, as one must identify shared state, introduce locks to protect that state, and acquire/release locks at the right points. In contrast, async offers a form of cooperative scheduling, where switches can only occur at well-defined points (e.g., an `await`). Thus, programming is simpler (no need for locks).

Normally, concurrent programs allow (a) parallel execution on multiple CPU cores at the same time and (b) execution of code at the same time that input/output occurs, say to the disk or network. In many languages, threads offer both benefits and cooperative scheduling only provides the I/O benefit. However, Python threads only offer the I/O benefit due to the GIL (Global Interpreter Lock), [though this may be changing](https://docs.python.org/3/howto/free-threading-python.html). Thus, async is especially appealing in Python since it (in theory) matches threading in terms of performance benefits, and surpasses threads in terms of ease-of-programming.

AgForecast implements REST calls in FastAPI, which is based on ASGI, which is the async-based alternative to WSGI. To support AgForecast, we implemented ASGI server functionality in OpenLambda. This makes OpenLambda an ASGI server implementation (in the same role as Uvicorn, Gunicorn, Daphne, etc). When a request arrives, OpenLambda uses `asyncio.run(...)` to invoke the user-provided entry point, with async send/receive callbacks. The user-provided entry-point can then be a full application, written in any ASGI-compatible framework (FastAPI, Starlette, Django/Channels, etc).

A user can indicate their lambda function is an ASGI application entry point by configuring their `ol.yaml` as follows:

```yaml
environment:
  OL_ENTRY_FILE: app.py
  OL_ASGI_ENTRY: app
```

### Challenge 5: Worker Pools

AgForecast indirectly uses `/dev/shm`, an in-memory file system called tmpfs. Docker containers have a `/dev/shm` mount by default; as it is frequently used for inter-process communication, its behavior is configurable via the `--ipc` flag. In contrast, OL functions do not have any `/dev/shm` mount.

Why does AgForecast need `/dev/shm`? AgForecast uses a `concurrent.futures.ProcessPoolExecutor` to run a Python compute-heavy function called `compute_risks` in parallel over different chunks of data. `ProcessPoolExecutor` creates different Python processes for different chunks of work, which is desirable as a way to get around Python's per-process GIL (Global Interpreter Lock), which is held whenever regular Python code is being executed. With multiple processes (created by the pool), each process will have its own GIL that it holds during execution; thus, multiple processes can hold their own locks at the same time and execute on multiple CPU cores in parallel.

As a fix to get AgForecast working as an OL function, we replaced `ProcessPoolExecutor` with a `ThreadPoolExecutor`; this avoids inter-process communication via `/dev/shm` because coordination occurs within a process, between threads. Unfortunately, all the threads share a GIL, so the performance benefits of using a pool for parallelism is lost in this case (the only value would be if `compute_risks` were I/O heavy, which it is not).

Using a `ThreadPoolExecutor` works as a short term fix, but this experience suggests that eventually adding `/dev/shm` availability (and thus `ProcessPoolExecutor` functionality) would be a useful future feature for OL.

## Recap of New Features

Porting real, complex applications to serverless platforms highlights the most important features to develop. In this post, we described 5 challenges we encountered when porting AgForecast to OL, and the following features we added to better support similar applications:

- environment variable configuration for lambda functions (challenge 1)
- pip-compile as a lambda function (challenge 2)
- direct GitHub deployment (challenge 3)
- ASGI support (challenge 4)

We also made some minor changes to AgForecast:

- customizable directory use for stations/measurements (challenge 1)
- use of a thread pool instead of a process pool (challenge 5)

The second change suggests a future possible OL feature: adding `/dev/shm` availability to support process pools.
