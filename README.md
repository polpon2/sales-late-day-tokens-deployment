# Deployment Guide

This guide outlines the deployment process for the Late-token project on a Kubernetes cluster. We will start by deploying Signoz & RabbitMQ and then proceed with other components.

## Prerequisites

Before you begin, ensure that you have the following prerequisites in place:

- A functioning Kubernetes cluster.
- `kubectl` command-line tool properly configured to work with your cluster. (or equivalent)
- You have [traefik](https://doc.traefik.io/traefik/getting-started/install-traefik/) installed and running, https://doc.traefik.io/traefik/getting-started/install-traefik/
- [Optional] You have [signoz](https://signoz.io/docs/install/kubernetes/others/#install-signoz-on-kubernetes-with-helm) installed follow this guide, https://signoz.io/docs/install/kubernetes/others/#install-signoz-on-kubernetes-with-helm

&nbsp;

## Cloning the Deployment Repository

Begin by cloning the Token deployment repository and organizing the file tree as follows:

```bash
# Clone the deployment repository
git clone https://github.com/polpon2/sales-late-day-tokens-deployment.git

# Move into the cloned repository
cd sales-late-day-tokens-deployment

# Your file tree should look like this:
#
# sales-late-day-tokens-deployment
# ├── rabbitmq
# │   ├── cluster-operator.yml
# │   └── rabbitmq.yaml
# ├── README.md
# ├── db-deployment.yaml
# ├── db-service.yaml
# ├── dockerconfigjson.yaml (create this, instruction below)
# ├── pvc.yaml
# ├── route.yaml
# ├── token-backend-deployment.yaml
# ├── token-backend-service.yaml
# ├── token-dead-letter-deployment.yaml
# ├── token-dead-letter-service.yaml
# ├── token-delivery-deployment.yaml
# ├── token-delivery-service.yaml
# ├── token-inventory-deployment.yaml
# ├── token-inventory-service.yaml
# ├── token-manager-deployment.yaml
# ├── token-manager-service.yaml
# ├── token-order-deployment.yaml
# ├── token-order-service.yaml
# ├── token-payment-deployment.yaml
# └── token-payment-service.yaml
```
&nbsp;

## RabbitMQ Deployment

Firstly, let's deploy RabbitMQ:

```bash
# Assuming you are in the 'sales-late-day-tokens-deployment' folder

# Start rabbitmq cluster
kubectl apply -f rabbitmq/cluster-operator.yml

# Then create the cluster object named 'rabbit-mq'
kubectl apply -f rabbitmq/rabbitmq.yaml
```

&nbsp;

## Secret Configuration
Before you could pull the image from the github package, you first need a correct secret configuration for k8s.

https://dev.to/asizikov/using-github-container-registry-with-kubernetes-38fb

#### TDLR:
create & apply the secret out right
```bash
kubectl create secret docker-registry dockerconfigjson-github-com --docker-server=ghcr.io --docker-username={Your username} --docker-password={Your docker PAT (personal access token)}
```
OR create the config file so we could apply it later
```bash
kubectl create secret docker-registry dockerconfigjson-github-com --docker-server=ghcr.io --docker-username={Your username} --docker-password={Your docker PAT (personal access token)} --dry-run=client --output=yaml > ghsecret.yaml
```

&nbsp;

## Apply the rest of the services and deployment
Make sure [traefik](https://doc.traefik.io/traefik/getting-started/install-traefik/) is installed and running

```bash
# Assuming you are in the 'sales-late-day-tokens-deployment' folder
kubectl apply -f .
```

Check if all the processes is pulled successfully
```bash
watch -n 1 kubectl get all
```

Should take ~30secs - 1min for all services to start, then voila. Late Token sales backend is now running, with swagger at http://localhost/docs
&nbsp;

## Deleting Deployments
Deleting Services
```bash
# Assuming you are in the 'sales-late-day-tokens-deployment' folder
kubectl delete -f .
```

Deleting the Rabbitmq Cluster Object and Cluster itself
```bash
# Delete cluster object
kubectl delete rabbitmqclusters.rabbitmq.com rabbit-mq

# Delete cluster
kubectl delete -f rabbitmq/cluster-operator.yml
```

Stop/Star Signoz Cluster

https://signoz.io/docs/operate/kubernetes/#stopstart-signoz-cluster

```bash
# To stop SigNoz cluster:
helm -n platform uninstall "my-release"

# To start/resume SigNoz cluster:
helm -n platform install "my-release"
```
