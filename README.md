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
| Master | kubemaster | 192.168.56.2/24 |
| Worker | kubenode01 | 192.168.56.3/24 |
| Worker | kubenode02 | 192.168.56.4/24 |
| NFS Server | nfsserver1 | 192.168.56.81/24 |

and you should have **Helm** installed

# Deploy a Microservice application to our cluster for testing purposes

We'are going to use a microservice application provided by google (for learning pusposes)

This is the repository: [https://github.com/GoogleCloudPlatform/microservices-demo]

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

# Understanding Prometheus Stack Components

(7:16)
