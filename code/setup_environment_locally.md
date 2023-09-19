# Get where I need to be locally:
`cd GitHub`

# Section 2.1: How Kubernetes runs and manages containers

## run a Pod with a single container; the restart flag tells Kubernetes
## to create just the Pod and no other resources:
`kubectl run hello-kiamol --image=kiamol/ch02-hello-kiamol`
## can add --restart=Never if desired; see https://github.com/sixeyed/kiamol/issues/45
 
## wait for the Pod to be ready:
`kubectl wait --for=condition=Ready pod hello-kiamol`
 
## list all the Pods in the cluster:
`kubectl get pods`
 
## show detailed information about the Pod:
`kubectl describe pod hello-kiamol`

## get the basic information about the Pod:
`kubectl get pod hello-kiamol`
 
## specify custom columns in the output, selecting network details:
`kubectl get pod hello-kiamol --output custom-columns=NAME:metadata.name,NODE_IP:status.hostIP,POD_IP:status.podIP`
 
## specify a JSONPath query in the output,
## selecting the ID of the first container in the Pod:
`kubectl get pod hello-kiamol -o jsonpath='{.status.containerStatuses[0].containerID}'`

## find the Pod’s container:
`docker container ls -q --filter label=io.kubernetes.container.name=hello-kiamol`
 
## now delete that container:
`docker container rm -f $(docker container ls -q --filter label=io.kubernetes.container.name=hello-kiamol)`
 
## check the Pod status:
`kubectl get pod hello-kiamol`
 
## and find the container again:
`docker container ls -q --filter label=io.kubernetes.container.name=hello-kiamol`

## listen on port 8080 on your machine and send traffic
## to the Pod on port 80:
`kubectl port-forward pod/hello-kiamol 8080:80`
 
## now browse to http://localhost:8080
 
## when you’re done press ctrl-c to end the port forward

# 2.2 Running Pods with controllers

## create a Deployment called "hello-kiamol-2", running the same web app:
`kubectl create deployment hello-kiamol-2 --image=kiamol/ch02-hello-kiamol`
 
## list all the Pods:
`kubectl get pods`

## print the labels that the Deployment adds to the Pod:
`kubectl get deploy hello-kiamol-2 -o jsonpath='{.spec.template.metadata.labels}'`
 
## list the Pods that have that matching label:
`kubectl get pods -l app=hello-kiamol-2`

## list all Pods, showing the Pod name and labels:
`kubectl get pods -o custom-columns=NAME:metadata.name,LABELS:metadata.labels`
 
## update the "app" label for the Deployment’s Pod:
`kubectl label pods -l app=hello-kiamol-2 --overwrite app=hello-kiamol-x`
 
## fetch Pods again:
`kubectl get pods -o custom-columns=NAME:metadata.name,LABELS:metadata.labels`

## run a port forward from your local machine to the Deployment:
`kubectl port-forward deploy/hello-kiamol-2 8080:80`
 
## browse to http://localhost:8080
## when you’re done, exit with ctrl-c

# Section 2.3: Defining Deployments in application manifests

## switch from the root of the kiamol repository to the chapter 2 folder:
`cd ch02`
 
## deploy the application from the manifest file:
`kubectl apply -f pod.yaml`
```bash
pod/hello-kiamol-3 created
```
 
## list running Pods:
`kubectl get pods`
```bash
NAME                              READY   STATUS    RESTARTS   AGE
hello-kiamol                      1/1     Running   1          4d2h
hello-kiamol-2-5dbf59b864-f72vp   1/1     Running   0          4d2h
hello-kiamol-2-5dbf59b864-vk6br   1/1     Running   0          4d1h
hello-kiamol-3                    1/1     Running   0          3s
```

## deploy the application from the manifest file:
`kubectl apply -f https://raw.githubusercontent.com/sixeyed/kiamol/master/ch02/pod.yaml`
```bash
pod/hello-kiamol-3 unchanged
```

## run the app using the Deployment manifest:
`kubectl apply -f deployment.yaml`
 
## find Pods managed by the new Deployment:
`kubectl get pods -l app=hello-kiamol-4`
```bash
NAME                             READY   STATUS    RESTARTS   AGE
hello-kiamol-4-fb9d497f8-rlzg7   1/1     Running   0          4s
```

# Section 2.4: Working with applications in Pods

## check the internal IP address of the first Pod we ran: 
`kubectl get pod hello-kiamol -o custom-columns=NAME:metadata.name,POD_IP:status.podIP`
```bash
NAME           POD_IP
hello-kiamol   10.1.0.16
```
 
