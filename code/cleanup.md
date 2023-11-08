
```bash
kubectl delete pods --all
kubectl delete deploy --all
kubectl delete svc --all
kubectl delete pvc --all
kubectl delete pv --all
kubectl delete jobs --all
```

Then `kubectl get all` to check.
```
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   22m
```
