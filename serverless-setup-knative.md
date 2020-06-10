# Serverless Setup using Knative
This section shows Knative setup on Ocp4 as part of 4K (K8s-Knative-Kafka-Kamel) demo.

## Step 1: Install OpenShift Serverless Operator
installation docs here [https://docs.openshift.com/container-platform/4.4/serverless/installing_serverless/installing-openshift-serverless.html](https://docs.openshift.com/container-platform/4.4/serverless/installing_serverless/installing-openshift-serverless.html)

## Step 2: Setup knative eventing from OCP Serverless Operator
installation docs here [https://docs.openshift.com/container-platform/4.4/serverless/installing_serverless/installing-knative-eventing.html#installing-knative-eventing](https://docs.openshift.com/container-platform/4.4/serverless/installing_serverless/installing-knative-eventing.html#installing-knative-eventing)

## Step 3: Setup knative serving from OCP Serverless Operator
installation docs here [https://docs.openshift.com/container-platform/4.4/serverless/installing_serverless/installing-knative-serving.html#installing-knative-serving](https://docs.openshift.com/container-platform/4.4/serverless/installing_serverless/installing-knative-serving.html#installing-knative-serving)

## Step 4: Setup knative CLI (kn) on your Desktop (optional)
installation docs here [https://docs.openshift.com/container-platform/4.4/serverless/installing_serverless/installing-kn.html#installing-kn](https://docs.openshift.com/container-platform/4.4/serverless/installing_serverless/installing-kn.html#installing-kn)

## Step 5: Deploy knative application from quay.io
use my sample Quarkus application repository for kafka client: [quay.io/efeluzy/tutorial-kafka-quarkus:v2](https://quay.io/efeluzy/tutorial-kafka-quarkus)
1. Topology > add container image. image url : quay.io/efeluzy/tutorial-kafka-quarkus:v2
2. Deploy as Serverless deployment (min pod 0 max pod 10)
3. new knative apps on topology
4. Check serverless apps using **kn** CLI
```bash
$ kn service list
```
result will be something like this
```bash
NAME    URL                                     LATEST          AGE     CONDITIONS   READY   REASON
hello   http://hello.default.apps-crc.testing   hello-gsdks-1   8m35s   3 OK / 3     True
```
