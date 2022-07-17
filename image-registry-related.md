```
docker login -u $(oc whoami) -p $(oc whoami -t) ${REGISTRY_URL}


docker push default-route-openshift-image-registry.apps.efeluzy.sandbox726.opentlc.com/erfin-demo/quarkus-vertx:v3
```

```
skopeo --insecure-policy copy --dest-creds=efeluzy:${MY_REGISTRY_TOKEN} docker-daemon:quarkus/quarkus-rest-health:v1 docker://quay.io/efeluzy/quarkus-rest-health:v1
```

