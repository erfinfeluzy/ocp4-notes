# Deploy with Camel-K using Tekton on OCP

## Prerequisite
This tutorial assumes the following requirements are met:

- OpenShift 4+ cluster and oc binary tool. See docs [here](https://try.openshift.com).
- Openshift Pipeline Operator. See docs [here](https://github.com/openshift/pipelines-tutorial).
- Camel K Client Tools - Kamel. See docs [here](https://camel.apache.org/camel-k/latest/cli/cli.html).
- Optional: Tekton CLI. See docs [here](https://github.com/tektoncd/cli).

## Step 1: Cluster setup
This step will install camel operator on your ocp cluster
```bash
kamel install --cluster-setup
```
## Step 2: Create namespace
``` bash
$ oc new-project camel-pipelines
```

## Step 3: Create permission for serviceaccounts
Use this file on [assets/camel-k-pipeline-permissions.yaml](https://raw.githubusercontent.com/erfinfeluzy/ocp4-notes/master/assets/camel-k-pipeline-permissions.yaml)
```bash
$ oc apply -f camel-k-pipeline-permissions.yaml
```

## Step 4: Create tekton pipeline
Use this file on [assets/camel-k-pipeline-task-definition.yaml](https://raw.githubusercontent.com/erfinfeluzy/ocp4-notes/master/assets/pipeline-task-definition.yaml)
```bash
$ oc apply -f camel-k-pipeline-task-definition.yaml
```
![https://camel.apache.org/camel-k/latest/_images/tekton/tekton-pipeline-definition.png](https://camel.apache.org/camel-k/latest/_images/tekton/tekton-pipeline-definition.png)

## Step 5: Trigerring a Pipeline execution
Use this file on [assets/pipeline-task-run.yaml](https://raw.githubusercontent.com/erfinfeluzy/ocp4-notes/master/assets/pipeline-task-run.yaml)
```bash
oc apply -f camel-k-pipeline-task-run.yaml
```
This files contains:
```javascript
apiVersion: tekton.dev/v1alpha1
kind: PipelineRun
metadata:
  name: camel-k-pipeline-run-1
spec:
  pipelineRef:
    name: camel-k-pipeline
  serviceAccountName: 'camel-k-pipeline' 
  resources:
    - name: source-repo
      resourceRef:
        name: camel-k-examples-git 
```
> Notes: originally it used field **serviceAccount**, changed to **serviceAccountName** due to error on tekton parsing the field serviceAccount
![https://camel.apache.org/camel-k/latest/_images/tekton/tekton-pipeline-run.png](https://camel.apache.org/camel-k/latest/_images/tekton/tekton-pipeline-run.png)

After successfully build and deploy, you can find deployed apps on Topology
![https://camel.apache.org/camel-k/latest/_images/tekton/tekton-pipeline-result.png](https://camel.apache.org/camel-k/latest/_images/tekton/tekton-pipeline-result.png)

## Step 6 (Optional) : see your deployment using Tekton cli
```bash
$ tkn pipeline ls
```
result will be something like this
```bash
NAME               AGE          LAST RUN                  STARTED      DURATION    STATUS
camel-k-pipeline   1 hour ago   camel-k-pipeline-d4vyhr   1 hour ago   3 minutes   Succeeded
```
## Source
- https://camel.apache.org/camel-k/latest/tutorials/tekton/tekton.html#_triggering_a_pipeline_execution