## run an interactive shell command in the Pod:
`kubectl exec -it hello-kiamol -- sh`
 
## inside the Pod, check the IP address:
`hostname -i`
 
## and test the web app:
`wget -O - http://localhost | head -n 4`
 
## leave the shell:
`exit`

## print the latest container logs from Kubernetes:
`kubectl logs --tail=2 hello-kiamol`
```bash
2023/09/14 17:21:04 [error] 34#34: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 127.0.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8080", referrer: "http://localhost:8080/"
127.0.0.1 - - [18/Sep/2023:20:03:20 +0000] "GET / HTTP/1.1" 200 353 "-" "Wget" "-"
```
 
## ...and compare the actual container logs--if you’re using Docker:
`docker container logs --tail=2 $(docker container ls -q --filter label=io.kubernetes.container.name=hello-kiamol)`

## make a call to the web app inside the container for the Pod we created from the Deployment YAML file: 
`kubectl exec deploy/hello-kiamol-4 -- sh -c 'wget -O - http://localhost > /dev/null'`
```bash
Connecting to localhost (127.0.0.1:80)
writing to stdout
-                    100% |********************************|   353  0:00:00 ETA
written to stdout
```
 
## and check that Pod’s logs:
`kubectl logs --tail=1 -l app=hello-kiamol-4`

## create the local directory:
`mkdir -p /tmp/kiamol/ch02`
 
## copy the web page from the Pod:
`kubectl cp hello-kiamol:/usr/share/nginx/html/index.html /tmp/kiamol/ch02/index.html`
```bash
tar: removing leading '/' from member names
```
 
## check the local file contents:
`cat /tmp/kiamol/ch02/index.html`
```html
<html>
  <body>
    <h1>
      Hello from Chapter 2!
    </h1>
    <h2>
      This is
      <a
        href="https://www.manning.com/books/learn-kubernetes-in-a-month-of-lunches"
        >Learn Kubernetes in a Month of Lunches</a
      >.
    </h2>
    <h3>By <a href="https://blog.sixeyed.com">Elton Stoneman</a>.</h3>
  </body>
</html>
```

# Section 2.5: Understanding Kubernetes resource management

## list all running Pods:
`kubectl get pods`
```bash
NAME                              READY   STATUS    RESTARTS   AGE
hello-kiamol                      1/1     Running   1          4d2h
hello-kiamol-2-5dbf59b864-f72vp   1/1     Running   0          4d2h
hello-kiamol-2-5dbf59b864-vk6br   1/1     Running   0          4d2h
hello-kiamol-3                    1/1     Running   0          14m
hello-kiamol-4-fb9d497f8-rlzg7    1/1     Running   0          12m
```
 
## delete all Pods:
`kubectl delete pods --all`
```bash
pod "hello-kiamol" deleted
pod "hello-kiamol-2-5dbf59b864-f72vp" deleted
pod "hello-kiamol-2-5dbf59b864-vk6br" deleted
pod "hello-kiamol-3" deleted
pod "hello-kiamol-4-fb9d497f8-rlzg7" deleted
```
 
## check again:
`kubectl get pods`
```bash
NAME                              READY   STATUS    RESTARTS   AGE
hello-kiamol-2-5dbf59b864-t6m95   1/1     Running   0          21s
hello-kiamol-4-fb9d497f8-7xkd7    1/1     Running   0          21s
```

## view Deployments:
`kubectl get deploy`
```bash
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
hello-kiamol-2   1/1     1            1           4d2h
hello-kiamol-4   1/1     1            1           13m
```
 
## delete all Deployments:
`kubectl delete deploy --all`
```bash
deployment.apps "hello-kiamol-2" deleted
deployment.apps "hello-kiamol-4" delete
```
 
## view Pods:
`kubectl get pods`
```bash
No resources found in default namespace.
```

## check all resources:
`kubectl get all`
```bash
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d4h
```

# Chapter 3: Connecting Pods over the network with Services

## start up your lab environment--run Docker Desktop if it's not running--and switch to this chapter’s directory in your copy of the source code:
`cd ch03`
 
## create two Deployments, which each run one Pod:
`kubectl apply -f sleep/sleep1.yaml -f sleep/sleep2.yaml`
 
## wait for the Pod to be ready:
`kubectl wait --for=condition=Ready pod -l app=sleep-2`
 
