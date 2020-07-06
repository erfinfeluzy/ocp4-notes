# Deploy Quarkus Application to OpenShift 4 with Helm Chart

## Prerequisites:
1. Install helm on local workstation. example using mac
```bash
$ brew install helm

$ helm version

version.BuildInfo{Version:"v3.2.4", GitCommit:"...", GitTreeState:"dirty", GoVersion:"go1.14.3"}
```
2. login to Openshift and select designated namespace
```bash
$ oc login <$OPENSHIFT_CLUSTER_URL>

$ oc new-project my-project

$ oc project my-project
```

## Step 1: Generate Helm chart on local
```bash
$ helm create quarkus-helm
```
> This command will generate helm chart and template, with below structure
```bash
quarkus-helm
|____Chart.yaml
|____charts
|____.helmignore
|____templates
| |____deployment.yaml
| |____NOTES.txt
| |____ingress.yaml
| |____tests
| | |____test-connection.yaml
| |____service.yaml
| |____hpa.yaml
| |____serviceaccount.yaml
| |_____helpers.tpl
|____values.yaml
```

## Step 2: Modify Image Repository
Edit **values.yaml** and use quarkus sample image from my repostory on **quay.io/efeluzy/quarkus-rest-health:v1**
```bash
$ vim values.yaml

...
image:
  repository: quay.io/efeluzy/quarkus-rest-health
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "v1"
...
```
> Notes: This is by default will use port 80 for service. You can change on **service.port:8080**

### Modify Service to use Application port on 8080
``` bash
$ vim template/service.yaml

          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
```

## Step 3: Add Openshift Secure Routes to Helm Chart
We will use Openshift secure route instead of ingress controller. Add route.yaml on template folder
```bash
$ vim template/route.yaml

kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: quarkus-helm
  namespace: erfin-demo
spec:
  to:
    kind: Service
    name: quarkus-helm
    weight: 100
  port:
    targetPort: http
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
  wildcardPolicy: None

```

## Step 4: Verify Helm Chart on your local
```bash
$ helm lint ./quarkus-helm

==> Linting ./quarkus-helm
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

## Step 5: Verify Template before deployment
```bash
$ helm template ./quarkus-helm
...
---
# Source: quarkus-helm/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
...
spec:
  replicas: 1
  template:
    spec:
      serviceAccountName: RELEASE-NAME-quarkus-helm
      securityContext:
        {}
      containers:
        - name: quarkus-helm
          securityContext:
            {}
          image: "quay.io/efeluzy/quarkus-rest-health:v1"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
```
## Step 5 :  Deploy to Openshift
```bash
$ helm install quarkus-helm ./quarkus-helm
```
### Try it on your browser!
```
$ curl https://${YOUR_OCP_ROUTE}/person
...
{"name":"Erfin Feluzy","id":"1234"}

```

## Upgrade Service
if you modify helm chart and redeploy it, you can use below commands
```bash
$ helm upgrade quarkus-helm ./quarkus-helm
```
result
```bash
Release "quarkus-helm" has been upgraded. Happy Helming!
NAME: quarkus-helm
LAST DEPLOYED: Mon Jul  6 15:53:16 2020
NAMESPACE: my-project
STATUS: deployed
REVISION: 3
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace erfin-demo -l "app.kubernetes.io/name=quarkus-helm,app.kubernetes.io/instance=quarkus-helm" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace erfin-demo port-forward $POD_NAME 8080:80
```
