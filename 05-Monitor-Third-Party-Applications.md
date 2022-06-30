# Monitor Third Party Applications

Until now, we have configured monitoring for our K8s Components and Resource Consumption on the Nodes.

We need also to monitor our Third-Party Application (Redis in our example) and our Own applcation (Online-google-shop).

The entire app (deployed in the [02 - Prometheus Demo Basics](./02-Prometheus-demo-basics.md)):
![Alt text](./images/application.png?raw=true)

To monitor Third-party application with Prometheus we need `Exporters` for the services.

# What are Exporters?

An `exporter` is an appication that connects to the service, gets metrics data from the service, **translates these metrics to a Prometheus understandable metrics**, and **expose** these translated metrics under `/metrics` endpoint .

When we deploy an Exporter in the cluster we need to tell Prometheus about this new Exporter. For that there is a Custom K8s Resource called `ServiceMonitor`.

# Deploy Redis Exporter

Redis exporter docs: [https://github.com/oliver006/redis_exporter]

## How we deploy the Redis Exporter in our cluster?

We are going to use Helm chart that has everthing configured for us.

Redis exporter Helm Chart: [https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-redis-exporter]

We need to customize the chart before using it:

create a `redis-values.yaml` that contains:

```yaml
serviceMonitor:
  enabled: true
  labels:
    release: monitoring # this is required to let Prometheus know the service

redisAddress: redis://[redis-service-name]:6379 #change this to your servicename (in our case `redis://redis-cart:6379`)
```

then install the chart with the customized values

`helm install redis-exporter prometheus-community/prometheus-redis-exporter -f redis-values.yaml`

list the pods to verify the exporter:

`kubectl get pods`

verify the ServiceMonitor

`kubectl get servicemonitor`

and you can see in the Prometheus UI a new target is added called `redis-exporter`

![Alt text](./images/redis-exporter.png?raw=true)

and that means that Prometheus now has `redis application metrics` you can see them in the UI

![Alt text](./images/redis-metrics.png?raw=true)
