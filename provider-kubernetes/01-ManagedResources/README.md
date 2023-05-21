## ManagedResources
Managed resources are the kubernetes representation of the external objects we want to create.

Leaening Goals:
* `forProvider` and `atProvider`
* `deletionPolicy`

NOTE: start two watches to follow along: 
- `watch -n 1 kubectl get pods -n default` to follow the resources create by crossplane
- `watch -n 1 kubectl get crossplane` to follow the crossplane resources

create a Managed Resource:
```shell
kubectl apply -f simple.deploy.yaml
kubectl get pods -n default
```
Expected: The external resource (`Pod`) is being created

```shell
kubectl describe object.kubernetes.crossplane.io kubernetes-provider-pod
kubectl get object kubernetes-provider-yaml
```
Expected: We can see the manifest of our external resource in `atProvider`.

add a label to the managed resource
```shell
kubectl patch object.kubernetes.crossplane.io kubernetes-provider-pod -p '{"spec":{"forProvider":{"manifest":{"metadata":{"labels":{"changes":"propagate"}}}}}}'
kubect describe pod -n default pod
```
Expected: `forProvider` is to `atProvider` what `spec` is to `status`. The change propagated, the provider reconciled the external resource.

remove the labels from the external resource
```shell
kubectl label pods -n default pod changes- example-
kubect describe pod -n default pod
```
Expected: The labels are added again by the provider-kubernetes. Reconciliation works in both directions.

delete the external resource
```shell
kubectl delete pod -n default pod
kubectl get pods -n default
```
Expected: The external resource vanishes and gets recreated.

delete the managed resource:
```shell
kubectl delete object.kubernetes.crossplane.io kubernetes-provider-pod
kubectl get pods -n default
```
Expected: The external resource vanishes as well

create another managed resource
```shell
kubectl apply -f simple.deploy.yaml
kubectl get deployments -n default
```
Expected: An external resource (`Deployment`) was created.

delete the managed resource
```shell
kubectl delete object.kubernetes.crossplane.io kubernetes-provider-deploy
kubectl get deployments -n default
```
Expected: The external resource stays behind

The difference lies in the `deletionPolicy`. The managed resource for the `Deployment` had a `deletionPolicy` of `Orphan`, so the provider will not try to delete the external resource.