# Mount Volume from Config Map

This is step by step tutorial to mount volume from config map. this tutorial will use my spring-boot-sample apps

# Prerequisite Steps:
```
$ oc login $OCP_SERVER
$ oc new-project erfin-demo
$ oc project erfin-demo
$ oc new-app quay.io/efeluzy/spring-boot-sample
```

## Step 1 : Create your custom configuration
Create your custom configuration file on your local workstation.
```bash
$ mkdir myconfig
$ touch myconfig/my-application.properties
$ vim myconfig/my-application.properties
....
server.port=9090

....
```

## Step 2 : Create ConfigMap on ocp
```bash
$ oc create configmap erfin-config --from-file=myconfig/
```
Validate new configmap
```bash
$ oc describe configmap erfin-config
```
## Step 3 : Mount ConfigMap as Volume
```bash
$ oc set volume deployment/spring-boot-sample \
    --add \
    --type=configmap \
    --configmap-name=erfin-config \
    --mount-path=/myconfig
```
This step will mount configmap on pods at */myconfig* folder.
check deployment
```bash
$ oc describe deployment/spring-boot-sample

...

spring-boot-sample:
    Image:        quay.io/efeluzy/spring-boot-sample@sha256:3e02091f772c664b46599959b7268f70a24dc5c3bfa21248a2b7608356dc2a39
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:
      /myconfig from volume-h7crg (rw)
```

Check myconfig on pods:
```bash
$ oc get pods
$ oc rsh spring-boot-sample-xxxx-xxxx
$ ls -ltr /myconfig

...
total 0
lrwxrwxrwx    1 root     root            32 Jul 29 05:35 my-application.properties -> ..data/my-application.properties

```