## check the IP address of the second Pod:
`kubectl get pod -l app=sleep-2 --output jsonpath='{.items[0].status.podIP}'`
```bash
10.1.0.29
```
 
## use that address to ping the second Pod from the first:
`kubectl exec deploy/sleep-1 -- ping -c 7 $(kubectl get pod -l app=sleep-2 --output jsonpath='{.items[0].status.podIP}')`
```bash
PING 10.1.0.29 (10.1.0.29): 56 data bytes
64 bytes from 10.1.0.29: seq=0 ttl=64 time=0.059 ms
64 bytes from 10.1.0.29: seq=1 ttl=64 time=0.113 ms
64 bytes from 10.1.0.29: seq=2 ttl=64 time=0.180 ms
64 bytes from 10.1.0.29: seq=3 ttl=64 time=0.385 ms
64 bytes from 10.1.0.29: seq=4 ttl=64 time=0.252 ms
64 bytes from 10.1.0.29: seq=5 ttl=64 time=0.382 ms
64 bytes from 10.1.0.29: seq=6 ttl=64 time=0.313 ms

--- 10.1.0.29 ping statistics ---
7 packets transmitted, 7 packets received, 0% packet loss
round-trip min/avg/max = 0.059/0.240/0.385 ms
```

## check the current Pod’s IP address:
`kubectl get pod -l app=sleep-2 --output jsonpath='{.items[0].status.podIP}'`
 
## delete the Pod so the Deployment replaces it:
`kubectl delete pods -l app=sleep-2`
 
## check the IP address of the replacement Pod:
`kubectl get pod -l app=sleep-2 --output jsonpath='{.items[0].status.podIP}'`
```bash
10.1.0.31
```


## deploy the Service defined in listing 3.1:
`kubectl apply -f sleep/sleep2-service.yaml`
 
## show the basic details of the Service:
`kubectl get svc sleep-2`
```bash
NAME      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
sleep-2   ClusterIP   10.98.2.165   <none>        80/TCP    6s
```
 
## run a ping command to check connectivity--this will fail:
`kubectl exec deploy/sleep-1 -- ping -c 1 sleep-2`
```bash
PING sleep-2 (10.98.2.165): 56 data bytes

--- sleep-2 ping statistics ---
1 packets transmitted, 0 packets received, 100% packet loss
command terminated with exit code 1
```

# Section 3.2: Routing traffic between Pods

## run the website and API as separate Deployments: 
`kubectl apply -f numbers/api.yaml -f numbers/web.yaml`
```bash
deployment.apps/numbers-api created
deployment.apps/numbers-web created
```
 
## wait for the Pod to be ready:
`kubectl wait --for=condition=Ready pod -l app=numbers-web`
```bash
pod/numbers-web-865c56b9d-9p4m7 condition met
```
 
## forward a port to the web app:
`kubectl port-forward deploy/numbers-web 8080:80`
 
## browse to the site at http://localhost:8080 and click the Go button--you'll see an error message
```
KIAMOL Random Number Generator
RNG service unavailable!

(Using API at: http://numbers-api/sixeyed/kiamol/master/ch03/numbers/rng)
```
 
## exit the port forward:
ctrl-c

## deploy the Service from listing 3.2:
`kubectl apply -f numbers/api-service.yaml`
```
service/numbers-api created
```

## check the Service details:
`kubectl get svc numbers-api`
```bash
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
numbers-api   ClusterIP   10.108.103.175   <none>        80/TCP    32s
```

## forward a port to the web app:
`kubectl port-forward deploy/numbers-web 8080:80`
 
## browse to the site at http://localhost:8080 and click the Go button
```
KIAMOL Random Number Generator
Here it is: 89

(Using API at: http://numbers-api/sixeyed/kiamol/master/ch03/numbers/rng)
```
82, 90, 34, 24, 16 on refresh
 
## exit the port forward:
ctrl-c


## check the name and IP address of the API Pod:
`kubectl get pod -l app=numbers-api -o custom-columns=NAME:metadata.name,POD_IP:status.podIP`
```
NAME                           POD_IP
numbers-api-7c599bfcf6-fd4qz   10.1.0.33
```
 
## delete that Pod:
`kubectl delete pod -l app=numbers-api`
```
pod "numbers-api-7c599bfcf6-fd4qz" deleted
```
 
## check the replacement Pod:
`kubectl get pod -l app=numbers-api -o custom-columns=NAME:metadata.name,POD_IP:status.podIP `
```
NAME                           POD_IP
numbers-api-7c599bfcf6-rztxn   10.1.0.34
```

