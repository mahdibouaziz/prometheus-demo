# What is Prometheus?

- Prometheus was created to **monitor** highly dynamic container environments like Kubernetes, Docker Swarm, ….
- It can also be used in a traditional bare metal server non-container infrastructure.

# Where and Why is Prometheus used?

Prometheus is used to:

- **Constantly monitor** all the services.
- Send **alerts** when a service crash.
- **Identify problems before they even occur** and alert the system administrators.

Some use cases:

- Check regularly the status of the **memory usage** of the nodes.
- Check the **storage space** available on the node.

# Prometheus Architecture

The main component is the `Prometheus Server`, it does the actual monitoring work.

**Prometheus Server** is made of 3 parts:

- `Time Series Database`: stores all the **metrics** data (e.g current CPU usage, number of exceptions in an application, ….).
- `Data Retrieval Worker`: responsible for getting (**pulling**) these metrics from applications, services, servers, …. And storing them (pushing) into the Time Series Database.
- `HTTP Server`: accepts **PromQL** queries, used to display the data in a UI like **Prometheus Web UI** or **Grafana**, ….

![Alt text](./images/prom-archi.png?raw=true)

# Targets and Metrics:

- `Targets`: **what** does Prometheus monitor
  - Linux/ Windows Server
  - Apache Server
  - Single Application
  - Database
- `Metrics`: **which units** are monitored for those targets
  - CPU Status
  - Memory/Disk space usage
  - Exception Count
  - Request Count/Duration

Metrics are what gets saved in a Prometheus DB component

Metrics are saved in a Human-readable text-based format, and they have **TYPE** and **HELP** attributes.

We have 3 types:

- `Counter`: how many times does X happen?
- `Gauge`: what is the current value of X now?
- `Histogram`: How long or how big?

# How does Prometheus collect those metrics from targets?

Prometheus pulls metrics data from the target from the HTTP endpoint which by default is `hostaddress/metrics`

For that to work:

- The targets must expose the `/metrics` endpoint.
- Data must be in the **correct format** (that Prometheus understands).

# Targets endpoints and Exporters:

# Alert Manager:

# Data Storage:

# PromQL Query Language:

# Prometheus Characteristics:

# Prometheus Federation:
