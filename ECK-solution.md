# ECK operator on Kubernetes

**Table of Contents**
- [Solution Explanation](#solution-explanation)
- [Requirements](#requirements)
- [Installation](#Installation)
  - [Installing the CRD and operator](#installing-the-crd-and-operator)
  - [Installing the elasticsearch cluster by eck operator](#installing-the-elasticsearch-cluster-by-eck-operator)

## Solution Explanation

* Best solution to manage multiple clusters
* Security feature and TLS connectivity enabled
* hot-warm-cold architectures with availability zone awareness

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
helm install es-quickstart elastic/eck-stack -n elastic-stack --create-namespace --values ./eck/values.yaml
```



