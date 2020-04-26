# tektontrigger-tutorial
A tutorial to use Tekton and Knative to create a devops pipeline

## Introduction

## Prerequisites

Before you begin, you’ll need the following:

- Get [an IBM Cloud account](https://cloud.ibm.com/registration)
- Privison an [IBM Kubernetes Service cluster](https://cloud.ibm.com/kubernetes/overview), using Kubernetes version 1.16 or later, at least with 3 worker nodes.
- Create a [private container registry](https://cloud.ibm.com/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace) in IBM Container Service
- Install [Kubernetes CLI](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- Config Kubernetes CLI to manage IBM Kubernetes Service cluster on IBM Cloud by following the [instructions](https://cloud.ibm.com/docs/containers?topic=containers-cs_cli_install#cs_cli_configure)
- Set up [Tekton](https://github.com/tektoncd/pipeline/blob/master/docs/install.md#adding-the-tekton-pipelines) and [Tekton Trigger](https://github.com/tektoncd/triggers/blob/master/docs/install.md#installing-tekton-triggers-1) in your cluster
- Set up [Knative] (https://cloud.ibm.com/docs/containers?topic=containers-serverless-apps-knative#knative-setup) in your cluster
- Get [a Github account](https://github.com/)

## Estimated time

It should take you about 30 minutes to complete this tutorial.

## Steps

### 1. Tekton trigger

Tekton Trigger allows you to extract information from events payloads and then create CI/CD resources in Kubernetes. In this step, we will create a Tekton trigger to initiate a CI/CD pipleline to build and deploy a Knative service when a Github event payload is received.

#### Define secrets and service accounts

We will need to define secrets and service accounts to allow service accounts created. The complete YAML file is available at src/tekton/role-resources.yaml.

```
kubectl apply -f src/tekton/role-resources.yaml
```

#### Create a TriggerTemplate

We will create a trigger template. Before applying below yaml file, replace <namespace> with your real namespace in Cloud. The complete YAML file is available at src/tekton/triggertemplate.yaml.

```
kubectl apply -f src/tekton/triggertemplate.yaml
```

#### Create a TriggerBinding

We will create a trigger binding by applying below yaml file. The complete YAML file is available at src/tekton/triggerbinding.yaml.

```
kubectl apply -f src/tekton/triggertemplate.yaml
```

#### Create an EventListener

We will create a event listener by applying below yaml file. The complate YAML file is available at src/tekton/eventlistener.yaml.

```
kubectl apply -f src/tekton/eventlistener.yaml
```

When an event listener is successfully set up in your cluster, you will see a running pod in your namespace:

```
kubectl get pods
```

Expected output:
```
NAME                                                 READY   STATUS      RESTARTS   AGE
el-my-listener-99b595cc6-4vqq6                          1/1     Running     0          21s
```

#### Create an Ingress for EventListener

The event listener is used to accept github events payloads. So it should be able to accesse from outside of the cluster. There we have to config an ingress for this event listener to support the external access of event listener.

1. Get the Ingress Subdomain of your cluster

You could use command line tool `ibmcloud` to get the Ingress Subdomain of your cluster.

```
ibmcloud ks cluster-get $MYCLUSTER | grep 'Ingress Subdomain'
```

Expected output:
```
Ingress Subdomain: <CLUSTER-NAME>.us-south.containers.appdomain.cloud
```

2. Replace `el-my-listener.<INGRESS-SUBDOMAIN>` in src/tekton/ingress.yaml with the real Ingress Subdomain of your cluster, e.g. el-my-listener.testcluster-973348.us-south.containers.appdomain.cloud. 

3. Apply ingress.yaml to create the ingress.

The complate YAML file is available at src/tekton/ingress.yaml.
```
kubectl apply -f src/tekton/ingress.yaml
```

4. After ingress is successfully set up, you will be able to view the ingress by below command:
```
kubectl get ingress
```

Expected output:
```
NAME             HOSTS                                                              ADDRESS         PORTS   AGE
el-my-listener   el-my-listener.testcluster-973348.us-south.us-south.containers.appdomain.cloud   169.47.66.178   80      14s
```

#### Config a Webhook at your github repository



### 2. Knative Eventing

#### Create a notification service
#### Create a default broker
#### Create an ApiServerSource
#### Create a trigger

### 3. Generate a PUSH event and check results

#### Generate a PUSH event from your github repository
#### Check Tekton trigger
#### Check Tekton pipeline and pipelinerun
#### Check the notification service
#### Check your email

## Summary

## Related links

- [Tekton tutorial](https://github.com/IBM/tekton-tutorial)
- [Knative101](https://github.com/IBM/knative101/tree/master/workshop)
- [Knative Eventing 101](https://github.com/IBM/knative101-eventing)
