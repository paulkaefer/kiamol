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
`kubectl delete deploy --all`
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


# Chapter 4: Configuring applications with ConfigMaps and Secrets

## Section 4.1: How Kubernetes supplies configuration to apps
## switch to the exercise directory for this chapter:
`cd ch04`
 
## deploy a Pod using the sleep image with no extra configuration:
`kubectl apply -f sleep/sleep.yaml`
```
deployment.apps/sleep created
```

## wait for the Pod to be ready:
`kubectl wait --for=condition=Ready pod -l app=sleep`
```
pod/sleep-8648c6f777-dbbpg condition met
```
 
## check some of the environment variables in the Pod container:
`kubectl exec deploy/sleep -- printenv HOSTNAME KIAMOL_CHAPTER`
```
sleep-8648c6f777-dbbpg
command terminated with exit code 1
```


## update the Deployment:
`kubectl apply -f sleep/sleep-with-env.yaml`
```
deployment.apps/sleep configured
```
 
## check the same environment variables in the new Pod:
`kubectl exec deploy/sleep -- printenv HOSTNAME KIAMOL_CHAPTER`
```
sleep-5b67d77966-f2dmt
04
```


## create a ConfigMap with data from the command line:
`kubectl create configmap sleep-config-literal --from-literal=kiamol.booksection='4.1415'`
```
configmap/sleep-config-literal created
```
 
## check the ConfigMap details:
`kubectl get cm sleep-config-literal`
```
NAME                   DATA   AGE
sleep-config-literal   1      9s
```
 
## show the friendly description of the ConfigMap:
`kubectl describe cm sleep-config-literal`
```
Name:         sleep-config-literal
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
kiamol.booksection:
----
4.1415

BinaryData
====

Events:  <none>
```
 
## deploy the updated Pod spec from listing 4.2:
`kubectl apply -f sleep/sleep-with-configMap-env.yaml`
```
deployment.apps/sleep configured
```
 
## check the Kiamol environment variables:
`kubectl exec deploy/sleep -- sh -c 'printenv | grep "^KIAMOL"'`
```
KIAMOL_CHAPTER=04
```

## Section 4.2: Storing and using configuration files in ConfigMaps


## load an environment variable into a new ConfigMap:
`kubectl create configmap sleep-config-env-file --from-env-file=sleep/ch04.env`
```
configmap/sleep-config-env-file created
```
 
## check the details of the ConfigMap:
`kubectl get cm sleep-config-env-file`
```
NAME                    DATA   AGE
sleep-config-env-file   4      13s
```
 
## update the Pod to use the new ConfigMap:
`kubectl apply -f sleep/sleep-with-configMap-env-file.yaml`
```
deployment.apps/sleep configured
```

## check the values in the container:
`kubectl exec deploy/sleep -- sh -c 'printenv | grep "^KIAMOL"'`
Not the same results as the book...
```
KIAMOL_CHAPTER=04
```


## deploy the app with a Service to access it:
`kubectl apply -f todo-list/todo-web.yaml`
```
deployment.apps/todo-web created
```
 
## wait for the Pod to be ready:
`kubectl wait --for=condition=Ready pod -l app=todo-web`
```
pod/todo-web-659fff7795-kxrfq condition met
```
 
## get the address of the app:
`kubectl get svc todo-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8080'`
```
http://localhost:8080
```

## browse to the app and have a play around then try browsing to /config
Doesn't work for me... `ERR_CONNECTION_REFUSED`.
 
## check the application logs:
`kubectl logs -l app=todo-web`
```
         at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeNextResourceFilter>g__Awaited|25_0(ResourceInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
         at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.Rethrow(ResourceExecutedContextSealed context)
         at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.Next(State& next, Scope& scope, Object& state, Boolean& isCompleted)
         at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.InvokeFilterPipelineAsync()
      --- End of stack trace from previous location ---
         at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeAsync>g__Awaited|17_0(ResourceInvoker invoker, Task task, IDisposable scope)
         at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeAsync>g__Awaited|17_0(ResourceInvoker invoker, Task task, IDisposable scope)
         at Microsoft.AspNetCore.Routing.EndpointMiddleware.<Invoke>g__AwaitRequestTask|6_0(Endpoint endpoint, Task requestTask, ILogger logger)
         at Microsoft.AspNetCore.Diagnostics.StatusCodePagesMiddleware.Invoke(HttpContext context)
         at Microsoft.AspNetCore.Server.Kestrel.Core.Internal.Http.HttpProtocol.ProcessRequests[TContext](IHttpApplication`1 application)
