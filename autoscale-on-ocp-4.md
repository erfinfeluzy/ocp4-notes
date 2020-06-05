# Autoscale in OCP 4.4

## Step 1: Set resource limit on Deployment or Deployment Config
Edit deployment config yaml or deployment yaml. set this varialble
- spec.containers[].resource.limits.cpu -> max limit
- spec.containers[].resource.request.cpu -> initial vcpu
```javascript
    spec:
      containers:
        - name: training-rest-notif-jms
          image: >-
            image-registry.openshift-image-registry.svc:5000/erfin-demo/training-rest-notif-jms@sha256:30807906a9d7d6419c3d4219a6f8469728e1bfba0410009532277d798108bcbf
          ports:
            - containerPort: 8080
              protocol: TCP
            - containerPort: 8443
              protocol: TCP
            - containerPort: 8778
              protocol: TCP
          resources:
            limits:
              cpu: 200m
            requests:
              cpu: 100m
```
> set request cpu 100 milicore max 200 milicore

## Step 2: Set Autoscaler

Run below command if using dc (DeploymentConfig)
```bash
> oc autoscale dc/training-rest-notif-jms --min 2 --max 10 --cpu-percent=80
```
Or, run below command if using rc (ReplicationController) -> Deployment
```bash
> oc autoscale rc/training-rest-notif-jms --min 2 --max 10 --cpu-percent=80
```
this oc command will generate HPA (Horizontal Pod Autoscaller)
```javascript
kind: HorizontalPodAutoscaler
apiVersion: autoscaling/v2beta1
metadata:
  name: training-rest-notif-jms
  namespace: erfin-demo

...


...
spec:
  scaleTargetRef:
    kind: DeploymentConfig
    name: training-rest-notif-jms
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        targetAverageUtilization: 80
```
> We will set minimum replica 2, max replica 10. autoscale when cpu reach 80% (targetAverageUtilization) or 160 milicore

## Notes
> beware of your application cpu consumption during startup. if you set request cpu to small, hence it will scale out your pod during apps startup process.
