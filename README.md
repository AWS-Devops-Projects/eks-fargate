## Start with the eksworkshop setup:
- build a cloud9 instance: https://eksworkshop.com/020_prerequisites/workspace/
- install/configure the cli tools: https://eksworkshop.com/020_prerequisites/k8stools/
- create iam role: https://eksworkshop.com/020_prerequisites/iamrole/
- attach the iam role to cloud9: https://eksworkshop.com/020_prerequisites/ec2instance/
- update iam settings for cloud9: https://eksworkshop.com/020_prerequisites/workspaceiam/
- clone the service repos: https://eksworkshop.com/020_prerequisites/clone/
- install/configure eksctl: https://eksworkshop.com/030_eksctl/prerequisites/
- launch eks: https://eksworkshop.com/030_eksctl/launcheks/
- test cluster/configure environment: https://eksworkshop.com/030_eksctl/test/


## Deploy  sample apps:
- deploy nodejs backend: https://eksworkshop.com/beginner/050_deploy/deploynodejs/
```
cd ~/environment/ecsdemo-nodejs
kubectl apply -f kubernetes/deployment.yaml
kubectl apply -f kubernetes/service.yaml
```
- deploy crystal backend: https://eksworkshop.com/beginner/050_deploy/deploycrystal/
```
cd ~/environment/ecsdemo-crystal
kubectl apply -f kubernetes/deployment.yaml
kubectl apply -f kubernetes/service.yaml
```
- scale up the backends: https://eksworkshop.com/beginner/050_deploy/scalebackend/
```
kubectl scale deployment ecsdemo-nodejs --replicas=3
kubectl scale deployment ecsdemo-crystal --replicas=3
```

## Examine current state of things:
```
kubectl get nodes # we see our 3 managed nodes
```

## There are currently a few limitations that you should be aware of:
- There is a maximum of 4 vCPU and 30Gb memory per pod.
- Currently there is no support for stateful workloads that require persistent volumes or file systems.
- You cannot run Daemonsets, Privileged pods, or pods that use HostNetwork or HostPort.
- The only load balancer you can use is an Application Load Balancer.

## Because of that last one, we want to make sure we deploy the frontend to a different namespace:
- edit yaml to deploy to new namespace
### example diff:
```
$ git diff
diff --git a/kubernetes/deployment.yaml b/kubernetes/deployment.yaml
index 3dcd89a..aad7093 100644
--- a/kubernetes/deployment.yaml
+++ b/kubernetes/deployment.yaml
@@ -4,7 +4,7 @@ metadata:
   name: ecsdemo-frontend
   labels:
     app: ecsdemo-frontend
-  namespace: default
+  namespace: frontend
 spec:
   replicas: 1
   selector:
diff --git a/kubernetes/service.yaml b/kubernetes/service.yaml
index 19c7497..474acaa 100644
--- a/kubernetes/service.yaml
+++ b/kubernetes/service.yaml
@@ -2,12 +2,12 @@ apiVersion: v1
 kind: Service
 metadata:
   name: ecsdemo-frontend
+  namespace: frontend
```

## Create a new namespace and deploy the frontend
```
kubectl create namespace frontend # create a new namespace for the frontend service
kubectl apply -f ~/environment/ecsdemo-frontend/kubernetes/deployment.yaml # deploy frontend application
kubectl apply -f ~/environment/ecsdemo-frontend/kubernetes/service.yaml # deploy frontend service
```

## Now deploy a farage profile:
```
eksctl create fargateprofile --cluster eks-fargate-demo --namespace default
```

### example output:
```
[ℹ]  creating Fargate profile "fp-2e5e699f" on EKS cluster "eks-fargate-demo"
[ℹ]  created Fargate profile "fp-2e5e699f" on EKS cluster "eks-fargate-demo"
```

## Next redeploy the backend apis:
```
kubectl apply -f ~/environment/ecsdemo-nodejs/kubernetes/  # this scales back to 1
kubectl apply -f ~/environment/ecsdemo-crystal/kubernetes/ # this scales back to 1
kubectl scale deployment ecsdemo-nodejs --replicas=3
kubectl scale deployment ecsdemo-crystal --replicas=3
# optionally delete the remaining pods
```

## View the deployments:
```
kubectl get deployments --all-namespaces
```

## View the pods:
```
kubectl get pods --all-namespaces -o wide | column -t | awk '{ print $2, $8 }' | column -t
```

## Examine the current state of things:
```
kubectl get nodes # now we see 6 additional nodes, 1 per pod, on fargate.
```
