# Session 4: the generator on GKE

## Pods after the rollout (`kubectl get pods -o wide`)

```
NAME                         READY   STATUS    RESTARTS   AGE   IP            NODE                             NOMINATED NODE   READINESS GATES
generator-594bd464d4-b7c9s   1/1     Running   0          2s    10.108.0.20   gke-freight-main-acdaf648-g198   <none>           <none>
```

## Rollout history (`kubectl rollout history deployment/generator`)

```
deployment.apps/generator
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

## What happened when I deleted the pod

I deleted the generator pod with `kubectl delete pod -l app=generator`, and within about a second a new pod with a different name (`generator-544757bfb5-5lnh4`) was Running. I never told Kubernetes to restart anything. The Deployment declares that one copy must exist, and its controller noticed that the actual state (0 pods) differed from the desired state (1 pod) and created a replacement. This loop of desired state against actual state is the core idea of Kubernetes.

## Rollout and rollback

I changed `MODE` to `air` and `RATE` to `5` in `k8s/generator.yaml` and ran `kubectl apply`. Kubernetes started the new pod before stopping the old one, and only air events appeared in the logs. `kubectl rollout undo` went back to the previous revision, and I then restored the values in the file and applied it again, because `undo` changes the cluster but not the file.