# Practical Prometheus-Demo

In this demo we will discuss and learn how to use Prometheus in practice

# Discussion: How to deploy the different Prometheus parts in Kubernetes Cluster?

## 1. Creating all configuration YAML files yourself

You need to create manifest files for each component of **Prometheus Statefulset**, **Alertmanager**, **Garfana**, all the **Configmaps** and **Secrets** that you need, etc ....

and then you need to execute them in the right order (because of the dependencies)

This option is **insufficient** and its a **lost of time and effort**

## 2. Using an Operator

Think of an operator is a manager of all Prometheus components that you create (manages the combination of all components as one unit)

we just need to **find a Prometheus operator** and then **deploy it in the K8s cluster**.

## 3. Using Helm chart to deploy Operator

This is the best option to set-up Prometheus.

`Helm` will do the **initial setup** and them the `Operator` will **manage the setup**

# Start the Demo

We Assume that we have 4 Ubuntu, The Kubernetes is installed and the `nfsserver1` host in the same network with the cluster :
| Role | Hostname | IP address |
| ---------- | ---------------- | --------------- |
| Master | kubemaster | 192.168.56.18/24 |
| Worker | kubenode01 | 192.168.56.19/24 |
| Worker | kubenode02 | 192.168.56.20/24 |
| NFS Server | nfsserver1 | 192.168.56.25/24 |

and you should have **Helm** installed

# Deploy a Microservice application to our cluster for testing purposes

We'are going to use a microservice application provided by google (for learning pusposes)

This is the repository: [https://github.com/mahdibouaziz/microservices-demo-google]

just clone it and run:

Don't forget to change the service `frontend-external` to NodePort

`kubectl apply -f ./release/kubernetes-manifests.yaml`

# Deploy Prometheus Stack using Helm

This is the Prometheus Helm repository: [https://github.com/prometheus-community/helm-charts]

add the repo as follows:

`helm repo add prometheus-community https://prometheus-community.github.io/helm-charts`

update the repo

`helm repo update`

You can then run `helm search repo prometheus-community` to see the charts.

install Prometheus into its own namespace

`kubectl create namespace monitoring`

install the chart:

`helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring`

verify the deployment:

`kubectl get all -n monitoring`

![Alt text](./images/all-prometheus-stack.png?raw=true)

# Understanding Prometheus Stack Components

We have 2 **StatefulSet**:

- `prometheus-monitoring-kube-prometheus-prometheus` the Promethes Server itself, this is gonna be managed by the `Operator`
- `alertmanager-monitoring-kube-prometheus-alertmanager`

We have 3 **Deployements**:

- `monitoring-kube-state-metrics` : Created rometheus and Alertmanager StatefulSet
- `monitoring-kube-prometheus-operator` : its own Helm chart (dependency of this Helm chart) and it scrapes K8s components metrics (monitor the health of deployments, Statefulsets, Pods, .... inside the cluster)
- `monitoring-grafana`

We have 1 **DeamonSet** (runs on each node):

- `monitoring-prometheus-node-exporter`: Translates Worker Node metrics to Prometheus metrics

we have also pod, services, secrets, configmaps, ....

==> we have setup a **monitoring stack** + we get **by default** a **monitoring configuration for the K8s cluster** for our **Worker Nodes and K8s Components**

we have another interesting things that get created `CRD`

# Data Visualization

We want to notice when something **unexpected** happens, Observe any **anomalies**

- CPU spikes
- High Load
- Insufficient storage
- Unauthorized Requests

## Prometheus Web UI

We need to expose this service `monitoring-kube-prometheus-prometheus` to be able to see the Prometheus Web UI, to do that:

`kubectl patch svc monitoring-kube-prometheus-prometheus -p '{"spec": {"type": "NodePort"}}' -n monitoring`

and this is the UI:
![Alt text](./images/Prometheus%20Web%20UI.png?raw=true)

This is a simple UI, but it can be useful for getting some of the basic information about the cluster

To get the **targets** that Prometheus is monitoring: `Status` -> `Targets`

You need to add the **target**, which you want to monitor

You can check if the data you want to observe is available or not by searching it on the `Expression` on the main page of Prometheus

NOTES:

- **Instace** = an endpoint you can scrape
- **Job** = Collection of Instances with the same purpose.

## Grafana
