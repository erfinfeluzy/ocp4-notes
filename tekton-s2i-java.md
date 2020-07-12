# Tekton s2i Java 8

## Prereqs
```bash
$ oc login $OCP_API
$ oc new-project pipelines-tutorial
```

## Create pipeline resource for git & image repo
go to Pipelines > pipeline resource > create pipeline resource
Create Git input as resource: 
```bash
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: erfin-git
  namespace: pipelines-tutorial
spec:
  params:
    - name: url
      value: 'https://github.com/erfinfeluzy/training-rest-notif-jms'
  type: git
```
Create Image target as resource:
```bash
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: target-image
  namespace: pipelines-tutorial
spec:
  params:
    - name: url
      value: >-
        image-registry.openshift-image-registry.svc:5000/pipelines-tutorial/my-apps
  type: image
```
Check resource on OCP web console, or
```bash
$ tkn resource ls
```
result
```bash
NAME           TYPE    DETAILS
erfin-git      git     url: https://github.com/erfinfeluzy/training-rest-notif-jms
target-image   image   url: image-registry.openshift-image-registry.svc:5000/pipelines-tutorial/my-apps
```
## Create pipeline run
apiVersion: tekton.dev/v1alpha1
kind: PipelineRun
metadata:
  name: my-pipelinerun-rev2
  namespace: pipelines-tutorial
spec:
  params:
    - name: APP_NAME
      value: my-apps
  pipelineRef:
    name: my-pipeline
  resources:
    - name: git-resource
      resourceRef:
        name: erfin-git
    - name: target-image
      resourceRef:
        name: target-image
  serviceAccountName: pipeline