```

## create the JSON ConfigMap:
`kubectl apply -f todo-list/configMaps/todo-web-config-dev.yaml`
```
configmap/todo-web-config-dev created
```
 
## update the app to use the ConfigMap:
`kubectl apply -f todo-list/todo-web-dev.yaml`
```
deployment.apps/todo-web configured
```
 
## refresh your web browser at the /config page for your Service 
This works! Just the config + diagnostics pages. Diagnostics details:
```
Hostname  .NET Version  OS Architecture OS Description
todo-web-b6d6f9ff5-rg5dz  .NET 6.0.8  Arm64 Linux 6.3.13-linuxkit #1 SMP PREEMPT Thu Sep 7 07:48:47 UTC 2023
```

Source code link: https://github.com/sixeyed/kiamol/tree/master/ch04/docker-images/todo-list


# Section 4.3: Surfacing configuration data from ConfigMaps

## show the default config file:
`kubectl exec deploy/todo-web -- sh -c 'ls -l /app/app*.json'`
```
-rw-r--r--    1 root     root           469 Jun 26  2022 /app/appsettings.json
```
 
## show the config file in the volume mount:
`kubectl exec deploy/todo-web -- sh -c 'ls -l /app/config/*.json'`
```
lrwxrwxrwx    1 root     root            18 Sep 21 17:31 /app/config/config.json -> ..data/config.json
```
 
## check it really is read-only:
`kubectl exec deploy/todo-web -- sh -c 'echo ch04 >> /app/config/config.json'`
```
sh: can't create /app/config/config.json: Read-only file system
command terminated with exit code 1
```

## check the current app logs:
`kubectl logs -l app=todo-web`
Errors...
 
## deploy the updated ConfigMap:
`kubectl apply -f todo-list/configMaps/todo-web-config-dev-with-logging.yaml`
```
configmap/todo-web-config-dev configured
```
 
## wait for the config change to make it to the Pod:
`sleep 120`
 
## check the new setting:
`kubectl exec deploy/todo-web -- sh -c 'ls -l /app/config/*.json'`
```
lrwxrwxrwx    1 root     root            18 Sep 21 17:31 /app/config/config.json -> ..data/config.json
lrwxrwxrwx    1 root     root            19 Sep 21 17:37 /app/config/logging.json -> ..data/logging.json
```
 
## load a few pages from the site at your Service IP address
 
## check the logs again:
`kubectl logs -l app=todo-web`
More errors...

## deploy the badly configured Pod:
`kubectl apply -f todo-list/todo-web-dev-broken.yaml`
```
deployment.apps/todo-web configured
```
 
## browse back to the app and see how it looks

## check the app logs:
`kubectl logs -l app=todo-web`
Errors...
 
## and check the Pod status:
`kubectl get pods -l app=todo-web`
```
NAME                        READY   STATUS             RESTARTS      AGE
todo-web-76b6c46bf6-554nz   0/1     CrashLoopBackOff   1 (13s ago)   15s
todo-web-b6d6f9ff5-rg5dz    1/1     Running            0             11m
```

## apply the change:
`kubectl apply -f todo-list/todo-web-dev-no-logging.yaml`
```
deployment.apps/todo-web configured
```
 
## list the config folder contents:
`kubectl exec deploy/todo-web -- sh -c 'ls /app/config'`
```
config.json
```
 
## now browse to a few pages on the app
Broken except /config and /diagnostics.
 
## check the logs:
`kubectl logs -l app=todo-web`
Errors...
 
## and check the Pods:
`kubectl get pods -l app=todo-web`
```
NAME                        READY   STATUS    RESTARTS   AGE
todo-web-85c477b74c-fqw4c   1/1     Running   0          46s
```


# Section 4.4: Configuring sensitive data with Secrets


## FOR WINDOWS USERS--this script adds a Base64 command to your session: 
`. .\base64.ps1`
 
## now create a secret from a plain text literal:
`kubectl create secret generic sleep-secret-literal --from-literal=secret=shh...`
```
secret/sleep-secret-literal created
```
 
## show the friendly details of the Secret:
`kubectl describe secret sleep-secret-literal`
```
Name:         sleep-secret-literal
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
secret:  6 bytes
````
 
## retrieve the encoded Secret value:
`kubectl get secret sleep-secret-literal -o jsonpath='{.data.secret}'`
```
c2hoLi4u
```
 
## and decode the data:
`kubectl get secret sleep-secret-literal -o jsonpath='{.data.secret}' | base64 -d`
```
shh...
```

## update the sleep Deployment:
`kubectl apply -f sleep/sleep-with-secret.yaml`
```
deployment.apps/sleep configured
```
 
