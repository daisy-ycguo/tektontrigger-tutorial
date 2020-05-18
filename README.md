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

Notification service is a Knative service which will receive event messages, check if the event is a `service creation` event, and send a notification message to an mail box.

You can create such a notification service by the following command after replacing `$EMAIL` with your mail box address:
```
kn service create --image docker.io/daisyycguo/servicenotifier mail-notifier --env EMAIL=$EMAIL
```
Expected output:
```
Service 'mail-notifier' successfully created in namespace 'default'.
Waiting for service 'mail-notifier' to become ready ... OK

Service URL:
http://mail-notifier-default.knative-guoyc-5290c8c8e5797924dc1ad5d1b85b37c0-0001.au-syd.containers.appdomain.cloud
```

#### Create a default broker

A default broker can be created in a namespace by labeling this namespace:

```text
kubectl label namespace default knative-eventing-injection=enabled
```

Expected output:
```
namespace/default labeled
```

After the label `knative-eventing-injection=enabled` is added to a namespace, a default broker will be created. You can check the default broker by:
```text
kubectl get broker
```

Expected output:
```
NAME      READY     REASON    HOSTNAME                                   AGE
default   True                default-broker.default.svc.cluster.local   14s
```

#### Create an ApiServerSource

Kubernetes `ApiServerSource` is a predefined event source in Knative Event. It can capture events of a Kubernetes object, e.g. creation, update, delete and etc. Here we will use `ApiServerSource` to monitor the events of Knative services.

The file `eventsource.yaml` is the definition of a `ApiServerSource` which will monitor the events of Knative services.

Run following command:
```text
cat src/knative/eventsource.yaml
```

You will see the content of `eventsource.yaml`:
```
apiVersion: sources.eventing.knative.dev/v1alpha1
kind: ApiServerSource
metadata:
  name: service-update
  namespace: default
spec:
  serviceAccountName: service-sa
  mode: Resource
  resources:
  - apiVersion: serving.knative.dev/v1alpha1
    kind: Service
  sink:
    apiVersion: eventing.knative.dev/v1alpha1
    kind: Broker
    name: default
```

There are four parts in its `spec`:
- serviceAccountName: the service account name
- resources: the Kubernetes objects which will be monitored
- sink: the destination of event messages. Here the messages will be sent to a default broker
- mode: Resource means the whole resource information will be sent via event message

Firstly run below command to create the ServiceAccount:
```
kubectl apply -f serviceaccount.yaml
```
Expected output:
```
serviceaccount/service-sa created
clusterrole.rbac.authorization.k8s.io/service-sa-watcher created
clusterrolebinding.rbac.authorization.k8s.io/service-sa-event-watcher-binding created
```

Then run below command to create the event source `service-update`:
```text
kubectl apply -f eventsource.yaml
```

Expected output:
```
apiserversource.sources.eventing.knative.dev/service-update created
```

Check if `service-update` is created by:
```text
kubectl get ApiServerSource
```

Expected output:
```
NAME             AGE
service-update   18s
```

Check the event adapter pod by:
```
kubectl get pods $(kubectl get pods --selector=eventing.knative.dev/sourceName=service-update --output=jsonpath="{.items..metadata.name}")
```

Expected output:
```
NAME                                                              READY     STATUS    RESTARTS   AGE
apiserversource-service-up-659843e6-14d0-11ea-82b7-b28431dtqgns   1/1       Running   0          99s
```

#### Create a trigger

Now we use a trigger to config service `mail-notifier` to the default broker. 

Run below command to review the content of `trigger.yaml`:
```text
cat trigger.yaml
```

Expected output:
```
apiVersion: eventing.knative.dev/v1alpha1
kind: Trigger
metadata:
  name: mytrigger
spec:
  broker: default
  filter:
    sourceAndType:
      type: dev.knative.apiserver.resource.update
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1alpha1
      kind: Service
      name: mail-notifier
```

Run below command to create trigger `mytrigger`:
```text
kubectl apply -f trigger.yaml
```

Expected output:
```
trigger.eventing.knative.dev/mytrigger created
```

Check if `mytrigger` is created by:
```text
kubectl get trigger
```

Expected output:
```
NAME        READY     REASON    BROKER    SUBSCRIBER_URI                                   AGE
mytrigger   True                default   http://mail-notifier.default.svc.cluster.local   3s
```


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
