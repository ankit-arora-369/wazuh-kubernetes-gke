# Usage

This guide describes the necessary steps to deploy Wazuh on Google Kubernetes Engine.

## Pre-requisites

- Kubernetes cluster already deployed on GKE.
- Kubernetes can run on a wide range of Cloud providers and bare-metal environments, this repository focuses on [GCP](https://cloud.google.com/). It was tested using [Google GKE](https://cloud.google.com/kubernetes-engine). You should be able to:
    - Create Persistent Volumes on top of GCP Persistent Disks when using a volumeClaimTemplates
- Having at least two Kubernetes nodes in order to meet the *podAntiAffinity* policy.


## Overview

### StateFulSet and Deployments Controllers

Like a Deployment, a StatefulSet manages Pods that are based on an identical container specification, but it maintains an identity attached to each of its pods. These pods are created from the same specification, but they are not interchangeable: each one has a persistent identifier maintained across any rescheduling.

It is useful for stateful applications like databases that save the data to a persistent storage. The states of each Wazuh manager as well as Elasticsearch are desirable to maintain, so we declare them using StatefulSet to ensure that they maintain their states in every startup.

Deployments are intended for stateless use and are quite lightweight and seem to be appropriate for Kibana and Nginx, where it is not necessary to maintain the states.

### Pods

#### Wazuh master

This pod contains the master node of the Wazuh cluster. The master node centralizes and coordinates worker nodes, making sure the critical and required data is consistent across all nodes.
The management is performed only in this node, so the agent registration service (authd) and the API are placed here.

Details:
- Image: Docker Hub 'wazuh/wazuh:3.11.3_7.5.2'
- Controller: StatefulSet

#### Wazuh worker 0 / 1

These pods contain a worker node of the Wazuh cluster. They will receive the agent events.

Details:
- Image: Docker Hub 'wazuh/wazuh:3.11.3_7.5.2'
- Controller: StatefulSet


#### Elasticsearch

Elasticsearch pod. No Elasticsearch cluster is supported yet.

Details:
- Image: wazuh/wazuh-elasticsearch:3.11.3_7.5.2
- Controller: StatefulSet

#### Kibana

Kibana pod. It lets you visualize your Elasticsearch data, along with other features as the Wazuh app.

Details:
- image: Docker Hub 'wazuh/kibana:3.11.3_7.5.2'
- Controller: Deployment

#### Nginx

The nginx pod acts as a reverse proxy for a safer access to Kibana.

Details:
- image: Docker Hub 'wazuh/nginx:3.11.3_7.5.2'
- Controller: Deployment


### Services

#### Elastic stack

- wazuh-elasticsearch:
  - Communication for Elasticsearch nodes.
- elasticsearch:
  - Elasticsearch API. Used by Kibana to write/read alerts.
- wazuh-nginx:
  - Nginx proxy to access Kibana: <LoadBalancerIP>
- kibana:
  - Kibana service.

#### Wazuh

- wazuh:
  - Wazuh API: wazuh-master.your-domain.com:55000
  - Agent registration service (authd): wazuh:1515
- wazuh-workers:
  - Reporting service: wazuh-workers:1514
- wazuh-cluster:
  - Communication for Wazuh manager nodes.


## Deploy


### Step 1: Deploy Kubernetes

Deploying the Kubernetes cluster is out of the scope of this guide.

This repository focuses on [GCP](https://cloud.google.com/) but it should be easy to adapt it to another Cloud provider. In case you are using GCP, we recommend [GKE](https://cloud.google.com/kubernetes-engine/docs/quickstart).


### Step 2: Create domains to access the services

We recommend creating domains and certificates to access the services. Examples:

- service/wazuh: Wazuh API and authd registration service.
- service/wazuh-workers: Reporting service.
- service/kibana: Kibana and Wazuh app.

### Step 3: Deployment

Clone this repository to deploy the necessary services and pods.

```BASH
$ git clone https://github.com/ankit-arora-369/wazuh-kubernetes-gke.git
$ cd wazuh-kubernetes-gke
```

### Step 3.1: Wazuh namespace and StorageClass

The Wazuh namespace is used to handle all the Kubernetes elements (services, deployments, pods) necessary for Wazuh. In addition, you must create a StorageClass to use AWS EBS storage in our StateFulSet applications.

```BASH
$ kubectl apply -f base/wazuh-ns.yaml
$ kubectl apply -f base/gcp-pd-storage-class.yaml
```

### Step 3.2: Deploy Elasticsearch

Elasticsearch deployment.

Single-Node deployment:

```BASH
$ kubectl apply -f elastic_stack/elasticsearch/elasticsearch-svc.yaml
$ kubectl apply -f elastic_stack/elasticsearch/single-node/elasticsearch-api-svc.yaml
$ kubectl apply -f elastic_stack/elasticsearch/single-node/elasticsearch-sts.yaml
```

Cluster deployment:

```BASH
$ kubectl apply -f elastic_stack/elasticsearch/elasticsearch-svc.yaml
$ kubectl apply -f elastic_stack/elasticsearch/cluster/elasticsearch-api-svc.yaml
$ kubectl apply -f elastic_stack/elasticsearch/cluster/elasticsearch-data-sts.yaml
$ kubectl apply -f elastic_stack/elasticsearch/cluster/elasticsearch-master-sts.yaml
```

### Step 3.3: Deploy Kibana and Nginx

Kibana and Nginx deployment.

```BASH
$ kubectl apply -f elastic_stack/kibana/kibana-svc.yaml
$ kubectl apply -f elastic_stack/kibana/nginx-svc.yaml

$ kubectl apply -f elastic_stack/kibana/kibana-deploy.yaml
$ kubectl apply -f elastic_stack/kibana/nginx-deploy.yaml
```
You can also use it with ingress and adding a domain to it with SSL secrets. Read [GKE Ingress for HTTP(S) load balancing](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress)

### Step 3.5: Deploy Wazuh

Wazuh cluster deployment.

```BASH
$ kubectl apply -f wazuh_managers/wazuh-master-svc.yaml
$ kubectl apply -f wazuh_managers/wazuh-cluster-svc.yaml
$ kubectl apply -f wazuh_managers/wazuh-workers-svc.yaml

$ kubectl apply -f wazuh_managers/wazuh-master-conf.yaml
$ kubectl apply -f wazuh_managers/wazuh-worker-0-conf.yaml
$ kubectl apply -f wazuh_managers/wazuh-worker-1-conf.yaml

$ kubectl apply -f wazuh_managers/wazuh-master-sts.yaml
$ kubectl apply -f wazuh_managers/wazuh-worker-0-sts.yaml
$ kubectl apply -f wazuh_managers/wazuh-worker-1-sts.yaml
```

### Verifying the deployment

#### Namespace

```BASH
$ kubectl get namespaces | grep wazuh
wazuh         Active    12m
```

#### Services

```BASH
$ kubectl get services -n wazuh
NAME                  TYPE           CLUSTER-IP       EXTERNAL-IP        PORT(S)                          AGE
elasticsearch         ClusterIP      xxx.yy.zzz.24    <none>             9200/TCP                         12m
kibana                ClusterIP      xxx.yy.zzz.76    <none>             5601/TCP                         11m
wazuh                 LoadBalancer   xxx.yy.zzz.209   xxx.yy.zzz.11      1515:32623/TCP,55000:30283/TCP   9m
wazuh-cluster         ClusterIP      None             <none>             1516/TCP                         9m
wazuh-elasticsearch   ClusterIP      None             <none>             9300/TCP                         12m
wazuh-nginx           LoadBalancer   xxx.yy.zzz.223   xxx.yy.zzz.12      80:31831/TCP,443:30974/TCP       11m
wazuh-workers         LoadBalancer   xxx.yy.zzz.26    xxx.yy.zzz.123     1514:31593/TCP                   9m
```

#### Deployments

```BASH
$ kubectl get deployments -n wazuh
NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
wazuh-kibana     1         1         1            1           11m
wazuh-nginx      1         1         1            1           11m
```

#### Statefulsets

```BASH
$ kubectl get statefulsets -n wazuh
NAME                     DESIRED   CURRENT   AGE
wazuh-elasticsearch      1         1         13m
wazuh-manager-master     1         1         9m
wazuh-manager-worker-0   1         1         9m
wazuh-manager-worker-1   1         1         9m

```

#### Pods

```BASH
$ kubectl get pods -n wazuh
NAME                              READY     STATUS    RESTARTS   AGE
wazuh-elasticsearch-0             1/1       Running   0          15m
wazuh-kibana-f4d9c7944-httsd      1/1       Running   0          14m
wazuh-manager-master-0            1/1       Running   0          12m
wazuh-manager-worker-0-0          1/1       Running   0          11m
wazuh-manager-worker-1-0          1/1       Running   0          11m
wazuh-nginx-748fb8494f-xwwhw      1/1       Running   0          14m
```

#### Accessing Kibana

In case you created domain names for the services, you should be able to access Kibana using the domain.

Also, you can access using the External-IP of nginx service running.

```BASH
$ kubectl get services -o wide -n wazuh
NAME                  TYPE           CLUSTER-IP       EXTERNAL-IP                                                                       PORT(S)                          AGE       SELECTOR
wazuh-nginx           LoadBalancer   xxx.xx.xxx.xxx   xxx.xx.xxx.xxx                                      80:31831/TCP,443:30974/TCP       15m       app=wazuh-nginx
```

## Agents

### Monitoring hosts

Wazuh agents are designed to monitor hosts. Just register the agent using the registration service, then configure the agent to use the reporting service.

### Monitoring containers

In this case, we have 2 options:

- Running the agent in the container: containers are sealed and designed to run a single process. It is not practicable solution.
- Install the agent on the host: This is the option that we recommend since the agent was originally designed for this purpose.

We are researching if the agent is able to run as a *DaemonSet* container. A *DaemonSet* is a special type of Pod which is logically guaranteed to run on each Kubernetes node. This kind of agent will have access only to its container, so we should mount volumes used by other containers to monitor logs, files, etc.
