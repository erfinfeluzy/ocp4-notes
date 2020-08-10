# Serverless Canary Deployment

This is step by step instruction for demonstrating Canary Deployment with knative. 
Application in this demo is using my custom apps on Quay, consume Kafka Topik and stream to REST API.

## Prerequisite
- Install OpenShift Serveless or knative using operator
- Install knative (kn) CLI

## Step 0: Login to Openshift CLI and choose designated namespace
```bash
$ oc login

$ oc project my-project
```
## Step 1: Deploy knative apps
```bash
$ kn service create quarkus-kafka-consumer --image quay.io/efeluzy/quarkus-kafka-consumer:v3
```

## Step 2: Revise knative apps
```bash
$ kn service update quarkus-kafka-consumer --image quay.io/efeluzy/quarkus-kafka-consumer:latest
```

## Step 3: Distribute service traffic
Using knative CLI
```bash
$ kn revision list
```
choose revision
```bash
$ kn service update quarkus-kafka-consumer --traffic quarkus-kafka-consumer-abcd=10 --traffic quarkus-kafka-consumer-vwxyz=90
```
or using Openshift web console:
![ocp web console](https://github.com/erfinfeluzy/ocp4-notes/blob/master/screenshot/serverless-ocp-traffic-distribution.png?raw=true)
> Note: 
> - **quarkus-kafka-consumer-vwxyz** is previous service revision tag name.
> - new service will serve 10%, previous service will serve 90%

## Result on Openshift Console
![ocp web console](https://github.com/erfinfeluzy/ocp4-notes/blob/master/screenshot/serverless-canary-result.png?raw=true)

## Step 4: Check service status
```bash
$ kn service describe quarkus-kafka-consumer
```
Result
```bash
Name:         quarkus-kafka-consumer
Namespace:    erfin-serverless-demo
Labels:       app.kubernetes.io/component=quarkus-kafka-consumer, app.kubernetes.io/instance=quarkus-k ...
Annotations:  openshift.io/generated-by=OpenShiftWebConsole
Age:          7h
URL:          http://quarkus-kafka-consumer-erfin-serverless-demo.apps.erfin-cluster.sandbox1459.opentlc.com

Revisions:
   50%  quarkus-kafka-consumer-tmjzr-3 (current @latest) #v3 [3] (12m)
        Image:  quay.io/efeluzy/quarkus-kafka-consumer:latest (at 09d28f)
   50%  quarkus-kafka-consumer-mjmr9 #latest [1] (7h)
        Image:  image-registry.openshift-image-registry.svc:5000/erfin-serverless-demo/quarkus-kafka-consumer:v3 (at 09d28f)

Conditions:
  OK TYPE                   AGE REASON
  ++ Ready                  10m
  ++ ConfigurationsReady    12m
  ++ RoutesReady            10m
  ```
