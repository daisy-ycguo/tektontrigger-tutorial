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

TODO: input a few introduction of Tekton triggers

#### Define secrets and service accounts
#### Create a TriggerTemplate
#### Create a TriggerBinding
#### Create an EventListener
#### Create an Ingress for EventListener
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
