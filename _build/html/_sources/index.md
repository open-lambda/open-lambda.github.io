---
myst:
  html_meta:
    description: "OpenLambda — Apache-licensed serverless computing built on Linux containers"
    keywords: "serverless, OpenLambda, Go, Linux containers, cloud computing"
---

# OpenLambda

```{image} https://img.shields.io/badge/License-Apache%202.0-blue.svg
:alt: Apache 2.0 License
:target: https://opensource.org/licenses/Apache-2.0
```

OpenLambda is an **Apache-licensed serverless computing project**, written (mostly) in Go
and based on Linux containers. The primary goal of OpenLambda is to enable exploration of
new approaches to serverless computing. We hope to eventually make it suitable for use in
production as well.

The main system implemented so far is a **single-node OpenLambda worker** that can take
HTTP requests and invoke lambdas locally to compute responses.

You can read more about the OpenLambda worker [here](https://github.com/open-lambda/open-lambda/blob/main/docs/worker.md)
or just get started by [deploying a worker](https://github.com/open-lambda/open-lambda/blob/main/docs/quickstart.md).

```{note}
We are currently working on a **cluster mode**, where a pool of VMs running the worker
service are managed by a centralized OpenLambda boss. With a bit of work, you could also
manually deploy workers yourself and put an HTTP load balancer in front of them.
```

---

## Related Publications

```{list-table}
:header-rows: 1
:widths: 60 30 10

* - Title
  - Authors
  - Venue
* - [**Forklift: Fitting Zygote Trees for Faster Package Initialization**](https://github.com/open-lambda/open-lambda)
  - Yang et al.
  - WoSC '24
* - [**SOCK: Rapid Task Provisioning with Serverless-Optimized Containers**](https://www.usenix.org/conference/atc18/presentation/oakes)
  - Oakes et al.
  - ATC '18
* - [**Pipsqueak: Lean Lambdas with Large Libraries**](https://github.com/open-lambda/open-lambda)
  - Oakes et al.
  - ICDCSW '17
* - [**Serverless Computation with OpenLambda**](https://www.usenix.org/publications/login/fall-2016-vol-41-no-3/hendrickson)
  - Hendrickson et al.
  - ;login '16
* - [**Serverless Computation with OpenLambda**](https://www.usenix.org/conference/hotcloud16/workshop-program/presentation/hendrickson)
  - Hendrickson et al.
  - HotCloud '16
```

---

## License

This project is licensed under the **Apache License** — see the
[LICENSE.md](https://github.com/open-lambda/open-lambda/blob/main/LICENSE) file for details.