## check the environment variable in the Pod:
`kubectl exec deploy/sleep -- printenv KIAMOL_SECRET`
```
shh...
```

## deploy the Secret:
`kubectl apply -f todo-list/secrets/todo-db-secret-test.yaml`
```
secret/todo-db-secret-test created
```
 
## check the data is encoded:
`kubectl get secret todo-db-secret-test -o jsonpath='{.data.POSTGRES_PASSWORD}'`
```
a2lhbW9sLTIqMio=
```

echo "a2lhbW9sLTIqMio=" | base64 -d
```
kiamol-2*2*
```

 
## see what annotations are stored:
kubectl get secret todo-db-secret-test -o jsonpath='{.metadata.annotations}'
```
{"kubectl.kubernetes.io/last-applied-configuration":"{\"apiVersion\":\"v1\",\"kind\":\"Secret\",\"metadata\":{\"annotations\":{},\"name\":\"todo-db-secret-test\",\"namespace\":\"default\"},\"stringData\":{\"POSTGRES_PASSWORD\":\"kiamol-2*2*\"},\"type\":\"Opaque\"}\n"}
```


## deploy the YAML from listing 4.13
`kubectl apply -f todo-list/todo-db-test.yaml`
```
deployment.apps/todo-db created
```
 
## check the database logs:
`kubectl logs -l app=todo-db --tail 1`
```
Error from server (BadRequest): container "db" in pod "todo-db-8b974978c-txzpf" is waiting to start: ContainerCreating
```
...ran that again:
```
2023-09-21 17:50:37.123 UTC [1] LOG:  database system is ready to accept connections
```
 
## verify the password file permissions:
`kubectl exec deploy/todo-db -- sh -c 'ls -l $(readlink -f /secrets/postgres_password)'`
```
-r--------    1 root     root            11 Sep 21 17:50 /secrets/..2023_09_21_17_50_27.776472020/postgres_password
```

## the ConfigMap configures the app to use Postgres:
`kubectl apply -f todo-list/configMaps/todo-web-config-test.yaml`
```
configmap/todo-web-config-test created
```

## the Secret contains the credentials to connect to Postgres:
`kubectl apply -f todo-list/secrets/todo-web-secret-test.yaml`
```
secret/todo-web-secret-test created
```
 
## the Deployment Pod spec uses the ConfigMap and Secret:
`kubectl apply -f todo-list/todo-web-test.yaml`
```
deployment.apps/todo-web-test created
```
 
## check the database credentials are set in the app:
`kubectl exec deploy/todo-web-test -- cat /app/secrets/secrets.json`
```
{
  "ConnectionStrings": {
    "ToDoDb": "Server=todo-db;Database=todo;User Id=postgres;Password=kiamol-2*2*;"
  }
}
```
Now `http://localhost:8080/` works, but can't see the list, nor add any items...
 
## browse to the app and add some items

# Section 4.5: Managing app configuration in Kubernetes


## delete all the resources in all the files in all the directories:
```bash
kubectl delete -f sleep/
kubectl delete -f todo-list/
kubectl delete -f todo-list/configMaps/
kubectl delete -f todo-list/secrets/
```
Each commmand deletes, but also has errors:

```
deployment.apps "sleep" deleted
Error from server (NotFound): error when deleting "sleep/sleep-with-configMap-env.yaml": deployments.apps "sleep" not found
Error from server (NotFound): error when deleting "sleep/sleep-with-env.yaml": deployments.apps "sleep" not found
Error from server (NotFound): error when deleting "sleep/sleep-with-secret.yaml": deployments.apps "sleep" not found
Error from server (NotFound): error when deleting "sleep/sleep.yaml": deployments.apps "sleep" not found
...
service "todo-db" deleted
deployment.apps "todo-db" deleted
deployment.apps "todo-web" deleted
service "todo-web-test" deleted
deployment.apps "todo-web-test" deleted
service "todo-web" deleted
Error from server (NotFound): error when deleting "todo-list/todo-web-dev-no-logging.yaml": deployments.apps "todo-web" not found
Error from server (NotFound): error when deleting "todo-list/todo-web-dev.yaml": deployments.apps "todo-web" not found
Error from server (NotFound): error when deleting "todo-list/todo-web.yaml": deployments.apps "todo-web" not found
...
configmap "todo-web-config-dev" deleted
configmap "todo-web-config-test" deleted
Error from server (NotFound): error when deleting "todo-list/configMaps/todo-web-config-dev.yaml": configmaps "todo-web-config-dev" not found
...
secret "todo-db-secret-test" deleted
secret "todo-web-secret-test" deleted
Error from server (NotFound): error when deleting "todo-list/secrets/todo-db-secret-test.yaml": secrets "todo-db-secret-test" not found
```

