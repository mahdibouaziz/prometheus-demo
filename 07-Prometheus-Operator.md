# Prometheus Operator

Deploying Prometheus and the associated Alertmanger tool can be a complicated task, but there is tooling available to simplify and automate the process, such as the `Prometheus Operator` project.

In this blog post, we explain what `operators` are in general, how the `Prometheus operator` works and how to configure it to best use `Prometheus` and `Alertmanager`.

# Operators

- **Operators** are a kind of software extension to Kubernetes.
- They provide a consistent approach to handle all the application operational processes automatically, without any human intervention, which they achieve through close cooperation with the Kubernetes API.

- Operators are built on two key principles of Kubernetes: `Custom Resources` (`CRs`), implemented here by way of `Custom Resource Definitions` (`CRDs`) and `custom controllers`.

- A `CR` is an extension of the Kubernetes API that provides a place where you can store and retrieve structured data—**the desired state of your application**.

- `Custom controllers` are used to observe this `CR` and with the received information **take actions** to adjust the Kubernetes cluster to the desired state.

# The Prometheus Operator

The main purpose of this operator is to simplify and automate the configuration and management of the Prometheus monitoring stack running on a Kubernetes cluster.

Essentially it is a custom controller that monitors the new object types introduced through the following `CRDs`:

- `Prometheus`: defines the desired Prometheus deployments as a StatefulSet
- `Alertmanager`: defines a desired Alertmanager deployment
- `ServiceMonitor`: declaratively specifies how groups of Kubernetes services should be monitored
- `PodMonitor`: declaratively specifies how groups of pods should be monitored
- `Probe`: declaratively specifies how groups of ingresses or static targets should be monitored
- `PrometheusRule`: defines a desired set of Prometheus alerting and/or recording rules
- `AlertmanagerConfig`: declaratively specifies subsections of the Alertmanager configuration

# Using the Prometheus Operator

To follow the examples shown in this post, it is necessary to meet the following requirements:

- **Kubernetes cluster**
- **A web application exposing Prometheus metrics**: We’re using the [microservices-demo](https://github.com/microservices-demo), which simulates the user-facing part of an e-commerce website and exposes a /metrics endpoint for each service. Follow the [documentation](https://microservices-demo.github.io/deployment/kubernetes-start.html) to deploy it on the cluster

## Deploy the Prometheus stack

go to 02-Prometheus demo basics

you can verify the CRDs:

`kubectl get crds`

## `ServiceMonitor`

- The operator uses `ServiceMonitor` to define a set of **targets** to be monitored by Prometheus.
- It uses in the spec **label selectors** to define which Services to monitor, the namespaces to look for, and the port on which the metrics are exposed.

Example of a ServiceMonitor:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: prometheus
  labels:
    name: prometheus
spec:
  selector:
    matchLabels:
      operated-prometheus: "true"
  namespaceSelector:
    any: true
  endpoints:
    - port: web
```

In this case:

- The ServiceMonitor only matches services containing the `operated-prometheus: "true"` label which is added automatically to all the Prometheus instances.
- it will also scrapes the `port` named `web` on all the underlying endpoints.
- As the `namespaceSelector` is set to `any:true` all the services in any namespace matching the selected labels are included.

## `PodMonitor`

There could be use cases that require **scraping Pods directly**, without direct association with services (for instance scraping sidecars). The operator also includes a `PodMonitor` CR, which is used to declaratively specify groups of pods that should be monitored.

As an example, we’re using the front-end app from the **microservices-demo** project, which, as we mentioned before, simulates the user-facing part of an e-commerce website that exposes a `/metrics` endpoint.

Define a `PodMonitor` in a manifest file `podmonitor.yaml` to select only this deployment pod from the **sock-shop namespace**. Even though it could be selected using a ServiceMonitor, we’ve used a targetPort field instead. This is because the pod exposes metrics on port 8079 and doesn’t include a port name:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: front-end
  labels:
    name: front-end
spec:
  namespaceSelector:
    matchNames:
      - sock-shop
  selector:
    matchLabels:
      name: front-end
  podMetricsEndpoints:
    - targetPort: 8079
```

Prometheus now should have The front end target.

## `Alertmanager`

The Prometheus Operator also introduces an Alertmanager resource, which allows users to declaratively describe an Alertmanager cluster.

It also adds an AlertmanagerConfig CR, which allows users to declaratively describe Alertmanager configurations.

First, create an alertmanager-config.yaml file to define an `AlertmanagerConfig` resource that sends notifications to a non-existent wechat receiver and its corresponding Secret file:

```yaml
apiVersion: monitoring.coreos.com/v1alpha1
kind: AlertmanagerConfig
metadata:
  name: config-alertmanager
  labels:
    alertmanagerConfig: socks-shop
spec:
  route:
    groupBy: ["job"]
    groupWait: 30s
    groupInterval: 5m
    repeatInterval: 12h
    receiver: "wechat-socks-shop"
  receivers:
    - name: "wechat-socks-shop"
      wechatConfigs:
        - apiURL: "http://wechatserver:8080/"
          corpID: "wechat-corpid"
          apiSecret:
            name: "wechat-config"
            key: "apiSecret"
---
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: wechat-config
data:
  apiSecret: cGFzc3dvcmQK
```

Then create the alertmanager.yaml file to define the `Alertmanager` cluster:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Alertmanager
metadata:
  name: socks-shop
spec:
  replicas: 1
  alertmanagerConfigSelector:
    matchLabels:
      alertmanagerConfig: example
```

The `alertmanagerConfigSelector` field is used to select the correct AlertmanagerConfig.

If you cant to see it (just export the port to nodeport)

## `PrometheusRules`

The PrometheusRule CR supports defining one or more RuleGroups. These groups consist of a set of rule objects that can represent either of the two types of rules supported by Prometheus, recording or alerting.

As an example, create the prometheus-rule.yaml file with the following PrometheusRule that will always trigger an alert:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  creationTimestamp: null
  labels:
    prometheus: socks-shop
    role: alert-rules
  name: prometheus-example-rules
spec:
  groups:
    - name: ./example.rules
      rules:
        - alert: ExampleAlert
          expr: vector(1)
```

ref: [Article](https://blog.container-solutions.com/prometheus-operator-beginners-guide)
