# ECK operator on Kubernetes

**Table of Contents**
- [Solution Explanation](#solution-explanation)
- [Requirements](#requirements)
- [Installation](#Installation)
  - [Installing the CRD and operator](#installing-the-crd-and-operator)
  - [Installing the elasticsearch cluster by eck operator](#installing-the-elasticsearch-cluster-by-eck-operator)
    - [Checking the status of installation process](#checking-the-status-of-installation-process)
    - [Checking the status of elasticsearch cluster](#checking-the-status-of-elasticsearch-cluster)
- [Additional Configuration](#additional-configuration)

## Solution Explanation
* Why we can choose ECK operator
  * Best solution to manage multiple clusters and larger data volume
  * X-pack Security feature and TLS connectivity are enabled
  * hot-warm-cold architectures with availability zone awareness
* Why we can't choose ECK operator
  * If we need so much customization on the manifests, better use the private helm chart on this repository to install elasticsearch

## Requirements
* Pre-installed kubernetes cluster
* kubectl cli. You can install kubectl by following [kubectl_installation](https://kubernetes.io/docs/tasks/tools/)
* Set up related kubeconfig_file credentials on your system to connect to the kubernetes cluster. You can follow [kubeconfig_setup](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)

## Installation

* ### Installing the CRD and operator:
```bash
kubectl create -f https://download.elastic.co/downloads/eck/2.8.0/crds.yaml
kubectl apply -f https://download.elastic.co/downloads/eck/2.8.0/operator.yaml
```
* ### Installing the elasticsearch cluster by eck operator:
```bash
helm upgrade --install es-quickstart elastic/eck-stack -n elastic-stack --create-namespace --values ./eck/values.yaml
```
* ### Checking the status of the installation process:
```bash
kubectl -n elastic-stack get pods --watch
```
* ### Checking the status of elasticsearch cluster 
(you can see the number of nodes and also the status of the cluster):
```bash
kubectl -n elastic-stack port-forward elasticsearch-es-node-0 9200:9200
# get the password for connecting to the cluster (x-pack-security)
PASSWORD=$(kubectl get secret elasticsearch-es-elastic-user -n elastic-stack -o go-template='{{.data.elastic | base64decode}}')
# on a new terminal
curl -u "elastic:$PASSWORD" -k "https://localhost:9200"
```

## Additional Configuration
You can change the file ./eck/values.yaml based on the ./eck/examples.yaml and then run the following command ( [Reference link](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-stack-helm-chart.html) ):
```bash
helm upgrade --install es-quickstart elastic/eck-stack -n elastic-stack --create-namespace --values ./eck/values.yaml
```