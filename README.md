# Elasticsearch on Kubernetes

**Table of Contents**
- [Own private helm chart solution](#own-private-helm-chart-solution)
  - [Solution Explanation](#solution-explanation)
  - [Requirements](#requirements)
  - [Installation](#Installation)
    - [Installing the cluster](#installing-the-cluster)
    - [Checking the status of installation process](#checking-the-status-of-installation-process)
    - [Checking the status of elasticsearch cluster](#checking-the-status-of-elasticsearch-cluster)
  - [Configuration](#configuration)
    - [Adding or removing a node in the cluster](#adding-or-removing-a-node-in-the-cluster)
    - [Changing java-related configuration](#changing-java-related-configuration)
  - [Upgrading](#upgrading)
  - [Next steps](#next-steps)
- [ECK operator solution](#eck-operator-solution)
# Own private helm chart solution
This solution contains installing the elasticsearch cluster by using the helm chart on this repository.
## Solution Explanation

* Making cluster nodes persistent by using StatefulSet object in kubernetes:
    * Each of the nodes on the cluster should have a unique identity and should not be identical and also they should not be interchanged.
    * Pods should retain their identities when deployed on a new host
    * By using a persistent volume claim template, each pod can get a unique PVC and volume (As we know, on Deployments in kubernetes, all replicas share the same VPC and volume)
* Enable data storage volume resiliency by using AWS EBS volumes as the storage provider for the StatefulSet
    * It's not a good idea to use hostpath-based persistent volumes because of the fact that, the pods should be only deployed on one host. And so we need a storage provider and as the current environment is AWS EKS, we can use the default storage class which is based on EBS volumes (gp2).
* Using headless service for service discovery inside the cluster to initilaize and also automatically adding and removing nodes:
    * I set up elasticsearch cluster in bare metal environments and I already know that we should connect all of the nodes in the cluster and all of them should know each other. Headless service in kubernetes, can return the list of all the related pods and then the component which was calling the headless service endpoint would decide what to do with the returned list of pods and ips. 
    * I've got the idea of using headless service for elasticseach cluster by reading the following [article](https://faun.pub/https-medium-com-thakur-vaibhav23-ha-es-k8s-7e655c1b7b61) (I just used the idea related to the service discovery from this article).
* Enabled zero-downtime deployment by using a startup probe for the pods. The startup probe would check the base URL of each node's endpoint to check if it's running or not (It would take a little bit for every node to join or rejoin the cluster in case of a new change in the cluster)
* Using helm chart to install the cluster:
  * It's useful for templating the kubernetes manifests for future changes.
  * All of the users can simply change cluster configuration or add and remove nodes of the cluster by just changing one file and it's not required for them to know kubernetes manifest files syntax.
* Installing prometheus exporter to monitor the cluster (Suppose we have a prometheus installed on the cluster)

## Requirements
* Pre-installed kubernetes cluster
* kubectl cli. You can install kubectl by following [kubectl_installation](https://kubernetes.io/docs/tasks/tools/)
* Installing helm cli by following the link [helm_installation](https://helm.sh/docs/intro/install/)
* Set up related kubeconfig_file credentials on your system to connect to the kubernetes cluster. You can follow [kubeconfig_setup](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)

## Installation
The elasticsearch cluster would be simply deployed by just using the helm chart in this repository.

* ### Installing the cluster:
```bash
helm upgrade elk-cluster src/ --install -f src/values.yaml --namespace elk-cluster --create-namespace
```
* ### Checking the status of the installation process:
```bash
kubectl -n elk-cluster get pods --watch
```
* ### Checking the status of elasticsearch cluster 
(you can see the number of nodes and also the status of the cluster):
```bash
kubectl -n elk-cluster port-forward elk-cluster-0 9200:9200
# on a new terminal
curl -i localhost:9202/_cluster/health?pretty
```

## Configuration
### Adding or removing a node in the cluster
> **WARNING** minimum number of replicas in the cluster can be set to 3

You can simply increase or decrease the number of nodes in the cluster by changing the "replicaCount" value in the file src/values.yaml and then run the following command to deploy the changes:
```bash
helm upgrade elk-cluster src/ --install -f src/values.yaml --namespace elk-cluster --create-namespace
```

### Changing java related configuration
By this chart, you can simply set the java heap size memory configuration by changing the value of "es_java_opts" in the file src/values.yaml and then running the following command:
```bash
helm upgrade elk-cluster src/ --install -f src/values.yaml --namespace elk-cluster --create-namespace
```

## Upgrading
Upgrading the version of elasticsearch can be simply done by changing the value of "imageTag" in the file src/values.yaml.
> **WARNING** Make sure to read the supported version for the upgrade and for sure take a backup of the cluster data before upgrading

## Next steps
* Enable xpack security on the cluster. This feature also requires to setup TLS connectivity between all of the nodes.
* Set up automatic backup and restore of the cluster data


## ECK operator solution
> See ECK operator solution documentation here [ECK -solution.md]


[ECK -solution.md]: ./ECK-solution.md