# Chapter 4 Lab

```bash
λ kubectl apply -f postgres/
secret/ch04-lab-db-secret created
service/ch04-lab-db created
deployment.apps/ch04-lab-db created

λ kubectl apply -f solution/
configmap/adminer-config created
secret/adminer-secret created
service/adminer-web created
deployment.apps/adminer-web created
```

# Chapter 5: Storing data with volumes, mounts, and claims

## switch to this chapter’s exercise directory:
`cd ch05`
 
## deploy a sleep Pod:
`kubectl apply -f sleep/sleep.yaml`
 
## write a file inside the container:
`kubectl exec deploy/sleep -- sh -c 'echo ch05 > /file.txt; ls /*.txt'`
Also tried the one without `sh -c` from after the replacement, but interestingly...
```bash
Wed Oct 04 09:09:52
~/GitHub/kiamol/ch05
paulkaefer ~/GitHub/kiamol/ch05 λ kubectl exec deploy/sleep -- ls /*.txt
ls: /*.txt: No such file or directory
command terminated with exit code 1

Wed Oct 04 09:09:55
~/GitHub/kiamol/ch05
paulkaefer ~/GitHub/kiamol/ch05 λ kubectl exec deploy/sleep -- ls *.txt
ls: *.txt: No such file or directory
command terminated with exit code 1

Wed Oct 04 09:09:57
~/GitHub/kiamol/ch05
paulkaefer ~/GitHub/kiamol/ch05 λ kubectl exec deploy/sleep -- sh -c 'ls *.txt'
file.txt

Wed Oct 04 09:10:14
~/GitHub/kiamol/ch05
paulkaefer ~/GitHub/kiamol/ch05 λ kubectl exec deploy/sleep -- sh -c 'ls /*.txt' 
/file.txt
```
 
## check the container ID:
`kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[0].containerID}'`
```
docker://b3140d8ca3ed7aa96309707d8d2ff82bace05ca62f4169a470b1f3595cb61b02
```

## kill all processes in the container, causing a Pod restart:
`kubectl exec -it deploy/sleep -- killall5`
 
## check the replacment container ID:
`kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[0].containerID}'`
```
docker://dffbc776cb327ba56a8861a4649cd9d4d7a273f0e046ce99078f74c0437c3aa7
```
 
## look for the file you wrote--it won’t be there:
`kubectl exec deploy/sleep -- ls /*.txt`
Nothing! Nor if I run:
`kubectl exec deploy/sleep -- sh -c 'ls /*.txt'`


## Now trying with emptyDir:

## update the sleep Pod to use an EmptyDir volume:
`kubectl apply -f sleep/sleep-with-emptyDir.yaml`
```
deployment.apps/sleep configured
```
 
## list the contents of the volume mount:
`kubectl exec deploy/sleep -- ls /data`
 
## create a file in the empty directory:
`kubectl exec deploy/sleep -- sh -c 'echo ch05 > /data/file.txt; ls /data'`
```
file.txt
```
 
## check the container ID:
`kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[0].containerID}'`
```
docker://412aa9574352e22adcedf6e353e55743d66bc2adf97ceb7b609c0e776eec441e
```
 
## kill the container processes:
`kubectl exec deploy/sleep -- killall5`
 
## check replacement container ID:
`kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[0].containerID}'`
```
docker://14aa54acc387cf9cb64cf5125ee48009f6ccf2cdcc88704c45915e917c90bfaa
```
 
## read the file in the volume:
`kubectl exec deploy/sleep -- cat /data/file.txt`
```
ch05
```


## deploy the Pi application:
`kubectl apply -f pi/v1/ `
```
configmap/pi-proxy-configmap created
service/pi-proxy created
deployment.apps/pi-proxy created
service/pi-web created
deployment.apps/pi-web created
```

## wait for the web Pod to be ready:
`kubectl wait --for=condition=Ready pod -l app=pi-web`
```
pod/pi-web-55b6cb574-6ldjm condition met
```

## find the app URL from your LoadBalancer:
`kubectl get svc pi-proxy -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8080/?dp=30000'`
```
http://localhost:8080/?dp=30000
```
Browsed there; also tried `http://localhost:8080/?dp=10` and `http://localhost:8080/?dp=100000` successfully!

![](./ch05/Screenshot_2023-10-04_pi_100000.png)
 
## browse to the URL, wait for the response then refresh the page
 
## check the cache in the proxy
kubectl exec deploy/pi-proxy -- ls -l /data/nginx/cache