## forward a port to the web app:
`kubectl port-forward deploy/numbers-web 8080:80`
 
## browse to the site at http://localhost:8080 and click the Go button
 
## exit the port forward:
ctrl-c

# Section 3.3: Routing external traffic to Pods

## deploy the LoadBalancer Service for the website--if your firewall checks 
## that you want to allow traffic, then it is OK to say yes:
`kubectl apply -f numbers/web-service.yaml`
 
## check the details of the Service:
`kubectl get svc numbers-web`
```
NAME          TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
numbers-web   LoadBalancer   10.101.94.89   localhost     8080:30122/TCP   5s
```
 
## use formatting to get the app URL from the EXTERNAL-IP field:
`kubectl get svc numbers-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8080'`
```
http://localhost:8080
```


# Section 3.4: Routing traffic outside Kubernetes

## delete the current API Service:
`kubectl delete svc numbers-api`
 
## deploy a new ExternalName Service:
`kubectl apply -f numbers-services/api-service-externalName.yaml`
 
## check the Service configuration:
`kubectl get svc numbers-api`
```
NAME          TYPE           CLUSTER-IP   EXTERNAL-IP                 PORT(S)   AGE
numbers-api   ExternalName   <none>       raw.githubusercontent.com   <none>    6s
```
 
## refresh the website in your browser and test with the Go button
```
KIAMOL Random Number Generator
RNG service unavailable!

(Using API at: http://numbers-api/sixeyed/kiamol/master/ch03/numbers/rng)
```

I think it *should* be using https://raw.githubusercontent.com/sixeyed/kiamol/master/ch03/numbers/rng.


## run the DNS lookup tool to resolve the Service name:
`kubectl exec deploy/sleep-1 -- sh -c 'nslookup numbers-api | tail -n 5'`



## remove the existing Service:
`kubectl delete svc numbers-api`
 
## deploy the headless Service:
`kubectl apply -f numbers-services/api-service-headless.yaml`
 
## check the Service:
`kubectl get svc numbers-api`
 
## check the endpoint: 
`kubectl get endpoints numbers-api`
 
## verify the DNS lookup:
`kubectl exec deploy/sleep-1 -- sh -c 'nslookup numbers-api | grep "^[^*]"'`
 
## browse to the app--it will fail when you try to get a number
```
KIAMOL Random Number Generator
RNG service unavailable!

(Using API at: http://numbers-api/sixeyed/kiamol/master/ch03/numbers/rng)
```

# Section 3.5: Understanding Kubernetes Service resolution

## show the endpoints for the sleep-2 Service:
`kubectl get endpoints sleep-2`
```
NAME      ENDPOINTS      AGE
sleep-2   10.1.0.31:80   38m
```
 
## delete the Pod:
`kubectl delete pods -l app=sleep-2`
```
pod "sleep-2-789c9f5fb8-45gtv" deleted
```
 
## check the endpoint is updated with the IP of the replacement Pod:
`kubectl get endpoints sleep-2`
```
NAME      ENDPOINTS      AGE
sleep-2   10.1.0.35:80   38m
```
 
## delete the whole Deployment:
`kubectl delete deploy sleep-2`
```
deployment.apps "sleep-2" deleted
```
 
## check the endpoint still exists, with no IP addresses:
`kubectl get endpoints sleep-2`
```
NAME      ENDPOINTS   AGE
sleep-2   <none>      39m
```


## check the Services in the default namespace:
`kubectl get svc --namespace default`

## check Services in the system namespace:
`kubectl get svc -n kube-system`

## try a DNS lookup to a fully qualified Service name:
`kubectl exec deploy/sleep-1 -- sh -c 'nslookup numbers-api.default.svc.cluster.local | grep "^[^*]"'`

## and for a Service in the system namespace:
`kubectl exec deploy/sleep-1 -- sh -c 'nslookup kube-dns.kube-system.svc.cluster.local | grep "^[^*]"'`



## delete Deployments:
kubectl delete deploy --all
```
deployment.apps "numbers-api" deleted
deployment.apps "numbers-web" deleted
deployment.apps "sleep-1" deleted
```
 
## and Services:
`kubectl delete svc --all`
```
service "kubernetes" deleted
service "numbers-api" deleted
service "numbers-web" deleted
service "sleep-2" deleted
```
 
## check what’s running:
`kubectl get all`
```
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   8s

```



