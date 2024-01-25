# Info
This is code and output from examples in the book.

# Get where I need to be locally:
`cd GitHub`

## Section 2.1: How Kubernetes runs and manages containers

### run a Pod with a single container; the restart flag tells Kubernetes
### to create just the Pod and no other resources:
`kubectl run hello-kiamol --image=kiamol/ch02-hello-kiamol`
### can add --restart=Never if desired; see https://github.com/sixeyed/kiamol/issues/45
 
### wait for the Pod to be ready:
`kubectl wait --for=condition=Ready pod hello-kiamol`
 
### list all the Pods in the cluster:
`kubectl get pods`
 
### show detailed information about the Pod:
`kubectl describe pod hello-kiamol`

### get the basic information about the Pod:
`kubectl get pod hello-kiamol`
 
### specify custom columns in the output, selecting network details:
`kubectl get pod hello-kiamol --output custom-columns=NAME:metadata.name,NODE_IP:status.hostIP,POD_IP:status.podIP`
 
### specify a JSONPath query in the output,
### selecting the ID of the first container in the Pod:
`kubectl get pod hello-kiamol -o jsonpath='{.status.containerStatuses[0].containerID}'`

### find the Pod’s container:
`docker container ls -q --filter label=io.kubernetes.container.name=hello-kiamol`
 
### now delete that container:
`docker container rm -f $(docker container ls -q --filter label=io.kubernetes.container.name=hello-kiamol)`
 
### check the Pod status:
`kubectl get pod hello-kiamol`
 
### and find the container again:
`docker container ls -q --filter label=io.kubernetes.container.name=hello-kiamol`

### listen on port 8080 on your machine and send traffic
### to the Pod on port 80:
`kubectl port-forward pod/hello-kiamol 8080:80`
 
### now browse to http://localhost:8080
 
### when you’re done press ctrl-c to end the port forward

## Section 2.2: Running Pods with controllers

### create a Deployment called "hello-kiamol-2", running the same web app:
`kubectl create deployment hello-kiamol-2 --image=kiamol/ch02-hello-kiamol`
 
### list all the Pods:
`kubectl get pods`

### print the labels that the Deployment adds to the Pod:
`kubectl get deploy hello-kiamol-2 -o jsonpath='{.spec.template.metadata.labels}'`
 
### list the Pods that have that matching label:
`kubectl get pods -l app=hello-kiamol-2`

### list all Pods, showing the Pod name and labels:
`kubectl get pods -o custom-columns=NAME:metadata.name,LABELS:metadata.labels`
 
### update the "app" label for the Deployment’s Pod:
`kubectl label pods -l app=hello-kiamol-2 --overwrite app=hello-kiamol-x`
 
### fetch Pods again:
`kubectl get pods -o custom-columns=NAME:metadata.name,LABELS:metadata.labels`

### run a port forward from your local machine to the Deployment:
`kubectl port-forward deploy/hello-kiamol-2 8080:80`
 
### browse to http://localhost:8080
### when you’re done, exit with ctrl-c

# Section 2.3: Defining Deployments in application manifests

### switch from the root of the kiamol repository to the chapter 2 folder:
`cd ch02`
 
### deploy the application from the manifest file:
`kubectl apply -f pod.yaml`
```bash
pod/hello-kiamol-3 created
```
 
### list running Pods:
`kubectl get pods`
```bash
NAME                              READY   STATUS    RESTARTS   AGE
hello-kiamol                      1/1     Running   1          4d2h
hello-kiamol-2-5dbf59b864-f72vp   1/1     Running   0          4d2h
hello-kiamol-2-5dbf59b864-vk6br   1/1     Running   0          4d1h
hello-kiamol-3                    1/1     Running   0          3s
```

### deploy the application from the manifest file:
`kubectl apply -f https://raw.githubusercontent.com/sixeyed/kiamol/master/ch02/pod.yaml`
```bash
pod/hello-kiamol-3 unchanged
```

### run the app using the Deployment manifest:
`kubectl apply -f deployment.yaml`
 
### find Pods managed by the new Deployment:
`kubectl get pods -l app=hello-kiamol-4`
```bash
NAME                             READY   STATUS    RESTARTS   AGE
hello-kiamol-4-fb9d497f8-rlzg7   1/1     Running   0          4s
```

## Section 2.4: Working with applications in Pods

### check the internal IP address of the first Pod we ran: 
`kubectl get pod hello-kiamol -o custom-columns=NAME:metadata.name,POD_IP:status.podIP`
```bash
NAME           POD_IP
hello-kiamol   10.1.0.16
```
 
### run an interactive shell command in the Pod:
`kubectl exec -it hello-kiamol -- sh`
 
### inside the Pod, check the IP address:
`hostname -i`
 
### and test the web app:
`wget -O - http://localhost | head -n 4`
 
### leave the shell:
`exit`

### print the latest container logs from Kubernetes:
`kubectl logs --tail=2 hello-kiamol`
```bash
2023/09/14 17:21:04 [error] 34#34: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 127.0.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8080", referrer: "http://localhost:8080/"
127.0.0.1 - - [18/Sep/2023:20:03:20 +0000] "GET / HTTP/1.1" 200 353 "-" "Wget" "-"
```
 
### ...and compare the actual container logs--if you’re using Docker:
`docker container logs --tail=2 $(docker container ls -q --filter label=io.kubernetes.container.name=hello-kiamol)`

### make a call to the web app inside the container for the Pod we created from the Deployment YAML file: 
`kubectl exec deploy/hello-kiamol-4 -- sh -c 'wget -O - http://localhost > /dev/null'`
```bash
Connecting to localhost (127.0.0.1:80)
writing to stdout
-                    100% |********************************|   353  0:00:00 ETA
written to stdout
```
 
### and check that Pod’s logs:
`kubectl logs --tail=1 -l app=hello-kiamol-4`

### create the local directory:
`mkdir -p /tmp/kiamol/ch02`
 
### copy the web page from the Pod:
`kubectl cp hello-kiamol:/usr/share/nginx/html/index.html /tmp/kiamol/ch02/index.html`
```bash
tar: removing leading '/' from member names
```
 
### check the local file contents:
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

### list all running Pods:
`kubectl get pods`
```bash
NAME                              READY   STATUS    RESTARTS   AGE
hello-kiamol                      1/1     Running   1          4d2h
hello-kiamol-2-5dbf59b864-f72vp   1/1     Running   0          4d2h
hello-kiamol-2-5dbf59b864-vk6br   1/1     Running   0          4d2h
hello-kiamol-3                    1/1     Running   0          14m
hello-kiamol-4-fb9d497f8-rlzg7    1/1     Running   0          12m
```
 
### delete all Pods:
`kubectl delete pods --all`
```bash
pod "hello-kiamol" deleted
pod "hello-kiamol-2-5dbf59b864-f72vp" deleted
pod "hello-kiamol-2-5dbf59b864-vk6br" deleted
pod "hello-kiamol-3" deleted
pod "hello-kiamol-4-fb9d497f8-rlzg7" deleted
```
 
### check again:
`kubectl get pods`
```bash
NAME                              READY   STATUS    RESTARTS   AGE
hello-kiamol-2-5dbf59b864-t6m95   1/1     Running   0          21s
hello-kiamol-4-fb9d497f8-7xkd7    1/1     Running   0          21s
```

### view Deployments:
`kubectl get deploy`
```bash
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
hello-kiamol-2   1/1     1            1           4d2h
hello-kiamol-4   1/1     1            1           13m
```
 
### delete all Deployments:
`kubectl delete deploy --all`
```bash
deployment.apps "hello-kiamol-2" deleted
deployment.apps "hello-kiamol-4" delete
```
 
### view Pods:
`kubectl get pods`
```bash
No resources found in default namespace.
```

### check all resources:
`kubectl get all`
```bash
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d4h
```

# Chapter 3: Connecting Pods over the network with Services

### start up your lab environment--run Docker Desktop if it's not running--and switch to this chapter’s directory in your copy of the source code:
`cd ch03`
 
### create two Deployments, which each run one Pod:
`kubectl apply -f sleep/sleep1.yaml -f sleep/sleep2.yaml`
 
### wait for the Pod to be ready:
`kubectl wait --for=condition=Ready pod -l app=sleep-2`
 
### check the IP address of the second Pod:
`kubectl get pod -l app=sleep-2 --output jsonpath='{.items[0].status.podIP}'`
```bash
10.1.0.29
```
 
### use that address to ping the second Pod from the first:
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

### check the current Pod’s IP address:
`kubectl get pod -l app=sleep-2 --output jsonpath='{.items[0].status.podIP}'`
 
### delete the Pod so the Deployment replaces it:
`kubectl delete pods -l app=sleep-2`
 
### check the IP address of the replacement Pod:
`kubectl get pod -l app=sleep-2 --output jsonpath='{.items[0].status.podIP}'`
```bash
10.1.0.31
```


### deploy the Service defined in listing 3.1:
`kubectl apply -f sleep/sleep2-service.yaml`
 
### show the basic details of the Service:
`kubectl get svc sleep-2`
```bash
NAME      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
sleep-2   ClusterIP   10.98.2.165   <none>        80/TCP    6s
```
 
### run a ping command to check connectivity--this will fail:
`kubectl exec deploy/sleep-1 -- ping -c 1 sleep-2`
```bash
PING sleep-2 (10.98.2.165): 56 data bytes

--- sleep-2 ping statistics ---
1 packets transmitted, 0 packets received, 100% packet loss
command terminated with exit code 1
```

# Section 3.2: Routing traffic between Pods

### run the website and API as separate Deployments: 
`kubectl apply -f numbers/api.yaml -f numbers/web.yaml`
```bash
deployment.apps/numbers-api created
deployment.apps/numbers-web created
```
 
### wait for the Pod to be ready:
`kubectl wait --for=condition=Ready pod -l app=numbers-web`
```bash
pod/numbers-web-865c56b9d-9p4m7 condition met
```
 
### forward a port to the web app:
`kubectl port-forward deploy/numbers-web 8080:80`
 
### browse to the site at http://localhost:8080 and click the Go button--you'll see an error message
```
KIAMOL Random Number Generator
RNG service unavailable!

(Using API at: http://numbers-api/sixeyed/kiamol/master/ch03/numbers/rng)
```
 
### exit the port forward:
ctrl-c

### deploy the Service from listing 3.2:
`kubectl apply -f numbers/api-service.yaml`
```
service/numbers-api created
```

### check the Service details:
`kubectl get svc numbers-api`
```bash
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
numbers-api   ClusterIP   10.108.103.175   <none>        80/TCP    32s
```

### forward a port to the web app:
`kubectl port-forward deploy/numbers-web 8080:80`
 
### browse to the site at http://localhost:8080 and click the Go button
```
KIAMOL Random Number Generator
Here it is: 89

(Using API at: http://numbers-api/sixeyed/kiamol/master/ch03/numbers/rng)
```
82, 90, 34, 24, 16 on refresh
 
### exit the port forward:
ctrl-c


### check the name and IP address of the API Pod:
`kubectl get pod -l app=numbers-api -o custom-columns=NAME:metadata.name,POD_IP:status.podIP`
```
NAME                           POD_IP
numbers-api-7c599bfcf6-fd4qz   10.1.0.33
```
 
### delete that Pod:
`kubectl delete pod -l app=numbers-api`
```
pod "numbers-api-7c599bfcf6-fd4qz" deleted
```
 
### check the replacement Pod:
`kubectl get pod -l app=numbers-api -o custom-columns=NAME:metadata.name,POD_IP:status.podIP `
```
NAME                           POD_IP
numbers-api-7c599bfcf6-rztxn   10.1.0.34
```

### forward a port to the web app:
`kubectl port-forward deploy/numbers-web 8080:80`
 
### browse to the site at http://localhost:8080 and click the Go button
 
### exit the port forward:
ctrl-c

# Section 3.3: Routing external traffic to Pods

### deploy the LoadBalancer Service for the website--if your firewall checks 
### that you want to allow traffic, then it is OK to say yes:
`kubectl apply -f numbers/web-service.yaml`
 
### check the details of the Service:
`kubectl get svc numbers-web`
```
NAME          TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
numbers-web   LoadBalancer   10.101.94.89   localhost     8080:30122/TCP   5s
```
 
### use formatting to get the app URL from the EXTERNAL-IP field:
`kubectl get svc numbers-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8080'`
```
http://localhost:8080
```


# Section 3.4: Routing traffic outside Kubernetes

### delete the current API Service:
`kubectl delete svc numbers-api`
 
### deploy a new ExternalName Service:
`kubectl apply -f numbers-services/api-service-externalName.yaml`
 
### check the Service configuration:
`kubectl get svc numbers-api`
```
NAME          TYPE           CLUSTER-IP   EXTERNAL-IP                 PORT(S)   AGE
numbers-api   ExternalName   <none>       raw.githubusercontent.com   <none>    6s
```
 
### refresh the website in your browser and test with the Go button
```
KIAMOL Random Number Generator
RNG service unavailable!

(Using API at: http://numbers-api/sixeyed/kiamol/master/ch03/numbers/rng)
```

I think it *should* be using https://raw.githubusercontent.com/sixeyed/kiamol/master/ch03/numbers/rng.


### run the DNS lookup tool to resolve the Service name:
`kubectl exec deploy/sleep-1 -- sh -c 'nslookup numbers-api | tail -n 5'`



### remove the existing Service:
`kubectl delete svc numbers-api`
 
### deploy the headless Service:
`kubectl apply -f numbers-services/api-service-headless.yaml`
 
### check the Service:
`kubectl get svc numbers-api`
 
### check the endpoint: 
`kubectl get endpoints numbers-api`
 
### verify the DNS lookup:
`kubectl exec deploy/sleep-1 -- sh -c 'nslookup numbers-api | grep "^[^*]"'`
 
### browse to the app--it will fail when you try to get a number
```
KIAMOL Random Number Generator
RNG service unavailable!

(Using API at: http://numbers-api/sixeyed/kiamol/master/ch03/numbers/rng)
```

# Section 3.5: Understanding Kubernetes Service resolution

### show the endpoints for the sleep-2 Service:
`kubectl get endpoints sleep-2`
```
NAME      ENDPOINTS      AGE
sleep-2   10.1.0.31:80   38m
```
 
### delete the Pod:
`kubectl delete pods -l app=sleep-2`
```
pod "sleep-2-789c9f5fb8-45gtv" deleted
```
 
### check the endpoint is updated with the IP of the replacement Pod:
`kubectl get endpoints sleep-2`
```
NAME      ENDPOINTS      AGE
sleep-2   10.1.0.35:80   38m
```
 
### delete the whole Deployment:
`kubectl delete deploy sleep-2`
```
deployment.apps "sleep-2" deleted
```
 
### check the endpoint still exists, with no IP addresses:
`kubectl get endpoints sleep-2`
```
NAME      ENDPOINTS   AGE
sleep-2   <none>      39m
```


### check the Services in the default namespace:
`kubectl get svc --namespace default`

### check Services in the system namespace:
`kubectl get svc -n kube-system`

### try a DNS lookup to a fully qualified Service name:
`kubectl exec deploy/sleep-1 -- sh -c 'nslookup numbers-api.default.svc.cluster.local | grep "^[^*]"'`

### and for a Service in the system namespace:
`kubectl exec deploy/sleep-1 -- sh -c 'nslookup kube-dns.kube-system.svc.cluster.local | grep "^[^*]"'`



### delete Deployments:
`kubectl delete deploy --all`
```
deployment.apps "numbers-api" deleted
deployment.apps "numbers-web" deleted
deployment.apps "sleep-1" deleted
```
 
### and Services:
`kubectl delete svc --all`
```
service "kubernetes" deleted
service "numbers-api" deleted
service "numbers-web" deleted
service "sleep-2" deleted
```
 
### check what’s running:
`kubectl get all`
```
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   8s

```


# Chapter 4: Configuring applications with ConfigMaps and Secrets

## Section 4.1: How Kubernetes supplies configuration to apps
### switch to the exercise directory for this chapter:
`cd ch04`
 
### deploy a Pod using the sleep image with no extra configuration:
`kubectl apply -f sleep/sleep.yaml`
```
deployment.apps/sleep created
```

### wait for the Pod to be ready:
`kubectl wait --for=condition=Ready pod -l app=sleep`
```
pod/sleep-8648c6f777-dbbpg condition met
```
 
### check some of the environment variables in the Pod container:
`kubectl exec deploy/sleep -- printenv HOSTNAME KIAMOL_CHAPTER`
```
sleep-8648c6f777-dbbpg
command terminated with exit code 1
```


### update the Deployment:
`kubectl apply -f sleep/sleep-with-env.yaml`
```
deployment.apps/sleep configured
```
 
### check the same environment variables in the new Pod:
`kubectl exec deploy/sleep -- printenv HOSTNAME KIAMOL_CHAPTER`
```
sleep-5b67d77966-f2dmt
04
```


### create a ConfigMap with data from the command line:
`kubectl create configmap sleep-config-literal --from-literal=kiamol.booksection='4.1415'`
```
configmap/sleep-config-literal created
```
 
### check the ConfigMap details:
`kubectl get cm sleep-config-literal`
```
NAME                   DATA   AGE
sleep-config-literal   1      9s
```
 
### show the friendly description of the ConfigMap:
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
 
### deploy the updated Pod spec from listing 4.2:
`kubectl apply -f sleep/sleep-with-configMap-env.yaml`
```
deployment.apps/sleep configured
```
 
### check the Kiamol environment variables:
`kubectl exec deploy/sleep -- sh -c 'printenv | grep "^KIAMOL"'`
```
KIAMOL_CHAPTER=04
```

## Section 4.2: Storing and using configuration files in ConfigMaps


### load an environment variable into a new ConfigMap:
`kubectl create configmap sleep-config-env-file --from-env-file=sleep/ch04.env`
```
configmap/sleep-config-env-file created
```
 
### check the details of the ConfigMap:
`kubectl get cm sleep-config-env-file`
```
NAME                    DATA   AGE
sleep-config-env-file   4      13s
```
 
### update the Pod to use the new ConfigMap:
`kubectl apply -f sleep/sleep-with-configMap-env-file.yaml`
```
deployment.apps/sleep configured
```

### check the values in the container:
`kubectl exec deploy/sleep -- sh -c 'printenv | grep "^KIAMOL"'`
Not the same results as the book...
```
KIAMOL_CHAPTER=04
```


### deploy the app with a Service to access it:
`kubectl apply -f todo-list/todo-web.yaml`
```
deployment.apps/todo-web created
```
 
### wait for the Pod to be ready:
`kubectl wait --for=condition=Ready pod -l app=todo-web`
```
pod/todo-web-659fff7795-kxrfq condition met
```
 
### get the address of the app:
`kubectl get svc todo-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8080'`
```
http://localhost:8080
```

### browse to the app and have a play around then try browsing to /config
Doesn't work for me... `ERR_CONNECTION_REFUSED`.
 
### check the application logs:
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

### create the JSON ConfigMap:
`kubectl apply -f todo-list/configMaps/todo-web-config-dev.yaml`
```
configmap/todo-web-config-dev created
```
 
### update the app to use the ConfigMap:
`kubectl apply -f todo-list/todo-web-dev.yaml`
```
deployment.apps/todo-web configured
```
 
### refresh your web browser at the /config page for your Service 
This works! Just the config + diagnostics pages. Diagnostics details:
```
Hostname  .NET Version  OS Architecture OS Description
todo-web-b6d6f9ff5-rg5dz  .NET 6.0.8  Arm64 Linux 6.3.13-linuxkit #1 SMP PREEMPT Thu Sep 7 07:48:47 UTC 2023
```

Source code link: https://github.com/sixeyed/kiamol/tree/master/ch04/docker-images/todo-list


# Section 4.3: Surfacing configuration data from ConfigMaps

### show the default config file:
`kubectl exec deploy/todo-web -- sh -c 'ls -l /app/app*.json'`
```
-rw-r--r--    1 root     root           469 Jun 26  2022 /app/appsettings.json
```
 
### show the config file in the volume mount:
`kubectl exec deploy/todo-web -- sh -c 'ls -l /app/config/*.json'`
```
lrwxrwxrwx    1 root     root            18 Sep 21 17:31 /app/config/config.json -> ..data/config.json
```
 
### check it really is read-only:
`kubectl exec deploy/todo-web -- sh -c 'echo ch04 >> /app/config/config.json'`
```
sh: can't create /app/config/config.json: Read-only file system
command terminated with exit code 1
```

### check the current app logs:
`kubectl logs -l app=todo-web`
Errors...
 
### deploy the updated ConfigMap:
`kubectl apply -f todo-list/configMaps/todo-web-config-dev-with-logging.yaml`
```
configmap/todo-web-config-dev configured
```
 
### wait for the config change to make it to the Pod:
`sleep 120`
 
### check the new setting:
`kubectl exec deploy/todo-web -- sh -c 'ls -l /app/config/*.json'`
```
lrwxrwxrwx    1 root     root            18 Sep 21 17:31 /app/config/config.json -> ..data/config.json
lrwxrwxrwx    1 root     root            19 Sep 21 17:37 /app/config/logging.json -> ..data/logging.json
```
 
### load a few pages from the site at your Service IP address
 
### check the logs again:
`kubectl logs -l app=todo-web`
More errors...

### deploy the badly configured Pod:
`kubectl apply -f todo-list/todo-web-dev-broken.yaml`
```
deployment.apps/todo-web configured
```
 
### browse back to the app and see how it looks

### check the app logs:
`kubectl logs -l app=todo-web`
Errors...
 
### and check the Pod status:
`kubectl get pods -l app=todo-web`
```
NAME                        READY   STATUS             RESTARTS      AGE
todo-web-76b6c46bf6-554nz   0/1     CrashLoopBackOff   1 (13s ago)   15s
todo-web-b6d6f9ff5-rg5dz    1/1     Running            0             11m
```

### apply the change:
`kubectl apply -f todo-list/todo-web-dev-no-logging.yaml`
```
deployment.apps/todo-web configured
```
 
### list the config folder contents:
`kubectl exec deploy/todo-web -- sh -c 'ls /app/config'`
```
config.json
```
 
### now browse to a few pages on the app
Broken except /config and /diagnostics.
 
### check the logs:
`kubectl logs -l app=todo-web`
Errors...
 
### and check the Pods:
`kubectl get pods -l app=todo-web`
```
NAME                        READY   STATUS    RESTARTS   AGE
todo-web-85c477b74c-fqw4c   1/1     Running   0          46s
```


# Section 4.4: Configuring sensitive data with Secrets


### FOR WINDOWS USERS--this script adds a Base64 command to your session: 
`. .\base64.ps1`
 
### now create a secret from a plain text literal:
`kubectl create secret generic sleep-secret-literal --from-literal=secret=shh...`
```
secret/sleep-secret-literal created
```
 
### show the friendly details of the Secret:
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
 
### retrieve the encoded Secret value:
`kubectl get secret sleep-secret-literal -o jsonpath='{.data.secret}'`
```
c2hoLi4u
```
 
### and decode the data:
`kubectl get secret sleep-secret-literal -o jsonpath='{.data.secret}' | base64 -d`
```
shh...
```

### update the sleep Deployment:
`kubectl apply -f sleep/sleep-with-secret.yaml`
```
deployment.apps/sleep configured
```
 
### check the environment variable in the Pod:
`kubectl exec deploy/sleep -- printenv KIAMOL_SECRET`
```
shh...
```

### deploy the Secret:
`kubectl apply -f todo-list/secrets/todo-db-secret-test.yaml`
```
secret/todo-db-secret-test created
```
 
### check the data is encoded:
`kubectl get secret todo-db-secret-test -o jsonpath='{.data.POSTGRES_PASSWORD}'`
```
a2lhbW9sLTIqMio=
```

echo "a2lhbW9sLTIqMio=" | base64 -d
```
kiamol-2*2*
```

 
### see what annotations are stored:
kubectl get secret todo-db-secret-test -o jsonpath='{.metadata.annotations}'
```
{"kubectl.kubernetes.io/last-applied-configuration":"{\"apiVersion\":\"v1\",\"kind\":\"Secret\",\"metadata\":{\"annotations\":{},\"name\":\"todo-db-secret-test\",\"namespace\":\"default\"},\"stringData\":{\"POSTGRES_PASSWORD\":\"kiamol-2*2*\"},\"type\":\"Opaque\"}\n"}
```


### deploy the YAML from listing 4.13
`kubectl apply -f todo-list/todo-db-test.yaml`
```
deployment.apps/todo-db created
```
 
### check the database logs:
`kubectl logs -l app=todo-db --tail 1`
```
Error from server (BadRequest): container "db" in pod "todo-db-8b974978c-txzpf" is waiting to start: ContainerCreating
```
...ran that again:
```
2023-09-21 17:50:37.123 UTC [1] LOG:  database system is ready to accept connections
```
 
### verify the password file permissions:
`kubectl exec deploy/todo-db -- sh -c 'ls -l $(readlink -f /secrets/postgres_password)'`
```
-r--------    1 root     root            11 Sep 21 17:50 /secrets/..2023_09_21_17_50_27.776472020/postgres_password
```

### the ConfigMap configures the app to use Postgres:
`kubectl apply -f todo-list/configMaps/todo-web-config-test.yaml`
```
configmap/todo-web-config-test created
```

### the Secret contains the credentials to connect to Postgres:
`kubectl apply -f todo-list/secrets/todo-web-secret-test.yaml`
```
secret/todo-web-secret-test created
```
 
### the Deployment Pod spec uses the ConfigMap and Secret:
`kubectl apply -f todo-list/todo-web-test.yaml`
```
deployment.apps/todo-web-test created
```
 
### check the database credentials are set in the app:
`kubectl exec deploy/todo-web-test -- cat /app/secrets/secrets.json`
```
{
  "ConnectionStrings": {
    "ToDoDb": "Server=todo-db;Database=todo;User Id=postgres;Password=kiamol-2*2*;"
  }
}
```
Now `http://localhost:8080/` works, but can't see the list, nor add any items...
 
### browse to the app and add some items

# Section 4.5: Managing app configuration in Kubernetes


### delete all the resources in all the files in all the directories:
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

### switch to this chapter’s exercise directory:
`cd ch05`
 
### deploy a sleep Pod:
`kubectl apply -f sleep/sleep.yaml`
 
### write a file inside the container:
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
 
### check the container ID:
`kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[0].containerID}'`
```
docker://b3140d8ca3ed7aa96309707d8d2ff82bace05ca62f4169a470b1f3595cb61b02
```

### kill all processes in the container, causing a Pod restart:
`kubectl exec -it deploy/sleep -- killall5`
 
### check the replacment container ID:
`kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[0].containerID}'`
```
docker://dffbc776cb327ba56a8861a4649cd9d4d7a273f0e046ce99078f74c0437c3aa7
```
 
### look for the file you wrote--it won’t be there:
`kubectl exec deploy/sleep -- ls /*.txt`
Nothing! Nor if I run:
`kubectl exec deploy/sleep -- sh -c 'ls /*.txt'`


### Now trying with emptyDir:

### update the sleep Pod to use an EmptyDir volume:
`kubectl apply -f sleep/sleep-with-emptyDir.yaml`
```
deployment.apps/sleep configured
```
 
### list the contents of the volume mount:
`kubectl exec deploy/sleep -- ls /data`
 
### create a file in the empty directory:
`kubectl exec deploy/sleep -- sh -c 'echo ch05 > /data/file.txt; ls /data'`
```
file.txt
```
 
### check the container ID:
`kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[0].containerID}'`
```
docker://412aa9574352e22adcedf6e353e55743d66bc2adf97ceb7b609c0e776eec441e
```
 
### kill the container processes:
`kubectl exec deploy/sleep -- killall5`
 
### check replacement container ID:
`kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[0].containerID}'`
```
docker://14aa54acc387cf9cb64cf5125ee48009f6ccf2cdcc88704c45915e917c90bfaa
```
 
### read the file in the volume:
`kubectl exec deploy/sleep -- cat /data/file.txt`
```
ch05
```


### deploy the Pi application:
`kubectl apply -f pi/v1/ `
```
configmap/pi-proxy-configmap created
service/pi-proxy created
deployment.apps/pi-proxy created
service/pi-web created
deployment.apps/pi-web created
```

### wait for the web Pod to be ready:
`kubectl wait --for=condition=Ready pod -l app=pi-web`
```
pod/pi-web-55b6cb574-6ldjm condition met
```

### find the app URL from your LoadBalancer:
`kubectl get svc pi-proxy -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8080/?dp=30000'`
```
http://localhost:8080/?dp=30000
```
 
### browse to the URL, wait for the response then refresh the page
Browsed there; also tried `http://localhost:8080/?dp=10` and `http://localhost:8080/?dp=100000` successfully!

![](./ch05/Screenshot_2023-10-04_pi_100000.png)
 
### check the cache in the proxy
`kubectl exec deploy/pi-proxy -- ls -l /data/nginx/cache`
```
total 32
drwx------    4 nginx    nginx         4096 Oct  4 14:20 0
drwx------    3 nginx    nginx         4096 Oct  4 14:20 1
drwx------    3 nginx    nginx         4096 Oct  4 14:21 2
drwx------    3 nginx    nginx         4096 Oct  4 14:20 4
drwx------    4 nginx    nginx         4096 Oct  4 14:20 5
drwx------    4 nginx    nginx         4096 Oct  4 14:21 9
drwx------    3 nginx    nginx         4096 Oct  4 14:21 a
drwx------    3 nginx    nginx         4096 Oct  4 14:20 d
```

### delete the proxy Pod: 
`kubectl delete pod -l app=pi-proxy`
```
pod "pi-proxy-866647b9c9-fbkhj" deleted
```
 
### check the cache directory of the replacement Pod:
`kubectl exec deploy/pi-proxy -- ls -l /data/nginx/cache`
```
total 0
```
 
### refresh your browser at the Pi app URL


### update the proxy Pod to use a HostPath volume:
`kubectl apply -f pi/nginx-with-hostPath.yaml`
 
### list the contents of the cache directory:
`kubectl exec deploy/pi-proxy -- ls -l /data/nginx/cache`
 
### browse to the app URL

### delete the proxy Pod:
`kubectl delete pod -l app=pi-proxy`
```
pod "pi-proxy-fbf5595-qbdcg" deleted
```
 
### check the cache directory in the replacement Pod:
`kubectl exec deploy/pi-proxy -- ls -l /data/nginx/cache`
```
drwx------    3 nginx    nginx           60 Oct  4 14:50 1
drwx------    3 nginx    nginx           60 Oct  4 14:52 8
```

```bash
Wed Oct 04 09:54:04
~/GitHub/kiamol/ch05
paulkaefer ~/GitHub/kiamol/ch05 λ kubectl exec deploy/pi-proxy -- cat /data/nginx/cache/8/2b/333d56fdcab55021d083c0ddd7fc22b8
??e????????0|e?&??n?
KEY: http://pi-web/?dp=27000
HTTP/1.1 200 OK
Connection: close
Content-Type: text/html; charset=utf-8
Date: Wed, 04 Oct 2023 14:52:31 GMT
Server: Kestrel

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Pi.Web</title>
    <link rel="stylesheet" href="/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="/css/site.css" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container">
                <a class="navbar-brand" href="/Home">Pi.Web</a>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                        aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex flex-sm-row-reverse">
                    <ul class="navbar-nav flex-grow-1">

                        <li class="nav-item">
                            <a class="nav-link text-dark" href="https://github.com/sixeyed/presentations/tree/master/buildstuff/2019/reverse-proxy-nginx-traefik/pi">Source</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    <div class="container">
        <main role="main" class="pb-3">
            
<div class="container">
  <div class="row">
      <div class="col-md-1">
          <img src="/img/pi-large.png" alt="Pi" width="100%" />
      </div>
      <div class="col-md-4">
          <h3>To: 27,000 d.p.</h3>
      </div>
      <div class="col-md-3">
          <h3>in: 503 ms.</h3>
      </div>
      <div class="col-md-4">
          <h3>from: pi-web-55b6cb574-6ldjm</h3>
      </div>
  </div>
  <hr/>
  <div class="row">            
    <p class="text-break">3.141592653589793238462643383279502884197169399375105820974944592307816406286208998628034825342117067982148086513282306647093844609550582231725359408128481117450284102701938521105559644622948954930381964428810975665933446128475648233786783165271201909145648566923460348610454326648213393607260249141273724587006606315588174881520920962829254091715364367892590360011330530548820466521384146951941511609433057270365759591953092186117381932611793105118548074462379962749567351885752724891227938183011949129833673362440656643086021394946395224737190702179860943702770539217176293176752384674818467669405132000568127145263560827785771342757789609173637178721468440901224953430146549585371050792279689258923542019956112129021960864034418159813629774771309960518707211349999998372978049951059731732816096318595024459455346908302642522308253344685035261931188171010003137838752886587533208381420617177669147303598253490428755468731159562863882353787593751957781857780532171226806613001927876611195909216420198938095257201065485863278865936153381827968230301952035301852968995773622599413891249721775283479131515574857242454150695950829533116861727855889075098381754637464939319255060400927701671139009848824012858361603563707660104710181942955596198946767837449448255379774726847104047534646208046684259069491293313677028989152104752162056966024058038150193511253382430035587640247496473263914199272604269922796782354781636009341721641219924586315030286182974555706749838505494588586926995690927210797509302955321165344987202755960236480665499119881834797753566369807426542527862551818417574672890977772793800081647060016145249192173217214772350141441973568548161361157352552133475741849468438523323907394143334547762416862518983569485562099219222184272550254256887671790494601653466804988627232791786085784383827967976681454100953883786360950680064225125205117392984896084128488626945604241965285022210661186306744278622039194945047123713786960956364371917287467764657573962413890865832645995813390478027590099465764078951269468398352595709825822620522489407726719478268482601476990902640136394437455305068203496252451749399651431429809190659250937221696461515709858387410597885959772975498930161753928468138268683868942774155991855925245953959431049972524680845987273644695848653836736222626099124608051243884390451244136549762780797715691435997700129616089441694868555848406353422072225828488648158456028506016842739452267467678895252138522549954666727823986456596116354886230577456498035593634568174324112515076069479451096596094025228879710893145669136867228748940560101503308617928680920874760917824938589009714909675985261365549781893129784821682998948722658804857564014270477555132379641451523746234364542858444795265867821051141354735739523113427166102135969536231442952484937187110145765403590279934403742007310578539062198387447808478489683321445713868751943506430218453191048481005370614680674919278191197939952061419663428754440643745123718192179998391015919561814675142691239748940907186494231961567945208095146550225231603881930142093762137855956638937787083039069792077346722182562599661501421503068038447734549202605414665925201497442850732518666002132434088190710486331734649651453905796268561005508106658796998163574736384052571459102897064140110971206280439039759515677157700420337869936007230558763176359421873125147120532928191826186125867321579198414848829164470609575270695722091756711672291098169091528017350671274858322287183520935396572512108357915136988209144421006751033467110314126711136990865851639831501970165151168517143765761835155650884909989859982387345528331635507647918535893226185489632132933089857064204675259070915481416549859461637180270981994309924488957571282890592323326097299712084433573265489382391193259746366730583604142813883032038249037589852437441702913276561809377344403070746921120191302033038019762110110044929321516084244485963766983895228684783123552658213144957685726243344189303968642624341077322697802807318915441101044682325271620105265227211166039666557309254711055785376346682065310989652691862056476931257058635662018558100729360659876486117910453348850346113657686753249441668039626579787718556084552965412665408530614344431858676975145661406800700237877659134401712749470420562230538994561314071127000407854733269939081454664645880797270826683063432858785698305235808933065757406795457163775254202114955761581400250126228594130216471550979259230990796547376125517656751357517829666454779174501129961489030463994713296210734043751895735961458901938971311179042978285647503203198691514028708085990480109412147221317947647772622414254854540332157185306142288137585043063321751829798662237172159160771669254748738986654949450114654062843366393790039769265672146385306736096571209180763832716641627488880078692560290228472104031721186082041900042296617119637792133757511495950156604963186294726547364252308177036751590673502350728354056704038674351362222477158915049530984448933309634087807693259939780541934144737744184263129860809988868741326047215695162396586457302163159819319516735381297416772947867242292465436680098067692823828068996400482435403701416314965897940924323789690706977942236250822168895738379862300159377647165122893578601588161755782973523344604281512627203734314653197777416031990665541876397929334419521541341899485444734567383162499341913181480927777103863877343177207545654532207770921201905166096280490926360197598828161332316663652861932668633606273567630354477628035045077723554710585954870279081435624014517180624643626794561275318134078330336254232783944975382437205835311477119926063813346776879695970309833913077109870408591337464144282277263465947047458784778720192771528073176790770715721344473060570073349243693113835049316312840425121925651798069411352801314701304781643788518529092854520116583934196562134914341595625865865570552690496520985803385072242648293972858478316305777756068887644624824685792603953527734803048029005876075825104747091643961362676044925627420420832085661190625454337213153595845068772460290161876679524061634252257719542916299193064553779914037340432875262888963995879475729174642635745525407909145135711136941091193932519107602082520261879853188770584297259167781314969900901921169717372784768472686084900337702424291651300500516832336435038951702989392233451722013812806965011784408745196012122859937162313017114448464090389064495444006198690754851602632750529834918740786680881833851022833450850486082503930213321971551843063545500766828294930413776552793975175461395398468339363830474611996653858153842056853386218672523340283087112328278921250771262946322956398989893582116745627010218356462201349671518819097303811980049734072396103685406643193950979019069963955245300545058068550195673022921913933918568034490398205955100226353536192041994745538593810234395544959778377902374216172711172364343543947822181852862408514006660443325888569867054315470696574745855033232334210730154594051655379068662733379958511562578432298827372319898757141595781119635833005940873068121602876496286744604774649159950549737425626901049037781986835938146574126804925648798556145372347867330390468838343634655379498641927056387293174872332083760112302991136793862708943879936201629515413371424892830722012690147546684765357616477379467520049075715552781965362132392640616013635815590742202020318727760527721900556148425551879253034351398442532234157623361064250639049750086562710953591946589751413103482276930624743536325691607815478181152843667957061108615331504452127473924544945423682886061340841486377670096120715124914043027253860764823634143346235189757664521641376796903149501910857598442391986291642193994907236234646844117394032659184044378051333894525742399508296591228508555821572503107125701266830240292952522011872676756220415420516184163484756516999811614101002996078386909291603028840026910414079288621507842451670908700069928212066041837180653556725253256753286129104248776182582976515795984703562226293486003415872298053498965022629174878820273420922224533985626476691490556284250391275771028402799806636582548892648802545661017296702664076559042909945681506526530537182941270336931378517860904070866711496558343434769338578171138645587367812301458768712660348913909562009939361031029161615288138437909904231747336394804575931493140529763475748119356709110137751721008031559024853090669203767192203322909433467685142214477379393751703443661991040337511173547191855046449026365512816228824462575916333039107225383742182140883508657391771509682887478265699599574490661758344137522397096834080053559849175417381883999446974867626551658276584835884531427756879002909517028352971634456212964043523117600665101241200659755851276178583829204197484423608007193045761893234922927965019875187212726750798125547095890455635792122103334669749923563025494780249011419521238281530911407907386025152274299581807247162591668545133312394804947079119153267343028244186041426363954800044800267049624820179289647669758318327131425170296923488962766844032326092752496035799646925650493681836090032380929345958897069536534940603402166544375589004563288225054525564056448246515187547119621844396582533754388569094113031509526179378002974120766514793942590298969594699556576121865619673378623625612521632086286922210327488921865436480229678070576561514463204692790682120738837781423356282360896320806822246801224826117718589638140918390367367222088832151375560037279839400415297002878307667094447456013455641725437090697939612257142989467154357846878861444581231459357198492252847160504922124247014121478057345510500801908699603302763478708108175450119307141223390866393833952942578690507643100638351983438934159613185434754649556978103829309716465143840700707360411237359984345225161050702705623526601276484830840761183013052793205427462865403603674532865105706587488225698157936789766974220575059683440869735020141020672358502007245225632651341055924019027421624843914035998953539459094407046912091409387001264560016237428802109276457931065792295524988727584610126483699989225695968815920560010165525637567856672279661988578279484885583439751874454551296563443480396642055798293680435220277098429423253302257634180703947699415979159453006975214829336655566156787364005366656416547321704390352132954352916941459904160875320186837937023488868947915107163785290234529244077365949563051007421087142613497459561513849871375704710178795731042296906667021449863746459528082436944578977233004876476524133907592043401963403911473202338071509522201068256342747164602433544005152126693249341967397704159568375355516673027390074972973635496453328886984406119649616277344951827369558822075735517665158985519098666539354948106887320685990754079234240230092590070173196036225475647894064754834664776041146323390565134330684495397907090302346046147096169688688501408347040546074295869913829668246818571031887906528703665083243197440477185567893482308943106828702722809736248093996270607472645539925399442808113736943388729406307926159599546262462970706259484556903471197299640908941805953439325123623550813494900436427852713831591256898929519642728757394691427253436694153236100453730488198551706594121735246258954873016760029886592578662856124966552353382942878542534048308330701653722856355915253478445981831341129001999205981352205117336585640782648494276441137639386692480311836445369858917544264739988228462184490087776977631279572267265556259628254276531830013407092233436577916012809317940171859859993384923549564005709955856113498025249906698423301735035804408116855265311709957089942732870925848789443646005041089226691783525870785951298344172953519537885534573742608590290817651557803905946408735061232261120093731080485485263572282576820341605048466277504500312620080079980492548534694146977516493270950493463938243222718851597405470214828971117779237612257887347718819682546298126868581705074027255026332904497627789442362167411918626943965067151577958675648239939176042601763387045499017614364120469218237076488783419689686118155815873606293860381017121585527266830082383404656475880405138080163363887421637140643549556186896411228214075330265510042410489678352858829024367090488711819090949453314421828766181031007354770549815968077200947469613436092861484941785017180779306810854690009445899527942439813921350558642219648349151263901280383200109773868066287792397180146134324457264009737425700735921003154150893679300816998053652027600727749674584002836240534603726341655425902760183484030681138185510597970566400750942608788573579603732451414678670368809880609716425849759513806930944940151542222194329130217391253835591503100333032511174915696917450271494331515588540392216409722910112903552181576282328318234254832611191280092825256190205263016391147724733148573910777587442538761174657867116941477642144111126358355387136101102326798775641024682403226483464176636980663785768134920453022408197278564719839630878154322116691224641591177673225326433568614618654522268126887268445968442416107854016768142080885028005414361314623082102594173756238994207571362751674573189189456283525704413354375857534269869947254703165661399199968262824727064133622217892390317608542894373393561889165125042440400895271983787386480584726895462438823437517885201439560057104811949884239060613695734231559079670346149143447886360410318235073650277859089757827273130504889398900992391350337325085598265586708924261242947367019390772713070686917092646254842324074855036608013604668951184009366860954632500214585293095000090715105823626729326453738210493872499669933942468551648326113414611068026744663733437534076429402668297386522093570162638464852851490362932019919968828517183953669134522244470804592396602817156551565666111359823112250628905854914509715755390024393153519090210711945730024388017661503527086260253788179751947806101371500448991721002220133501310601639154158957803711779277522597874289191791552241718958536168059474123419339842021874564925644346239253195313510331147639491199507285843065836193536932969928983791494193940608572486396883690326556436421664425760791471086998431573374964883529276932822076294728238153740996154559879825989109371712621828302584811238901196822142945766758071865380650648702613389282299497257453033283896381843944770779402284359883410035838542389735424395647555684095224844554139239410001620769363684677641301781965937997155746854194633489374843912974239143365936041003523437770658886778113949861647874714079326385873862473288964564359877466763847946650407411182565837887845485814896296127399841344272608606187245545236064315371011274680977870446409475828034876975894832824123929296058294861919667091895808983320121031843034012849511620353428014412761728583024355983003204202451207287253558119584014918096925339507577840006746552603144616705082768277222353419110263416315714740612385042584598841990761128725805911393568960143166828317632356732541707342081733223046298799280490851409479036887868789493054695570307261900950207643349335910602454508645362893545686295853131533718386826561786227363716975774183023986006591481616404944965011732131389574706208847480236537103115089842799275442685327797431139514357417221975979935968525228574526379628961269157235798662057340837576687388426640599099350500081337543245463596750484423528487470144354541957625847356421619813407346854111766883118654489377697956651727966232671481033864391375186594673002443450054499539974237232871249483470604406347160632583064982979551010954183623503030945309733583446283947630477564501500850757894954893139394489921612552559770143685894358587752637962559708167764380012543650237141278346792610199558522471722017772370041780841942394872540680155603599839054898572354674564239058585021671903139526294455439131663134530893906204678438778505423939052473136201294769187497519101147231528932677253391814660730008902776896311481090220972452075916729700785058071718638105496797310016787085069420709223290807038326345345203802786099055690013413718236837099194951648960075504934126787643674638490206396401976668559233565463913836318574569814719621084108096188460545603903845534372914144651347494078488442377217515433426030669883176833100113310869042193903108014378433415137092435301367763108491351615642269847507430329716746964066653152703532546711266752246055119958183196376370761799191920357958200759560530234626775794393630746305690108011494271410093913691381072581378135789400559950018354251184172136055727522103526803735726527922417373605751127887218190844900617801388971077082293100279766593583875890939568814856026322439372656247277603789081445883785501970284377936240782505270487581647032458129087839523245323789602984166922548964971560698119218658492677040395648127810217991321741630581055459880130048456299765112124153637451500563507012781592671424134210330156616535602473380784302865525722275304999883701534879300806260180962381516136690334111138653851091936739383522934588832255088706450753947395204396807906708680644509698654880168287434378612645381583428075306184548590379821799459968115441974253634439960290251001588827216474500682070419376158454712318346007262933955054823955713725684023226821301247679452264482091023564775272308208106351889915269288910845557112660396503439789627825001611015323516051965590421184494990778999200732947690586857787872098290135295661397888486050978608595701773129815531495168146717695976099421003618355913877781769845875810446628399880600616229848616935337386578773598336161338413385368421197893890018529569196780455448285848370117096721253533875862158231013310387766827211572694951817958975469399264219791552338576623167627547570354699414892904130186386119439196283887054367774322427680913236544948536676800000106526248547305586159899914017076983854831887501429389089950685453076511680333732226517566220752695179144225280816517166776672793035485154204023817460892328391703275425750867655117859395002793389592057668278967764453184040418554010435134838953120132637836928358082719378312654961745997056745071833206503455664403449045362756001125018433560736122276594927839370647842645676338818807565612168960504161139039063960162022153684941092605387688714837989559999112099164646441191856827700457424343402167227644558933012778158686952506949936461017568506016714535431581480105458860564550133203758645485840324029871709348091055621167154684847780394475697980426318099175642280987399876697323769573701580806822904599212366168902596273043067931653114940176473769387351409336183321614280214976339918983548487562529875242387307755955595546519639440182184099841248982623673771467226061633643296406335728107078875816404381485018841143188598827694490119321296827158884133869434682859006664080631407775772570563072940049294030242049841656547973670548558044586572022763784046682337985282710578431975354179501134727362577408021347682604502285157979579764746702284099956160156910890384582450267926594205550395879229818526480070683765041836562094555434613513415257006597488191634135955671964965403218727160264859304903978748958906612725079482827693895352175362185079629778514618843271922322381015874445052866523802253284389137527384589238442253547265309817157844783421582232702069028723233005386216347988509469547200479523112015043293226628272763217790884008786148022147537657810581970222630971749507212724847947816957296142365859578209083073323356034846531873029302665964501371837542889755797144992465403868179921389346924474198509733462679332107268687076806263991936196504409954216762784091466985692571507431574079380532392523947755744159184582156251819215523370960748332923492103451462643744980559610330799414534778457469999212859999939961228161521931488876938802228108300198601654941654261696858678837260958774567618250727599295089318052187292461086763995891614585505839727420980909781729323930106766386824040111304024700735085782872462713494636853181546969046696869392547251941399291465242385776255004748529547681479546700705034799958886769501612497228204030399546327883069597624936151010243655535223069061294938859901573466102371223547891129254769617600504797492806072126803922691102777226102544149221576504508120677173571202718024296810620377657883716690910941807448781404907551782038565390991047759414132154328440625030180275716965082096427348414695726397884256008453121406593580904127113592004197598513625479616063228873618136737324450607924411763997597461938358457491598809766744709300654634242346063423747466608043170126005205592849369594143408146852981505394717890045183575515412522359059068726487863575254191128887737176637486027660634960353679470269232297186832771739323619200777452212624751869833495151019864269887847171939664976907082521742336566272592844062043021411371992278526998469884770232382384005565551788908766136013047709843861168705231055314916251728373272867600724817298763756981633541507460883866364069347043720668865127568826614973078865701568501691864748854167915459650723428773069985371390430026653078398776385032381821553559732353068604301067576083890862704984188859513809103042359578249514398859011318583584066747237029714978508414585308578133915627076035639076394731145549583226694570249413983163433237897595568085683629725386791327505554252449194358912840504522695381217913191451350099384631177401797151228378546011603595540286440590249646693070776905548102885020808580087811577381719174177601733073855475800605601433774329901272867725304318251975791679296996504146070664571258883469797964293162296552016879730003564630457930884032748077181155533090988702550520768046303460865816539487695196004408482065967379473168086415645650530049881616490578831154345485052660069823093157776500378070466126470602145750579327096204782561524714591896522360839664562410519551052235723973951288181640597859142791481654263289200428160913693777372229998332708208296995573772737566761552711392258805520189887620114168005468736558063347160373429170390798639652296131280178267971728982293607028806908776866059325274637840539769184808204102194471971386925608416245112398062011318454124478205011079876071715568315407886543904121087303240201068534194723047666672174986986854707678120512473679247919315085644477537985379973223445612278584329684664751333657369238720146472367942787004250325558992688434959287612400755875694641370562514001179713316620715371543600687647731867558714878398908107429530941060596944315847753970094398839491443235366853920994687964506653398573888786614762944341401049888993160051207678103588611660202961193639682134960750111649832785635316145168457695687109002999769841263266502347716728657378579085746646077228341540311441529418804782543876177079043000156698677679576090996693607559496515273634981189641304331166277471233881740603731743970540670310967676574869535878967003192586625941051053358438465602339179674926784476370847497833365557900738419147319886271352595462518160434225372996286326749682405806029642114638643686422472488728343417044157348248183330164056695966886676956349141632842641497453334999948000266998758881593507357815195889900539512085351035726137364034367534714104836017546488300407846416745216737190483109676711344349481926268111073994825060739495073503169019731852119552635632584339099822498624067031076831844660729124874754031617969941139738776589986855417031884778867592902607004321266617919223520938227878880988633599116081923535557046463491132085918979613279131975649097600013996234445535014346426860464495862476909434704829329414041114654092398834443515913320107739441118407410768498106634724104823935827401944935665161088463125678529776973468430306146241803585293315973458303845541033701091676776374276210213701354854450926307190114731848574923318167207213727935567952844392548156091372812840633303937356242001604566455741458816605216660873874804724339121295587776390696903707882852775389405246075849623157436917113176134783882719416860662572103685132156647800147675231039357860689611125996028183930954870905907386135191459181951029732787557104972901148717189718004696169777001791391961379141716270701895846921434369676292745910994006008498356842520191559370370101104974733949387788598941743303178534870760322198297057975119144051099423588303454635349234982688362404332726741554030161950568065418093940998202060999414021689090070821330723089662119775530665918814119157783627292746156185710372172471009521423696483086410259288745799932237495519122195190342445230753513380685680735446499512720317448719540397610730806026990625807602029273145525207807991418429063884437349968145827337207266391767020118300464819000241308350884658415214899127610651374153943565721139032857491876909441370209051703148777346165287984823533829726013611098451484182380812054099612527458088109948697221612852489742555551607637167505489617301680961380381191436114399210638005083214098760459930932485102516829446726066613815174571255975495358023998314698220361338082849935670557552471290274539776214049318201465800802156653606776550878380430413431059180460680083459113664083488740800574127258670479225831912741573908091438313845642415094084913391809684025116399193685322555733896695374902662092326131885589158083245557194845387562878612885900410600607374650140262782402734696252821717494158233174923968353013617865367376064216677813773995100658952887742766263684183068019080460984980946976366733566228291513235278880615776827815958866918023894033307644191240341202231636857786035727694154177882643523813190502808701857504704631293335375728538660588890458311145077394293520199432197117164223500564404297989208159430716701985746927384865383343614579463417592257389858800169801475742054299580124295810545651083104629728293758416116253256251657249807849209989799062003593650993472158296517413579849104711166079158743698654122234834188772292944633517865385673196255985202607294767407261676714557364981210567771689348491766077170527718760119990814411305864557791052568430481144026193840232247093924980293355073184589035539713308844617410795916251171486487446861124760542867343670904667846867027409188101424971114965781772427934707021668829561087779440504843752844337510882826477197854000650970403302186255614733211777117441335028160884035178145254196432030957601869464908868154528562134698835544456024955666843660292219512483091060537720198021831010327041783866544718126039719068846237085751808003532704718565949947612424811099928867915896904956394762460842406593094862150769031498702067353384834955083636601784877106080980426924713241000946401437360326564518456679245666955100150229833079849607994988249706172367449361226222961790814311414660941234159359309585407913908720832273354957208075716517187659944985693795623875551617575438091780528029464200447215396280746360211329425591600257073562812638733106005891065245708024474937543184149401482119996276453106800663118382376163966318093144467129861552759820145141027560068929750246304017351489194576360789352855505317331416457050499644389093630843874484783961684051845273288403234520247056851646571647713932377551729479512613239822960239454857975458651745878771331813875295980941217422730035229650808917770506825924882232215493804837145478164721397682096332050830564792048208592047549985732038887639160199524091893894557676874973085695595801065952650303626615975066222508406742889826590751063756356996821151</p>        
  </div>
</div>
        </main>
    </div>

    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; 2019 - Pi.Web - <a href="/Home/Privacy">Privacy</a>
        </div>
    </footer>
    <script src="/lib/jquery/dist/jquery.min.js"></script>
    <script src="/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="/js/site.js?v=BxFAw9RUJ1E4NycpKEjCNDeoSvr4RPHixdBq5wDnkeY"></script>
    
</body>
</html>
```

### refresh your browser

### run a Pod with a volume mount to the host:
`kubectl apply -f sleep/sleep-with-hostPath.yaml`
```
deployment.apps/sleep configured
```

### check the log files inside the container:
`kubectl exec deploy/sleep -- ls -l /var/log`
```
total 0
```

### check the logs on the node using the volume:
`kubectl exec deploy/sleep -- ls -l /node-root/var/log`
```
total 0
drwxr-xr-x    2 root     root           300 Oct  4 14:55 containers
drwxr-xr-x   15 root     root           300 Oct  4 14:55 pods
```

### check the container user:
`kubectl exec deploy/sleep -- whoami`
```
root
```

### update the Pod spec:
`kubectl apply -f sleep/sleep-with-hostPath-subPath.yaml`
```
deployment.apps/sleep configured
```

### check the Pod logs on the node:
`kubectl exec deploy/sleep -- sh -c 'ls /pod-logs | grep _pi-'`
 
### check the container logs:
`kubectl exec deploy/sleep -- sh -c 'ls /container-logs | grep nginx'`
```
pi-proxy-fbf5595-z72vn_default_nginx-3e3867270569fbffc25d50c839fa821c52a2db352314a2e442db42afaf65e7e0.log
```


# Section 5.3: Storing clusterwide data with persistent volumes and claims

### apply a custom label to the first node in your cluster: 
`kubectl label node $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') kiamol=ch05`
```
node/docker-desktop labeled
```

### check the nodes with a label selector:
`kubectl get nodes -l kiamol=ch05`
```
NAME             STATUS   ROLES           AGE   VERSION
docker-desktop   Ready    control-plane   20d   v1.27.2
```

### deploy a PV that uses a local volume on the labeled node:
`kubectl apply -f todo-list/persistentVolume.yaml`
```
persistentvolume/pv01 created
```

### check the PV:
`kubectl get pv`
```
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv01   50Mi       RWO            Retain           Available                                   7s
```

### Listing 5.6:
Not sure I get this:
```
storageClassName: ""        # A blank class means a PV needs to exist.
```
Ah, from [here](https://kubernetes.io/docs/concepts/storage/storage-classes/), I see,
> When a PVC does not specify a `storageClassName`, the default StorageClass is used.

### create a PVC that will bind to the PV:
`kubectl apply -f todo-list/postgres-persistentVolumeClaim.yaml`
```
persistentvolumeclaim/postgres-pvc created
```
 
### check PVCs:
`kubectl get pvc`
```
NAME           STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-pvc   Bound    pv01     50Mi       RWO                           10s
```

### check PVs:
`kubectl get pv`
```
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
pv01   50Mi       RWO            Retain           Bound    default/postgres-pvc                           6m22s
```


### create a PVC that doesn’t match any available PVs:
`kubectl apply -f todo-list/postgres-persistentVolumeClaim-too-big.yaml`
```
persistentvolumeclaim/postgres-pvc-toobig created
```

### check claims:
`kubectl get pvc`
```
NAME                  STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-pvc          Bound     pv01     50Mi       RWO                           87s
postgres-pvc-toobig   Pending                                                     15s
```

### run the sleep Pod, which has access to the node’s disk:
`kubectl apply -f sleep/sleep-with-hostPath.yaml`
```
deployment.apps/sleep configured
```

### wait for the Pod to be ready:
`kubectl wait --for=condition=Ready pod -l app=sleep`
```
pod/sleep-6f58497b9f-5j6jt condition met
```

### create the directory path on the node, which the PV expects:
`kubectl exec deploy/sleep -- mkdir -p /node-root/volumes/pv01`


### deploy the database:
`kubectl apply -f todo-list/postgres/`
```
secret/todo-db-secret created
service/todo-db created
deployment.apps/todo-db created
```
 
### wait for Postgres to initialize:
`sleep 30`
 
### check the database logs: 
`kubectl logs -l app=todo-db --tail 1`
```
2023-10-05 14:05:42.275 UTC [1] LOG:  database system is ready to accept connections
```
 
### check the data files in the volume:
`kubectl exec deploy/sleep -- sh -c 'ls -l /node-root/volumes/pv01 | grep wal'`
```
drwx------    3 70       70              80 Oct  5 14:05 pg_wal
```
Additional investigation:
```bash
λ kubectl exec deploy/sleep -- sh -c 'ls -l /node-root/volumes/pv01/pg_wal'
total 16384
-rw-------    1 70       70        16777216 Oct  5 14:05 000000010000000000000001
drwx------    2 70       70              40 Oct  5 14:05 archive_status

λ kubectl exec deploy/sleep -- sh -c 'ls -al /node-root/volumes/pv01/pg_wal/archive_status'
total 0
drwx------    2 70       70              40 Oct  5 14:05 .
drwx------    3 70       70              80 Oct  5 14:05 ..

λ kubectl exec deploy/sleep -- sh -c 'cat /node-root/volumes/pv01/pg_wal/000000010000000000000001'
```
That printed gibberish, but I see segments like `pg_partitioned_table` and `pg_statistic_ext`.

### deploy the web app components:
`kubectl apply -f todo-list/web/`
```
configmap/todo-web-config created
secret/todo-web-secret created
service/todo-web created
deployment.apps/todo-web created
```
 
### wait for the web Pod:
`kubectl wait --for=condition=Ready pod -l app=todo-web`
```
pod/todo-web-679889ff7-j7lrj condition met
```
 
### get the app URL from the Service:
`kubectl get svc todo-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8081/new'`
```
http://localhost:8081/new
```

### browse to the app, and add a new item
```
This page isn’t workinglocalhost is currently unable to handle this request.
HTTP ERROR 500
```
So I'll skip the next steps :-(
### delete the database Pod:
`kubectl delete pod -l app=todo-db`
### check the contents of the volume on the node:
`kubectl exec deploy/sleep -- ls -l /node-root/volumes/pv01/pg_wal`
 
### check that your item is still in the to-do list

# Section 5.4: Dynamic volume provisioning and storage classes

### deploy the PVC from listing 5.8:
`kubectl apply -f todo-list/postgres-persistentVolumeClaim-dynamic.yaml`
```
persistentvolumeclaim/postgres-pvc-dynamic created
```

### check claims and volumes:
`kubectl get pvc`
`kubectl get pv`
```
NAME                   STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-pvc           Bound     pv01                                       50Mi       RWO                           13m
postgres-pvc-dynamic   Bound     pvc-da17382a-f0ee-43fc-9ded-126ec784daa2   100Mi      RWO            hostpath       8s
postgres-pvc-toobig    Pending                                                                                       12m

NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                          STORAGECLASS   REASON   AGE
pv01                                       50Mi       RWO            Retain           Bound    default/postgres-pvc                                   19m
pvc-da17382a-f0ee-43fc-9ded-126ec784daa2   100Mi      RWO            Delete           Bound    default/postgres-pvc-dynamic   hostpath                16s
```

### delete the claim:
`kubectl delete pvc postgres-pvc-dynamic`
```
persistentvolumeclaim "postgres-pvc-dynamic" deleted
```

### check volumes again:
`kubectl get pv`
```
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
pv01   50Mi       RWO            Retain           Bound    default/postgres-pvc                           20m
```

`kubectl get pvc`
```
NAME                  STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-pvc          Bound     pv01     50Mi       RWO                           15m
postgres-pvc-toobig   Pending                                                     13m
```


### list the storage classes in the cluster:
`kubectl get storageclass`
```
NAME                 PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
hostpath (default)   docker.io/hostpath   Delete          Immediate           false                  20d
```
 
### clone the default on Windows:
`Set-ExecutionPolicy Bypass -Scope Process -Force; ./cloneDefaultStorageClass.ps1`
 
### OR on Mac/Linux:
`chmod +x cloneDefaultStorageClass.sh && ./cloneDefaultStorageClass.sh`
```
configmap/clone-script created
pod/clone-sc created
pod/clone-sc condition met
storageclass.storage.k8s.io/kiamol created
configmap "clone-script" deleted
pod "clone-sc" deleted
```

### list storage classes:
`kubectl get sc`
```
NAME                 PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
hostpath (default)   docker.io/hostpath   Delete          Immediate           false                  20d
kiamol               docker.io/hostpath   Delete          Immediate           false                  49s
```

### create a new PVC using the custom storage class:
`kubectl apply -f storageClass/postgres-persistentVolumeClaim-storageClass.yaml`
```
persistentvolumeclaim/postgres-pvc-kiamol created
```

### update the database to use the new PVC:
`kubectl apply -f storageClass/todo-db.yaml`
```
deployment.apps/todo-db configured
```

### check the storage:
`kubectl get pvc`
`kubectl get pv`
```
NAME                  STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-pvc          Bound     pv01                                       50Mi       RWO                           20m
postgres-pvc-kiamol   Bound     pvc-3694c25d-f61d-411f-83d6-4cae06552867   100Mi      RWO            kiamol         20s
postgres-pvc-toobig   Pending                                                                                       19m

NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS   REASON   AGE
pv01                                       50Mi       RWO            Retain           Bound    default/postgres-pvc                                  26m
pvc-3694c25d-f61d-411f-83d6-4cae06552867   100Mi      RWO            Delete           Bound    default/postgres-pvc-kiamol   kiamol                  30s
```
 
### check the Pods:
`kubectl get pods -l app=todo-db`
```
NAME                       READY   STATUS    RESTARTS   AGE
todo-db-745877d464-6pxvr   1/1     Running   0          34s
```
 
### refresh the list in your to-do app
Adding items is still not working, but I can still access `http://localhost:8081/new`. The list is (still) empty.

# Section 5.5: Understanding storage choices in Kubernetes

### delete deployments, PVCs, PVs, and Services:
`kubectl delete -f pi/v1 -f sleep/ -f storageClass/ -f todo-list/web -f todo-list/postgres -f todo-list/`
```
configmap "pi-proxy-configmap" deleted
service "pi-proxy" deleted
deployment.apps "pi-proxy" deleted
service "pi-web" deleted
deployment.apps "pi-web" deleted
deployment.apps "sleep" deleted
persistentvolumeclaim "postgres-pvc-kiamol" deleted
deployment.apps "todo-db" deleted
configmap "todo-web-config" deleted
secret "todo-web-secret" deleted
service "todo-web" deleted
deployment.apps "todo-web" deleted
secret "todo-db-secret" deleted
service "todo-db" deleted
persistentvolume "pv01" deleted
persistentvolume "pv01" deleted
persistentvolumeclaim "postgres-pvc-toobig" deleted
persistentvolumeclaim "postgres-pvc" deleted
Error from server (NotFound): error when deleting "sleep/sleep-with-emptyDir.yaml": deployments.apps "sleep" not found
Error from server (NotFound): error when deleting "sleep/sleep-with-hostPath-subPath.yaml": deployments.apps "sleep" not found
Error from server (NotFound): error when deleting "sleep/sleep-with-hostPath.yaml": deployments.apps "sleep" not found
Error from server (NotFound): error when deleting "sleep/sleep-with-projected.yaml": deployments.apps "sleep" not found
Error from server (NotFound): error when deleting "sleep/sleep.yaml": deployments.apps "sleep" not found
Error from server (NotFound): error when deleting "storageClass/clone-storageClass-script.yaml": configmaps "clone-script" not found
Error from server (NotFound): error when deleting "storageClass/clone-storageClass-script.yaml": pods "clone-sc" not found
Error from server (NotFound): error when deleting "todo-list/postgres/todo-db.yaml": deployments.apps "todo-db" not found
Error from server (NotFound): error when deleting "todo-list/postgres-persistentVolumeClaim-dynamic.yaml": persistentvolumeclaims "postgres-pvc-dynamic" not found
```
 
### delete the custom storage class:
`kubectl delete sc kiamol`
```
storageclass.storage.k8s.io "kiamol" deleted
```
When I ran that again, I got:
```
Error from server (NotFound): storageclasses.storage.k8s.io "kiamol" not found
```

# Chapter 5 Lab
I am just running the steps from the README, including the sample solution.
I had to modify the third command to be `kubectl logs -l app=todo-web-lab`.

Most of `http://localhost:8082` doesn't work for me, except `http://localhost:8082/diagnostics`.


# Chapter 6: Scaling applications across multiple Pods with controllers

Figure 6.1: "Every software problem can be solved by adding another layer of abstraction."


### switch to this chapter's exercises:
`cd ch06`

### deploy the ReplicaSet and Service:
`kubectl apply -f whoami/`
```
Unable to connect to the server: EOF
```

`kubectl get all`
```
E1012 12:09:05.835511    8875 memcache.go:265] couldn't get current server API group list: Get "https://kubernetes.docker.internal:6443/api?timeout=32s": EOF
E1012 12:09:15.891282    8875 memcache.go:265] couldn't get current server API group list: Get "https://kubernetes.docker.internal:6443/api?timeout=32s": EOF
^C
```

Reseting Kubernetes Cluster through Docker Desktop... That worked!

```
service/whoami-web created
replicaset.apps/whoami-web created
```

### check the resource:
`kubectl get replicaset whoami-web`
```
NAME         DESIRED   CURRENT   READY   AGE
whoami-web   1         1         1       19s
```

### make an HTTP GET call to the Service:
`curl $(kubectl get svc whoami-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8088')`
```
"I'm whoami-web-pp7d7 running on Linux 6.4.16-linuxkit #1 SMP PREEMPT Sat Sep 23 13:36:48 UTC 2023"
```

### delete all the Pods:
`kubectl delete pods -l app=whoami-web`
```
pod "whoami-web-pp7d7" deleted
```

### repeat the HTTP call:
`curl $(kubectl get svc whoami-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8088')`
```
"I'm whoami-web-tct2n running on Linux 6.4.16-linuxkit #1 SMP PREEMPT Sat Sep 23 13:36:48 UTC 2023"
```

### show the detail about the ReplicaSet:
`kubectl describe rs whoami-web`
```
Name:         whoami-web
Namespace:    default
Selector:     app=whoami-web
Labels:       kiamol=ch06
Annotations:  <none>
Replicas:     1 current / 1 desired
Pods Status:  1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=whoami-web
  Containers:
   web:
    Image:        kiamol/ch02-whoami
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  58s   replicaset-controller  Created pod: whoami-web-pp7d7
  Normal  SuccessfulCreate  19s   replicaset-controller  Created pod: whoami-web-tct2n
```


### deploy the update:
`kubectl apply -f whoami/update/whoami-replicas-3.yaml`
```
replicaset.apps/whoami-web configured
```
 
### check Pods:
`kubectl get pods -l app=whoami-web`
```
NAME               READY   STATUS    RESTARTS   AGE
whoami-web-jsnpt   1/1     Running   0          5s
whoami-web-tct2n   1/1     Running   0          4m3s
whoami-web-z928n   1/1     Running   0          5s
```
 
### delete all the Pods:
`kubectl delete pods -l app=whoami-web`
```
pod "whoami-web-jsnpt" deleted
pod "whoami-web-tct2n" deleted
pod "whoami-web-z928n" deleted
```

### check again:
`kubectl get pods -l app=whoami-web`
```
NAME               READY   STATUS    RESTARTS   AGE
whoami-web-59586   1/1     Running   0          11s
whoami-web-8qbbf   1/1     Running   0          11s
whoami-web-d47qj   1/1     Running   0          11s
```
 
### repeat this HTTP call a few times:
`curl $(kubectl get svc whoami-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8088')`
```
Thu Oct 12 12:16:04
~/GitHub/kiamol/ch06
paulkaefer ~/GitHub/kiamol/ch06 λ curl $(kubectl get svc whoami-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8088')
"I'm whoami-web-8qbbf running on Linux 6.4.16-linuxkit #1 SMP PREEMPT Sat Sep 23 13:36:48 UTC 2023"
Thu Oct 12 12:16:20
~/GitHub/kiamol/ch06
paulkaefer ~/GitHub/kiamol/ch06 λ curl $(kubectl get svc whoami-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8088')
"I'm whoami-web-59586 running on Linux 6.4.16-linuxkit #1 SMP PREEMPT Sat Sep 23 13:36:48 UTC 2023"
Thu Oct 12 12:16:20
~/GitHub/kiamol/ch06
paulkaefer ~/GitHub/kiamol/ch06 λ curl $(kubectl get svc whoami-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8088')
"I'm whoami-web-59586 running on Linux 6.4.16-linuxkit #1 SMP PREEMPT Sat Sep 23 13:36:48 UTC 2023"
Thu Oct 12 12:16:22
~/GitHub/kiamol/ch06
paulkaefer ~/GitHub/kiamol/ch06 λ curl $(kubectl get svc whoami-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8088')
"I'm whoami-web-8qbbf running on Linux 6.4.16-linuxkit #1 SMP PREEMPT Sat Sep 23 13:36:48 UTC 2023"
```
Ran a few more, and got `d47qj` then `8qbbf` and then those two repeated a couple times. I ran more, and saw the `59586` again after a bit.


### run a sleep Pod:
`kubectl apply -f sleep.yaml`
```
deployment.apps/sleep created
```
 
### check the details of the who-am-I Service:
`kubectl get svc whoami-web`
```
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
whoami-web   LoadBalancer   10.110.36.180   localhost     8088:31562/TCP   8m30s
```

### run a DNS lookup for the Service in the sleep Pod:
`kubectl exec deploy/sleep -- sh -c 'nslookup whoami-web | grep "^[^*]"'`
```
Server:   10.96.0.10
Address:  10.96.0.10:53
Name: whoami-web.default.svc.cluster.local
Address: 10.110.36.180
```
 
### make some HTTP calls:
`kubectl exec deploy/sleep -- sh -c 'for i in 1 2 3; do curl -w \\n -s http://whoami-web:8088; done;'`
```
"I'm whoami-web-8qbbf running on Linux 6.4.16-linuxkit #1 SMP PREEMPT Sat Sep 23 13:36:48 UTC 2023"
"I'm whoami-web-59586 running on Linux 6.4.16-linuxkit #1 SMP PREEMPT Sat Sep 23 13:36:48 UTC 2023"
"I'm whoami-web-59586 running on Linux 6.4.16-linuxkit #1 SMP PREEMPT Sat Sep 23 13:36:48 UTC 2023"
```


### deploy the Pi app:
`kubectl apply -f pi/web/`
```
service/pi-web created
deployment.apps/pi-web created
```

### check the ReplicaSet:
`kubectl get rs -l app=pi-web`
```
NAME               DESIRED   CURRENT   READY   AGE
pi-web-55b6cb574   2         2         2       8s
```

### scale up to more replicas:
`kubectl apply -f pi/web/update/web-replicas-3.yaml`
```
deployment.apps/pi-web configured
```
Cool. I see the YAML file has `replicas: 3`.

### check the RS:
`kubectl get rs -l app=pi-web`
```
NAME               DESIRED   CURRENT   READY   AGE
pi-web-55b6cb574   3         3         3       40s
```

### deploy a changed Pod spec with enhanced logging:
`kubectl apply -f pi/web/update/web-logging-level.yaml`
```
deployment.apps/pi-web configured
```

### check ReplicaSets again:
`kubectl get rs -l app=pi-web`
```
NAME                DESIRED   CURRENT   READY   AGE
pi-web-55b6cb574    0         0         0       62s
pi-web-658956d56f   3         3         3       12s
```


### we need to scale the Pi app fast:
`kubectl scale --replicas=4 deploy/pi-web`
```
deployment.apps/pi-web scaled
```

### check which ReplicaSet makes the change:
`kubectl get rs -l app=pi-web`
```
NAME                DESIRED   CURRENT   READY   AGE
pi-web-55b6cb574    0         0         0       2m6s
pi-web-658956d56f   4         4         4       76s
```

### now we can revert back to the original logging level:
`kubectl apply -f pi/web/update/web-replicas-3.yaml`
```
deployment.apps/pi-web configured
```

### but that will undo the scale we set manually:
`kubectl get rs -l app=pi-web`
```
NAME                DESIRED   CURRENT   READY   AGE
pi-web-55b6cb574    3         3         3       2m24s
pi-web-658956d56f   0         0         0       94s
```

### check the Pods:
`kubectl get pods -l app=pi-web`
```
NAME                     READY   STATUS    RESTARTS   AGE
pi-web-55b6cb574-4cqqc   1/1     Running   0          18s
pi-web-55b6cb574-kzngl   1/1     Running   0          16s
pi-web-55b6cb574-xx7bh   1/1     Running   0          14s
```

Probably best to not scale manually.

### list ReplicaSets with labels:
`kubectl get rs -l app=pi-web  --show-labels`
```
NAME                DESIRED   CURRENT   READY   AGE     LABELS
pi-web-55b6cb574    3         3         3       3m44s   app=pi-web,pod-template-hash=55b6cb574
pi-web-658956d56f   0         0         0       2m54s   app=pi-web,pod-template-hash=658956d56f
```

### list Pods with labels:
`kubectl get po -l app=pi-web  --show-labels`
```
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
pi-web-55b6cb574-4cqqc   1/1     Running   0          98s   app=pi-web,pod-template-hash=55b6cb574
pi-web-55b6cb574-kzngl   1/1     Running   0          96s   app=pi-web,pod-template-hash=55b6cb574
pi-web-55b6cb574-xx7bh   1/1     Running   0          94s   app=pi-web,pod-template-hash=55b6cb574
```

### deploy the proxy resources:
`kubectl apply -f pi/proxy/`
```
configmap/pi-proxy-configmap created
service/pi-proxy created
deployment.apps/pi-proxy created
```

### get the URL to the proxied app:
`kubectl get svc whoami-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8080/?dp=10000'`
```
http://localhost:8080/?dp=10000
```

### browse to the app, and try a few different values for 'dp' in the URL
```
To: 10,000 d.p.
in: 130 ms.
from: pi-web-55b6cb574-xx7bh

To: 20,000 d.p.
in: 433 ms.
from: pi-web-55b6cb574-kzngl

To: 150,000 d.p.
in: 15,226 ms.
from: pi-web-55b6cb574-4cqqc
-- ends in 04283622782

Tried with 250000; got
504 Gateway Time-out
nginx/1.17.10

To: 5,000 d.p.
in: 29 ms.
from: pi-web-55b6cb574-xx7bh
```

See `ch06/Screenshot 2023-10-12 at 12.53.35 PM.png` for my equivalent of Figure 6.10.
Then `ch06/Screenshot 2023-10-12 at 12.55.22 PM.png`, which was after I refreshed.

I suspect the caching could be improved... `http://localhost:8080/?dp=240000` took 16,254 ms.
Ah, perhaps the "It would be nice to fix this by using shared storage, so every proxy Pod had access to the same cache." partially explains this.

I stopped at `TRY IT NOW Deploy an update to the proxy spec`.

### Resuming 2023-10-31:
```bash
λ kubectl get all
NAME                            READY   STATUS    RESTARTS        AGE
pod/pi-proxy-866647b9c9-8lsv4   1/1     Running   6 (2m44s ago)   18d
pod/pi-proxy-866647b9c9-skxj6   1/1     Running   6 (2m44s ago)   18d
pod/pi-web-55b6cb574-4cqqc      1/1     Running   2 (3m16s ago)   18d
pod/pi-web-55b6cb574-kzngl      1/1     Running   2 (3m16s ago)   18d
pod/pi-web-55b6cb574-xx7bh      1/1     Running   2 (26h ago)     18d
pod/sleep-8648c6f777-k7mp7      1/1     Running   2 (26h ago)     18d
pod/whoami-web-59586            1/1     Running   2 (26h ago)     18d
pod/whoami-web-8qbbf            1/1     Running   2 (26h ago)     18d
pod/whoami-web-d47qj            1/1     Running   2 (26h ago)     18d

NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          18d
service/pi-proxy     LoadBalancer   10.100.94.73    localhost     8080:30747/TCP   18d
service/pi-web       ClusterIP      10.99.175.99    <none>        80/TCP           18d
service/whoami-web   LoadBalancer   10.110.36.180   localhost     8088:31562/TCP   18d

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/pi-proxy   2/2     2            2           18d
deployment.apps/pi-web     3/3     3            3           18d
deployment.apps/sleep      1/1     1            1           18d

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/pi-proxy-866647b9c9   2         2         2       18d
replicaset.apps/pi-web-55b6cb574      3         3         3       18d
replicaset.apps/pi-web-658956d56f     0         0         0       18d
replicaset.apps/sleep-8648c6f777      1         1         1       18d
replicaset.apps/whoami-web            3         3         3       18d
```

### deploy the updated spec:
`kubectl apply -f pi/proxy/update/nginx-hostPath.yaml`
```
deployment.apps/pi-proxy configured
```
 
### check the Pods--the new spec adds a third replica:
`kubectl get po -l app=pi-proxy`
```
NAME                        READY   STATUS    RESTARTS   AGE
pi-proxy-684954445c-95kjt   1/1     Running   0          7s
pi-proxy-684954445c-pgsgr   1/1     Running   0          6s
pi-proxy-684954445c-pmkld   1/1     Running   0          8s
```
 
### browse back to the Pi app, and refresh it a few times
http://localhost:8080/
 
### check the proxy logs:
`kubectl logs -l app=pi-proxy --tail 1`
```
192.168.65.3 - - [31/Oct/2023:15:15:21 +0000] "GET /?dp=100 HTTP/1.1" 200 1102 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36"
192.168.65.3 - - [31/Oct/2023:15:15:21 +0000] "GET /?dp=100 HTTP/1.1" 200 1102 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36"
192.168.65.3 - - [31/Oct/2023:15:15:03 +0000] "GET / HTTP/1.1" 200 1019 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36"
```

# Section 6.3: Scaling for high availability with DaemonSets

### deploy the DaemonSet:
`kubectl apply -f pi/proxy/daemonset/nginx-ds.yaml`
```
daemonset.apps/pi-proxy created
```
 
### check the endpoints used in the proxy service:
`kubectl get endpoints pi-proxy`
```
NAME       ENDPOINTS                                               AGE
pi-proxy   10.1.0.147:80,10.1.0.149:80,10.1.0.150:80 + 1 more...   18d
```

### delete the Deployment:
`kubectl delete deploy pi-proxy`
```
deployment.apps "pi-proxy" deleted
```

### check the DaemonSet:
`kubectl get daemonset pi-proxy`
```
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
pi-proxy   1         1         1       1            1           <none>          45s
```

### check the Pods:
`kubectl get po -l app=pi-proxy`
```
NAME             READY   STATUS    RESTARTS   AGE
pi-proxy-8lkwh   1/1     Running   0          58s
```
Maybe I wasn't fast enough to see the Terminating... like in Figure 6.13?

### refresh your latest Pi calculation on the browser

### check the status of the DaemonSet:
`kubectl get ds pi-proxy`
```
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
pi-proxy   1         1         1       1            1           <none>          3m5s
```
 
### delete its Pod:
`kubectl delete po -l app=pi-proxy`
Ran twice by accident:
```
pod "pi-proxy-8lkwh" deleted
pod "pi-proxy-xfp68" deleted
```

I did see the following if I reload just after I run the delete command:
```
This page isn’t working
localhost didn’t send any data.
ERR_EMPTY_RESPONSE
```
 
### check the Pods:
`kubectl get po -l app=pi-proxy`
```
NAME             READY   STATUS    RESTARTS   AGE
pi-proxy-skk9w   1/1     Running   0          29s
```

Looked @ browser:
```
Pi.Web
Source
Pi
To: 10,000 d.p.
in: 145 ms.
from: pi-web-55b6cb574-xx7bh
3.141592653589793238462643383279502884197169399375105820974944592307816406286208998628034825342117067982148086513282306647093844609550582231725359408128481117450284102701938521105559644622948954930381964428810975665933446128475648233786783165271201909145648566923460348610454326648213393607260249141273724587006606315588174881520920962829254091715364367
```

### update the DaemonSet spec:
`kubectl apply -f pi/proxy/daemonset/nginx-ds-nodeSelector.yaml`
```
daemonset.apps/pi-proxy configured
```

### check the DS:
`kubectl get ds pi-proxy`
```
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
pi-proxy   0         0         0       0            0           kiamol=ch06     33m
```

### check the Pods:
`kubectl get po -l app=pi-proxy`
```
No resources found in default namespace.
```
 
### now label a node in your cluster so it matches the selector:
`kubectl label node $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') kiamol=ch06 --overwrite`
```
node/docker-desktop labeled
```
 
### check the Pods again:
`kubectl get ds pi-proxy`
```
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
pi-proxy   1         1         1       1            1           kiamol=ch06     35m
```

### delete the DaemonSet, but leave the Pod alone: 
`kubectl delete ds pi-proxy --cascade=false`
```
warning: --cascade=false is deprecated (boolean value) and can be replaced with --cascade=orphan.
```
`kubectl delete ds pi-proxy --cascade=orphan`
 
### check the Pod:
`kubectl get po -l app=pi-proxy`
```
NAME             READY   STATUS    RESTARTS   AGE
pi-proxy-sklbm   1/1     Running   0          2m34s
```
 
### recreate the DS:
`kubectl apply -f pi/proxy/daemonset/nginx-ds-nodeSelector.yaml`
```
daemonset.apps/pi-proxy created
```
 
### check the DS and Pod:
```bash
kubectl get ds pi-proxy
 
kubectl get po -l app=pi-proxy
```
Results:
```
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
pi-proxy   1         1         1       1            1           kiamol=ch06     0s

NAME             READY   STATUS    RESTARTS   AGE
pi-proxy-sklbm   1/1     Running   0          2m34s
```
 
### delete the DS again, without the cascade option:
`kubectl delete ds pi-proxy`
```
daemonset.apps "pi-proxy" deleted
```
 
### check the Pods:
`kubectl get po -l app=pi-proxy`
```
NAME             READY   STATUS        RESTARTS   AGE
pi-proxy-sklbm   1/1     Terminating   0          2m34s
```

I first ran the above (starting with `delete the DaemonSet`...) all at once due to copy/paste issue, so I tried the following to success:
```
paulkaefer ~/GitHub/kiamol/ch06 λ kubectl apply -f pi/proxy/daemonset/nginx-ds-nodeSelector.yaml
daemonset.apps/pi-proxy created

Tue Oct 31 10:59:54
~/GitHub/kiamol/ch06
paulkaefer ~/GitHub/kiamol/ch06 λ kubectl delete ds pi-proxy --cascade=orphan
daemonset.apps "pi-proxy" deleted
```

# Section 6.4: Understanding object ownership in Kubernetes
```
# check which objects own the Pods:
kubectl get po -o custom-columns=NAME:'{.metadata.name}',OWNER:'{.metadata.ownerReferences[0].name}',OWNER_KIND:'{.metadata.ownerReferences[0].kind}'
 
# check which objects own the ReplicaSets:
kubectl get rs -o custom-columns=NAME:'{.metadata.name}',OWNER:'{.metadata.ownerReferences[0].name}',OWNER_KIND:'{.metadata.ownerReferences[0].kind}'
```
Output:
```
NAME                     OWNER              OWNER_KIND
pi-proxy-7wcc6           <none>             <none>
pi-web-55b6cb574-4cqqc   pi-web-55b6cb574   ReplicaSet
pi-web-55b6cb574-kzngl   pi-web-55b6cb574   ReplicaSet
pi-web-55b6cb574-xx7bh   pi-web-55b6cb574   ReplicaSet
sleep-8648c6f777-k7mp7   sleep-8648c6f777   ReplicaSet
whoami-web-59586         whoami-web         ReplicaSet
whoami-web-8qbbf         whoami-web         ReplicaSet
whoami-web-d47qj         whoami-web         ReplicaSet

NAME                OWNER    OWNER_KIND
pi-web-55b6cb574    pi-web   Deployment
pi-web-658956d56f   pi-web   Deployment
sleep-8648c6f777    sleep    Deployment
whoami-web          <none>   <none>
```

# 6.4 Understanding object ownership in Kubernetes
```
# remove all the controllers and Services:
kubectl delete all -l kiamol=ch06
```

```
service "pi-proxy" deleted
service "pi-web" deleted
service "whoami-web" deleted
deployment.apps "pi-web" deleted
deployment.apps "sleep" deleted
replicaset.apps "whoami-web" deleted
```


# Ch06 lab

Run the app:

```
kubectl apply -f lab/numbers/
```

```
service/numbers-api created
pod/numbers-api created
service/numbers-web created
replicationcontroller/numbers-web created
```

Get the URL to browse to:

```
kubectl get svc numbers-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8086'
```
http://localhost:8086

> Browse and try to get a random number, the app fails.
```
This page isn’t workingIf the problem continues, contact the site owner.
HTTP ERROR 400
```

### Sample Solution

Add the RNG label to a node:

```
kubectl label node $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') rng=hw
```

```
node/docker-desktop labeled
```

Deploy the new resources - a [Deployment](solution/web-deployment.yaml) for the web app and a [DaemonSet](solution/api-daemonset.yaml) for the API:

```
kubectl apply -f lab/solution/
```

```
daemonset.apps/numbers-api created
deployment.apps/numbers-web created
```

> Refresh the browser and confirm you can get a random number.
```
KIAMOL Random Number Generator
Here it is: 33

(Using API at: http://numbers-api/sixeyed/kiamol/master/ch03/numbers/rng)
```
Tried again... 36, 27.

Delete the old resources by their labels:

```
kubectl get all -l kiamol=ch06-lab

kubectl delete all -l kiamol=ch06-lab
```
Output:
```
NAME              READY   STATUS    RESTARTS   AGE
pod/numbers-api   1/1     Running   0          89s

NAME                                DESIRED   CURRENT   READY   AGE
replicationcontroller/numbers-web   2         2         2       89s

NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/numbers-api   ClusterIP      10.96.96.74     <none>        80/TCP           89s
service/numbers-web   LoadBalancer   10.111.72.120   localhost     8086:31843/TCP   89s

NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/numbers-api   1         1         1       1            1           rng=hw          51s

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/numbers-web   2/2     2            2           51s


pod "numbers-api" deleted
replicationcontroller "numbers-web" deleted
service "numbers-api" deleted
service "numbers-web" deleted
daemonset.apps "numbers-api" deleted
deployment.apps "numbers-web" deleted
```


# Chapter 7: Extending applications with multicontainer Pods

## Section 7.1: How containers communicate in a Pod

### switch to the chapter folder:
`cd ch07`

### deploy the Pod spec:
`kubectl apply -f sleep/sleep-with-file-reader.yaml`
```
deployment.apps/sleep created
```

### get the detailed Pod information:
`kubectl get pod -l app=sleep -o wide`
```
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE             NOMINATED NODE   READINESS GATES
sleep-79d7fffc9b-qtvvc   2/2     Running   0          8s    10.1.0.166   docker-desktop   <none>           <none>
```

### show the container names:
`kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[*].name}'`
```
file-reader sleep
```

### check the Pod logs--this will fail:
`kubectl logs -l app=sleep`
```
Defaulted container "sleep" out of: sleep, file-reader
```
Not the same as the book's output... maybe a later version changed things?


### write a file to the shared volume using one container:
`kubectl exec deploy/sleep -c sleep -- sh -c 'echo ${HOSTNAME} > /data-rw/hostname.txt'`

### read the file using the same container:
`kubectl exec deploy/sleep -c sleep -- cat /data-rw/hostname.txt`
```
sleep-79d7fffc9b-qtvvc
```

### read the file using the other container:
`kubectl exec deploy/sleep -c file-reader -- cat /data-ro/hostname.txt`
```
sleep-79d7fffc9b-qtvvc
```

### try to add to the file to the read-only container--this will fail:
`kubectl exec deploy/sleep -c file-reader -- sh -c 'echo more >> /data-ro/hostname.txt'`
```
sh: can't create /data-ro/hostname.txt: Read-only file system
command terminated with exit code 1
```


### deploy the update:
`kubectl apply -f sleep/sleep-with-server.yaml`
```
deployment.apps/sleep configured
```

### check the Pod status:
`kubectl get pods -l app=sleep`
```
NAME                     READY   STATUS    RESTARTS   AGE
sleep-58f748595f-qvcpz   2/2     Running   0          11s
```

### list the container names in the new Pod:
`kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[*].name}'`
```
server sleep
```

### make a network call between the containers:
`kubectl exec deploy/sleep -c sleep -- wget -q -O - localhost:8080`
```
kiamol
```
Out of curiosity, I tried `kubectl exec deploy/sleep -c sleep -- wget -q -O - paulkaefer.com/ip/index.php` & it returned the same IP as my local machine.
 
### check the server container logs:
`kubectl logs -l app=sleep -c server`
```
GET / HTTP/1.1
Host: localhost:8080
User-Agent: Wget
Connection: close
```

### create a Service targeting the server container port:
`kubectl expose -f sleep/sleep-with-server.yaml --type LoadBalancer --port 8020 --target-port 8080`
```
service/sleep exposed
```

### get the URL for your service:
`kubectl get svc sleep -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8020'`
```
http://localhost:8020
```

### open the URL in your browser
Just says `kiamol`.

### check the server container logs:
`kubectl logs -l app=sleep -c server`
```
sec-ch-ua-platform: "macOS"
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: image
Referer: http://localhost:8020/
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Cookie: .AspNetCore.Antiforgery.9TtSrW0hzOs=CfDJ8HUQ90OKa3NOhBcatrOFhtWBiWNdJyC3aW-tWuALIJWtpQoc1SjuL_e6qT3MfFeGa0N3vysf-2IV9P9vtb3Yn_udCOnVtWgMYQwD2tv8oWS8JXooyphloxoxLBJCZxtApyyC_GXfk-Vggu7S_kYsls8
```

### apply the updated spec with the init container:
`kubectl apply -f sleep/sleep-with-html-server.yaml`
```
deployment.apps/sleep configured
```

### check the Pod containers:
`kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.containerStatuses[*].name}'`
```
server sleep
```

### check the init containers:
`kubectl get pod -l app=sleep -o jsonpath='{.items[0].status.initContainerStatuses[*].name}'`
```
init-html
```

### check logs from the init container--there are none:
`kubectl logs -l app=sleep -c init-html`
No output.

### check that the file is available in the sidecar:
`kubectl exec deploy/sleep -c server -- ls -l /data-ro`
```
total 4
-rw-r--r--    1 root     root            62 Nov  2 15:27 index.html
```
I then ran:
```html
λ kubectl exec deploy/sleep -c server -- cat /data-ro/index.html
<!DOCTYPE html><html><body><h1>KIAMOL Ch07</h1></body></html>
```
...and http://localhost:8020/ shows **KIAMOL Ch07**.


### run the app, which uses a single config file:
`kubectl apply -f timecheck/timecheck.yaml`
```
deployment.apps/timecheck created
```

### check the container logs--there won’t be any:
`kubectl logs -l app=timecheck`
Nothing.

### check the log file inside the container:
`kubectl exec deploy/timecheck -- cat /logs/timecheck.log`
```
2023-11-02 15:33:04.615 +00:00 [INF] Environment: DEV; version: 1.0; time check: 15:33.04
2023-11-02 15:33:09.596 +00:00 [INF] Environment: DEV; version: 1.0; time check: 15:33.09
```

### check the config setup:
`kubectl exec deploy/timecheck -- cat /config/appsettings.json`
```
{
  "Application": {
    "Version": "1.0",
    "Environment": "DEV"
  },
  "Timer": {
    "IntervalSeconds": "5"
  },
  "Metrics": {
    "Enabled": false,
    "Port" : 8080
  }
}
```


### apply the ConfigMap and the new Deployment spec:
`kubectl apply -f timecheck/timecheck-configMap.yaml -f timecheck/timecheck-with-config.yaml`
```
configmap/timecheck-config created
deployment.apps/timecheck configured
```

### wait for the containers to start:
`kubectl wait --for=condition=ContainersReady pod -l app=timecheck,version=v2`
```
pod/timecheck-68696c9766-hdmlm condition met
```

### check the log file in the new app container:
`kubectl exec deploy/timecheck -- cat /logs/timecheck.log`
```
Defaulted container "timecheck" out of: timecheck, init-config (init)
2023-11-02 15:57:41.532 +00:00 [INF] Environment: TEST; version: 1.1; time check: 15:57.41
2023-11-02 15:57:48.519 +00:00 [INF] Environment: TEST; version: 1.1; time check: 15:57.48
```

### see the config file built by the init container:
`kubectl exec deploy/timecheck -- cat /config/appsettings.json`
```
Defaulted container "timecheck" out of: timecheck, init-config (init)
{
  "Application": {
    "Version": "1.1",
    "Environment": "TEST"
  },
  "Timer": {
    "IntervalSeconds": "7"
  }
}
```

### add the sidecar logging container:
`kubectl apply -f timecheck/timecheck-with-logging.yaml`
```
deployment.apps/timecheck configured
```

### wait for the containers to start:
`kubectl wait --for=condition=ContainersReady pod -l app=timecheck,version=v3`
```
pod/timecheck-69f5b9ddf6-fmtjz condition met
```

### check the Pods:
`kubectl get pods -l app=timecheck`
```
NAME                         READY   STATUS    RESTARTS   AGE
timecheck-69f5b9ddf6-fmtjz   2/2     Running   0          16s
```

### check the containers in the Pod:
`kubectl get pod -l app=timecheck -o jsonpath='{.items[0].status.containerStatuses[*].name}'`
```
logger timecheck
```

### now you can see the app logs in the Pod:
`kubectl logs -l app=timecheck -c logger`
```
2023-11-02 16:05:59.781 +00:00 [INF] Environment: TEST; version: 1.1; time check: 16:05.59
2023-11-02 16:06:06.770 +00:00 [INF] Environment: TEST; version: 1.1; time check: 16:06.06
2023-11-02 16:06:13.767 +00:00 [INF] Environment: TEST; version: 1.1; time check: 16:06.13
2023-11-02 16:06:20.768 +00:00 [INF] Environment: TEST; version: 1.1; time check: 16:06.20
2023-11-02 16:06:27.769 +00:00 [INF] Environment: TEST; version: 1.1; time check: 16:06.27
2023-11-02 16:06:34.765 +00:00 [INF] Environment: TEST; version: 1.1; time check: 16:06.34
2023-11-02 16:06:41.765 +00:00 [INF] Environment: TEST; version: 1.1; time check: 16:06.41
```


### apply the update:
`kubectl apply -f timecheck/timecheck-good-citizen.yaml`
```
deployment.apps/timecheck configured
```

### wait for all the containers to be ready:
`kubectl wait --for=condition=ContainersReady pod -l app=timecheck,version=v4`
```
pod/timecheck-9b5654bdd-552rn condition met
```

### check the running containers:
`kubectl get pod -l app=timecheck -o jsonpath='{.items[0].status.containerStatuses[*].name}'`
```
logger timecheck
```

### use the sleep container to check the timecheck app health:
`kubectl exec deploy/sleep -c sleep -- wget -q -O - http://timecheck:8080`
```
{"status": "OK"}
```

### check its metrics:
`kubectl exec deploy/sleep -c sleep -- wget -q -O - http://timecheck:8081`
```
# HELP timechecks_total The total number timechecks.
# TYPE timechecks_total counter
timechecks_total 6
```

# Section 7.4: Abstracting connections with ambassador containers

### deploy the app and Services:
`kubectl apply -f numbers/`
```
service/numbers-api created
deployment.apps/numbers-api created
service/numbers-web created
deployment.apps/numbers-web created
```

### find the URL for your app:
`kubectl get svc numbers-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8090'`
```
http://localhost:8090
```

### browse and get yourself a nice random number
Here it is: 44

### check that the web app has access to other endpoints:
`kubectl exec deploy/numbers-web -c web -- wget -q -O - http://timecheck:8080`
```
{"status": "OK"}
```
Ooh, I see. The numbers app is accesing the timecheck app.


### apply the update from listing 7.5:
`kubectl apply -f numbers/update/web-with-proxy.yaml`
```
deployment.apps/numbers-web configured
```

### refresh your browser, and get a new number
Loading SUPER slowly... Eventually timed out with:
```
KIAMOL Random Number Generator
RNG service unavailable!

(Using API at: http://localhost/api)
```

### check the proxy container logs:
`kubectl logs -l app=numbers-web -c proxy`
```
** Logging proxy listening on port: 1080 **
```

### try to read the health of the timecheck app:
`kubectl exec deploy/numbers-web -c web -- wget -q -O - http://timecheck:8080`
Stalled for over a minute. Killed it.

### check proxy logs again:
`kubectl logs -l app=numbers-web -c proxy`
```
** Logging proxy listening on port: 1080 **
```

# Section 7.5: Understanding the Pod environment

### apply the update:
`kubectl apply -f numbers/update/web-v2-broken-init-container.yaml`
```
deployment.apps/numbers-web configured
```

### check the new Pod:
`kubectl get po -l app=numbers-web,version=v2`
```
NAME                          READY   STATUS                  RESTARTS     AGE
numbers-web-676948668-ls6kl   0/2     Init:CrashLoopBackOff   1 (5s ago)   8s

NAME                          READY   STATUS       RESTARTS      AGE
numbers-web-676948668-ls6kl   0/2     Init:Error   2 (13s ago)   16s

NAME                          READY   STATUS                  RESTARTS      AGE
numbers-web-676948668-ls6kl   0/2     Init:CrashLoopBackOff   2 (14s ago)   29s
```

### check the logs for the new init container:
`kubectl logs -l app=numbers-web,version=v2 -c init-version`
```
sh: can't create /config-out/version.txt: Read-only file system
```

### check the status of the Deployment:
`kubectl get deploy numbers-web`
```
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
numbers-web   1/1     1            1           19m
```

### check the status of the ReplicaSets:
`kubectl get rs -l app=numbers-web`
```
NAME                     DESIRED   CURRENT   READY   AGE
numbers-web-676948668    1         1         0       78s
numbers-web-6bc7f44999   1         1         1       10m
numbers-web-865c56b9d    0         0         0       19m
```


### check the processes in the current container:
`kubectl exec deploy/sleep -c sleep -- ps`
```
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh -c trap : TERM INT; (while true; do sleep 1000; done) & wait
    7 root      0:00 /bin/sh -c trap : TERM INT; (while true; do sleep 1000; done) & wait
   54 root      0:00 sleep 1000
   55 root      0:00 ps
```

### apply the update:
`kubectl apply -f sleep/sleep-with-server-shared.yaml`
```
deployment.apps/sleep configured
```

### wait for the new containers:
`kubectl wait --for=condition=ContainersReady pod -l app=sleep,version=shared`
```
pod/sleep-7d79bb59bc-65g9m condition met
```

### check the processes again:
`kubectl exec deploy/sleep -c sleep -- ps`
```
PID   USER     TIME  COMMAND
    1 65535     0:00 /pause
    7 root      0:00 /bin/sh -c trap : TERM INT; (while true; do sleep 1000; done) & wait
   12 root      0:00 /bin/sh -c trap : TERM INT; (while true; do sleep 1000; done) & wait
   13 root      0:00 sleep 1000
   14 root      0:00 sh -c while true; do echo -e 'HTTP/1.1 200 OK Content-Type: text/plain Content-Length: 7  kiamol' | nc -l -p 8080; done
   20 root      0:00 nc -l -p 8080
   21 root      0:00 ps
```


```bash
λ kubectl delete all -l kiamol=ch07
service "numbers-api" deleted
service "numbers-web" deleted
service "sleep" deleted
service "timecheck" deleted
deployment.apps "numbers-api" deleted
deployment.apps "numbers-web" deleted
deployment.apps "sleep" deleted
deployment.apps "timecheck" deleted
```

# Chapter 7 lab:
First, I compared the starter file to the solution.
```
λ kubectl apply -f lab/pi/
service/pi-web created
deployment.apps/pi-web created

λ kubectl describe pod -l app=pi-web
Name:             pi-web-7c7589fcc7-mvqv9
Namespace:        default
Priority:         0
Service Account:  default
Node:             docker-desktop/192.168.65.3
Start Time:       Thu, 02 Nov 2023 12:35:44 -0500
Labels:           app=pi-web
                  pod-template-hash=7c7589fcc7
Annotations:      <none>
Status:           Running
IP:               10.1.0.178
IPs:
  IP:           10.1.0.178
Controlled By:  ReplicaSet/pi-web-7c7589fcc7
Containers:
  web:
    Container ID:  docker://e3b0727e788154a303457dddbb7d4f3465d2a962f418d043a4cdf69cefd5855a
    Image:         kiamol/ch05-pi
    Image ID:      docker-pullable://kiamol/ch05-pi@sha256:d9346358978c9cecaa1c83c6dca783f460eddc49bbaa6aa914d9c8dcd37689c4
    Port:          80/TCP
    Host Port:     0/TCP
    Command:
      /scripts/startup.sh
    State:          Waiting
      Reason:       RunContainerError
    Last State:     Terminated
      Reason:       ContainerCannotRun
      Message:      failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "/scripts/startup.sh": stat /scripts/startup.sh: no such file or directory: unknown
      Exit Code:    127
      Started:      Thu, 02 Nov 2023 12:35:46 -0500
      Finished:     Thu, 02 Nov 2023 12:35:46 -0500
    Ready:          False
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-6pj9p (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  kube-api-access-6pj9p:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  23s               default-scheduler  Successfully assigned default/pi-web-7c7589fcc7-mvqv9 to docker-desktop
  Normal   Pulled     22s               kubelet            Successfully pulled image "kiamol/ch05-pi" in 663.533542ms (663.945292ms including waiting)
  Normal   Pulled     21s               kubelet            Successfully pulled image "kiamol/ch05-pi" in 452.038792ms (494.799583ms including waiting)
  Normal   Pulling    8s (x3 over 23s)  kubelet            Pulling image "kiamol/ch05-pi"
  Normal   Pulled     8s                kubelet            Successfully pulled image "kiamol/ch05-pi" in 515.585542ms (515.602708ms including waiting)
  Normal   Created    7s (x3 over 22s)  kubelet            Created container web
  Warning  Failed     7s (x3 over 22s)  kubelet            Error: failed to start container "web": Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "/scripts/startup.sh": stat /scripts/startup.sh: no such file or directory: unknown


Name:             pi-web-7c7589fcc7-srmm8
Namespace:        default
Priority:         0
Service Account:  default
Node:             docker-desktop/192.168.65.3
Start Time:       Thu, 02 Nov 2023 12:35:44 -0500
Labels:           app=pi-web
                  pod-template-hash=7c7589fcc7
Annotations:      <none>
Status:           Running
IP:               10.1.0.179
IPs:
  IP:           10.1.0.179
Controlled By:  ReplicaSet/pi-web-7c7589fcc7
Containers:
  web:
    Container ID:  docker://4eda6dc98143bbafdc3d0ef6b781315f2620e5f41b95dd76a6e452bb6f8e292b
    Image:         kiamol/ch05-pi
    Image ID:      docker-pullable://kiamol/ch05-pi@sha256:d9346358978c9cecaa1c83c6dca783f460eddc49bbaa6aa914d9c8dcd37689c4
    Port:          80/TCP
    Host Port:     0/TCP
    Command:
      /scripts/startup.sh
    State:          Waiting
      Reason:       RunContainerError
    Last State:     Terminated
      Reason:       ContainerCannotRun
      Message:      failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "/scripts/startup.sh": stat /scripts/startup.sh: no such file or directory: unknown
      Exit Code:    127
      Started:      Thu, 02 Nov 2023 12:35:47 -0500
      Finished:     Thu, 02 Nov 2023 12:35:47 -0500
    Ready:          False
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2zp4q (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  kube-api-access-2zp4q:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  23s               default-scheduler  Successfully assigned default/pi-web-7c7589fcc7-srmm8 to docker-desktop
  Normal   Pulled     22s               kubelet            Successfully pulled image "kiamol/ch05-pi" in 461.340625ms (1.125120584s including waiting)
  Normal   Pulled     20s               kubelet            Successfully pulled image "kiamol/ch05-pi" in 490.414875ms (490.434001ms including waiting)
  Normal   Pulling    6s (x3 over 23s)  kubelet            Pulling image "kiamol/ch05-pi"
  Normal   Pulled     6s                kubelet            Successfully pulled image "kiamol/ch05-pi" in 518.102042ms (518.120125ms including waiting)
  Normal   Created    5s (x3 over 21s)  kubelet            Created container web
  Warning  Failed     5s (x3 over 21s)  kubelet            Error: failed to start container "web": Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "/scripts/startup.sh": stat /scripts/startup.sh: no such file or directory: unknown

λ kubectl apply -f lab/solution/
service/pi-web-version created
deployment.apps/pi-web configured


```

Now `http://localhost:8071/` shows `ch07-lab`; `http://localhost:8070/?dp=45`, e.g., returns:
```
in: 0 ms.
from: pi-web-68b64b6b97-j4ctq
3.141592653589793238462643383279502884197169399

To: 2,520 d.p.
in: 16 ms.
from: pi-web-68b64b6b97-j4ctq
3.141592653589793238462643383279502884197169399375105820974944592307816406286208998628034825342117067982148086513282306647093844609550582231725359408128481117450284102701938521105559644622948954930381964428810975665933446128475648233786783165271201909145648566923460348610454326648213393607260249141273724587006606315588174881520920962829254091715364367892590360011330530548820466521384146951941511609433057270365759591953092186117381932611793105118548074462379962749567351885752724891227938183011949129833673362440656643086021394946395224737190702179860943702770539217176293176752384674818467669405132000568127145263560827785771342757789609173637178721468440901224953430146549585371050792279689258923542019956112129021960864034418159813629774771309960518707211349999998372978049951059731732816096318595024459455346908302642522308253344685035261931188171010003137838752886587533208381420617177669147303598253490428755468731159562863882353787593751957781857780532171226806613001927876611195909216420198938095257201065485863278865936153381827968230301952035301852968995773622599413891249721775283479131515574857242454150695950829533116861727855889075098381754637464939319255060400927701671139009848824012858361603563707660104710181942955596198946767837449448255379774726847104047534646208046684259069491293313677028989152104752162056966024058038150193511253382430035587640247496473263914199272604269922796782354781636009341721641219924586315030286182974555706749838505494588586926995690927210797509302955321165344987202755960236480665499119881834797753566369807426542527862551818417574672890977772793800081647060016145249192173217214772350141441973568548161361157352552133475741849468438523323907394143334547762416862518983569485562099219222184272550254256887671790494601653466804988627232791786085784383827967976681454100953883786360950680064225125205117392984896084128488626945604241965285022210661186306744278622039194945047123713786960956364371917287467764657573962413890865832645995813390478027590099465764078951269468398352595709825822620522489407726719478268482601476990902640136394437455305068203496252451749399651431429809190659250937221696461515709858387410597885959772975498930161753928468138268683868942774155991855925245953959431049972524680845987273644695848653836736222626099124608051243884390451244136549762780797715691435997700129616089441694868555848406353422072225828488648158456028506016842739452267467678895252138522549954666727823986456596116354886230577456498035593634568174324112515076069479451096596
```

# Chapter 8: Running data-heavy apps with StatefulSets and Jobs

## Section 8.1: How Kubernetes models stability with StatefulSets

### switch to the chapter's source:
`cd ch08`

### deploy the StatefulSet, Service, and a Secret for the Postgres password:
`kubectl apply -f todo-list/db/`
First got:
```
The connection to the server kubernetes.docker.internal:6443 was refused - did you specify the right host or port?
```
...so I launched Docker Desktop. Take two:
```
secret/todo-db-secret created
service/todo-db created
statefulset.apps/todo-db created
```

### check the StatefulSet:
`kubectl get statefulset todo-db`
```
NAME      READY   AGE
todo-db   2/2     12s
```

### check the Pods:
`kubectl get pods -l app=todo-db`
```
NAME        READY   STATUS    RESTARTS   AGE
todo-db-0   1/1     Running   0          25s
todo-db-1   1/1     Running   0          23s
```

### find the hostname of Pod 0:
`kubectl exec pod/todo-db-0 -- hostname`
```
todo-db-0
```

### check the logs of Pod 1:
`kubectl logs todo-db-1 --tail 1`
```
2023-11-08 15:11:27.252 UTC [1] LOG:  database system is ready to accept connections
```

### check the internal ID of Pod 0:
`kubectl get pod todo-db-0 -o jsonpath='{.metadata.uid}'`
```
7c894ba2-8358-4229-ae73-b5951987079b
```

### delete the Pod:
`kubectl delete pod todo-db-0`
```
pod "todo-db-0" deleted
```

### check Pods:
`kubectl get pods -l app=todo-db`
```
NAME        READY   STATUS    RESTARTS   AGE
todo-db-0   1/1     Running   0          12s
todo-db-1   1/1     Running   0          8m38s
```

### check that the new Pod is a new Pod:
`kubectl get pod todo-db-0 -o jsonpath='{.metadata.uid}'`
```
f826be42-5f30-4a19-89f7-fc450bfd7887
```

## Section 8.2: Bootstrapping Pods with init containers in StatefulSets


### show the Service details:
`kubectl get svc todo-db`
```
NAME      TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
todo-db   ClusterIP   None         <none>        5432/TCP   158m
```

### run a sleep Pod to use for network lookups:
`kubectl apply -f sleep/sleep.yaml`
```
deployment.apps/sleep created
```

### run a DNS query for the Service name:
`kubectl exec deploy/sleep -- sh -c 'nslookup todo-db | grep "^[^*]"'`
First try (maybe because this series of commands I accidentally pasted all at once):
```
error: unable to upgrade connection: container not found ("sleep")
```
Second try:
```
Server:   10.96.0.10
Address:  10.96.0.10:53
Name: todo-db.default.svc.cluster.local
Address: 10.1.0.189
Name: todo-db.default.svc.cluster.local
Address: 10.1.0.190
```

### run a DNS lookup for Pod 0:
`kubectl exec deploy/sleep -- sh -c 'nslookup todo-db-0.todo-db.default.svc.cluster.local | grep "^[^*]"'`
```
error: unable to upgrade connection: container not found ("sleep")
```
Take 2:
```
Server:   10.96.0.10
Address:  10.96.0.10:53
Name: todo-db-0.todo-db.default.svc.cluster.local
Address: 10.1.0.190
```

### deploy the replicated StatefulSet setup:
`kubectl apply -f todo-list/db/replicated/`
```
configmap/todo-db-config created
configmap/todo-db-env created
configmap/todo-db-scripts created
statefulset.apps/todo-db configured
```

### wait for the Pods to spin up
`kubectl wait --for=condition=Ready pod -l app=todo-db`
```
pod/todo-db-0 condition met
pod/todo-db-1 condition met
```

### check the logs of Pod 0--the primary:
`kubectl logs todo-db-0 --tail 1`
```
Defaulted container "db" out of: db, wait-service (init), initialize-replication (init)
2023-11-08 17:58:12.129 UTC [6] LOG:  database system is ready to accept connections
```

### and of Pod 1--the secondary:
`kubectl logs todo-db-1 --tail 2`
```
Defaulted container "db" out of: db, wait-service (init), initialize-replication (init)
2023-11-08 17:58:06.974 UTC [48] LOG:  database system was shut down at 2023-11-08 17:58:06 UTC
2023-11-08 17:58:06.976 UTC [10] LOG:  database system is ready to accept connections
```

### add another replica:
`kubectl scale --replicas=3 statefulset/todo-db`
```
statefulset.apps/todo-db scaled
```

### wait for Pod 2 to spin up
`kubectl wait --for=condition=Ready pod -l app=todo-db`
```
pod/todo-db-0 condition met
pod/todo-db-1 condition met
pod/todo-db-2 condition met
```

### check that the new Pod sets itself up as another secondary:
`kubectl logs todo-db-2 --tail 2`
```
Defaulted container "db" out of: db, wait-service (init), initialize-replication (init)
2023-11-08 18:00:49.640 UTC [11] LOG:  database system is ready to accept read only connections
2023-11-08 18:00:49.643 UTC [31] LOG:  started streaming WAL from primary at 0/3000000 on timeline 1
```
OK. Didn't see WAL message above, but this is good. From [here](https://www.postgresql.org/docs/current/wal-intro.html), that's Write-Ahead Logging (WAL).

## Section 8.3: Requesting storage with volume claim templates

"You actually want each Pod to have its own PVC..." <-- but how does that work if we want all pods updating the same datasource?
(I do think cloud data storage is one option...)


### deploy the StatefulSet with volume claim templates:
`kubectl apply -f sleep/sleep-with-pvc.yaml`
```
service/sleep-with-pvc created
statefulset.apps/sleep-with-pvc created
```

### check that the PVCs are created:
`kubectl get pvc`
```
NAME                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-sleep-with-pvc-0   Bound    pvc-911dcf83-bb53-43a8-a44f-41fad562fc40   5Mi        RWO            hostpath       25s
data-sleep-with-pvc-1   Bound    pvc-be418a1e-32f0-4d29-870d-f8ee511096f3   5Mi        RWO            hostpath       21s
```

### write some data to the PVC mount in Pod 0:
`kubectl exec sleep-with-pvc-0 -- sh -c 'echo Pod 0 > /data/pod.txt'`
No output.

### confirm Pod 0 can read the data:
`kubectl exec sleep-with-pvc-0 -- cat /data/pod.txt`
```
Pod 0
```

### confirm Pod 1 can’t--this will fail:
`kubectl exec sleep-with-pvc-1 -- cat /data/pod.txt`
```
cat: can't open '/data/pod.txt': No such file or directory
command terminated with exit code 1
```


### delete the Pod:
`kubectl delete pod sleep-with-pvc-0`
```
pod "sleep-with-pvc-0" deleted
```

### check that the replacement gets created:
`kubectl get pods -l app=sleep-with-pvc`
```
NAME               READY   STATUS    RESTARTS   AGE
sleep-with-pvc-0   1/1     Running   0          11s
sleep-with-pvc-1   1/1     Running   0          104s
```

### check that the new Pod 0 can see the old data:
`kubectl exec sleep-with-pvc-0 -- cat /data/pod.txt`
```
Pod 0
```


### apply the update with volume claim templates--this will fail:
`kubectl apply -f todo-list/db/replicated/update/todo-db-pvc.yaml`
```
The StatefulSet "todo-db" is invalid: spec: Forbidden: updates to statefulset spec for fields other than 'replicas', 'ordinals', 'template', 'updateStrategy', 'persistentVolumeClaimRetentionPolicy' and 'minReadySeconds' are forbidden
```

### delete the existing set:
`kubectl delete statefulset todo-db`
```
statefulset.apps "todo-db" deleted
```

### create a new one with volume claims:
`kubectl apply -f todo-list/db/replicated/update/todo-db-pvc.yaml`
```
statefulset.apps/todo-db created
```

### check the volume claims:
`kubectl get pvc -l app=todo-db`
```
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-todo-db-0   Bound    pvc-5abb5279-2d89-4cfb-879e-1bf299ef554d   50Mi       RWO            hostpath       9s
```

### deploy the web app:
`kubectl apply -f todo-list/web/`
```
configmap/todo-web-config created
secret/todo-web-secret created
service/todo-web created
deployment.apps/todo-web created
```
 
### get the URL for the app:
`kubectl get svc todo-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8081/new'`
If you run this before running the above, you get:
```
Error from server (NotFound): services "todo-web" not found
```
The second time, I got:
```
http://localhost:8081/new
```

### browse and add a new item
When I click submit, I get:
```
This page isn’t working
localhost is currently unable to handle this request.
HTTP ERROR 500
````

### switch to read-only mode, using the database secondary:
`kubectl apply -f todo-list/web/update/todo-web-readonly.yaml`
```
deployment.apps/todo-web configured
```

### refresh the app--the /new page is read-only; browse to /list and you'll see your original data
Still can't see any items.

### check that there are no clients using the primary in Pod 0:
`kubectl exec -it todo-db-0 -- sh -c "psql -U postgres -t -c 'SELECT datname, query FROM pg_stat_activity WHERE datid > 0'"`
```
Defaulted container "db" out of: db, wait-service (init), initialize-replication (init)
 todo     | ROLLBACK
 postgres | SELECT datname, query FROM pg_stat_activity WHERE datid > 0
```

### check that the web app really is using the secondary in Pod 1:
`kubectl exec -it todo-db-1 -- sh -c "psql -U postgres -t -c 'SELECT datname, query FROM pg_stat_activity WHERE datid > 0'"`
```
Defaulted container "db" out of: db, wait-service (init), initialize-replication (init)
 postgres | SELECT datname, query FROM pg_stat_activity WHERE datid > 0
```

Trying on my own:
```
kubectl exec -it todo-db-1 -- sh -c "psql -U postgres -t -c 'SELECT * FROM pg_stat_activity WHERE datid > 0'"

Defaulted container "db" out of: db, wait-service (init), initialize-replication (init)
 13117 | postgres |  45 |       10 | postgres | psql             |             |                 |          -1 | 2023-11-08 18:45:13.126841+00 | 2023-11-08 18:45:13.128469+00 | 2023-11-08 18:45:13.128469+00 | 2023-11-08 18:45:13.12847+00 |                 |            | active |             |          572 | SELECT * FROM pg_stat_activity WHERE datid > 0 | client backend

kubectl exec -it todo-db-1 -- sh -c "psql -U postgres -t -c 'SELECT * FROM pg_stat_activity where false'"
Defaulted container "db" out of: db, wait-service (init), initialize-replication (init)
```
Ok, there might be a way to impute data... at this point, it's figuring out the table layout.

## Section 8.4: Running maintenance tasks with Jobs and CronJobs

### deploy the Job:
`kubectl apply -f pi/pi-job.yaml`
```
job.batch/pi-job created
```

### check the logs for the Pod:
`kubectl logs -l job-name=pi-job`
```
3.14159265358979323846264338327950288419716939937510
```

### check the status of the Job:
`kubectl get job pi-job`
```
NAME     COMPLETIONS   DURATION   AGE
pi-job   1/1           4s         21s
```

### deploy the new Job:
`kubectl apply -f pi/pi-job-random.yaml`
```
job.batch/pi-job-random created
```

### check the Pod status:
`kubectl get pods -l job-name=pi-job-random`
```
NAME                  READY   STATUS      RESTARTS   AGE
pi-job-random-h4jrj   0/1     Completed   0          12s
pi-job-random-mpvn4   0/1     Completed   0          12s
pi-job-random-pwghl   0/1     Completed   0          12s
```

Round 2:
```
NAME                  READY   STATUS            RESTARTS   AGE
pi-job-random-m6dqs   0/1     PodInitializing   0          5s
pi-job-random-pbgrs   1/1     Running           0          5s
pi-job-random-v6rrs   0/1     PodInitializing   0          5s
```

### check the Job status:
`kubectl get job pi-job-random`
```
NAME            COMPLETIONS   DURATION   AGE
pi-job-random   3/3           8s         22s
```

### list the Pod output:
`kubectl logs -l job-name=pi-job-random`
```
Defaulted container "pi" out of: pi, init-dp (init)
Defaulted container "pi" out of: pi, init-dp (init)
Defaulted container "pi" out of: pi, init-dp (init)
3.14159265358979323846264338327950288419716939937510582097494459230781640628620899862803482534211706798214808651328230664709384460955058223172535940812848111745028410270193852110555964462294895493038196442881097566593344612847564823378678316527120190914564856692346034861045432664821339360726024914127372458700660631558817488152092096282925409171536436789259036001133053054882046652138414695194151160943305727036575959195309218611738193261179310511854807446237996274956735188575272489122793818301194912983367336244065664308602139494639522473719070217986094370277053921717629317675238467481846766940513200056812714526356082778577134275778960917363717872146844090122495343014654958537105079227968925892354201995611212902196086403441815981362977477130996051870721134999999837297804995105973173281609631859502445945534690830264252230825334468503526193118817101000313783875288658753320838142061717766914730359825349042875546873115956286388235378759375195778185778053217122680661300192787661119590921642019893809525720106548586327886593615338182796823030195203530185296899577362259941389124972177528347913151557485724245415069595082953311686172785588907509838175463746493931925506040092770167113900984882401285836160356370766010471018194295559619894676783744944825537977472684710404753464620804668425906949129331367702898915210475216205696602405803815019351125338243003558764024749647326391419927260426992279678235478163600934172164121992458631503028618297455570674983850549458858692699569092721079750930295532116534498720275596023648066549911988183479775356636980742654252786255181841757467289097777279380008164706001614524919217321721477235014144197356854816136115735255213347574184946843852332390739414333454776241686251898356948556209921922218427255025425688767179049460165346680498862723279178608578438382796797668145410095388378636095068006422512520511739298489608412848862694560424196528502221066118630674427862203919494504712371378696095636437191728746776465757396241389086583264599581339047802759009946576407895126946839835259570982582262052248940772671947826848260147699090264013639443745530506820349625245174939965143142980919065925093722169646151570985838741059788595977297549893016175392846813826868386894277415599185592524595395943104997252468084598727364469584865383673622262609912460805124388439045124413654976278079771569143599770012961608944169486855584840635342207222582848864815845602850601684273945226746767889525213852254995466672782398645659611635488623057745649803559363456817432411251507606947945109659609402522887971089314566913686722874894056010150330861792868092087476091782493858900971490967598526136554978189312978482168299894872265880485756401427047755513237964145152374623436454285844479526586782105114135473573952311342716610213596953623144295248493718711014576540359027993440374200731057853906219838744780847848968332144571386875194350643021845319104848100537061468067491927819119793995206141966342875444064374512371819217999839101591956181467514269123974894090718649423196156794520809514655022523160388193014209376213785595663893778708303906979207734672218256259966150142150306803844773454920260541466592520149744285073251866600213243408819071048633173464965145390579626856100550810665879699816357473638405257145910289706414011097120628043903975951567715770042033786993600723055876317635942187312514712053292819182618612586732157919841484882916447060957527069572209175671167229109816909152801735067127485832228718352093539657251210835791513698820914442100675103346711031412671113699086585163983150197016515116851714376576183515565088490998985998238734552833163550764791853589322618548963213293308985706420467525907091548141654985946163718027098199430992448895757128289059232332609729971208443357326548938239119325974636673058360414281388303203824903758985243744170291327656180937734440307074692112019130203303801976211011004492932151608424448596376698389522868478312355265821314495768572624334418930396864262434107732269780280731891544110104468232527162010526522721116603966655730925471105578537634668206531098965269186205647693125705863566201855810072936065987648611791045334885034611365768675324944166803962657978771855608455296541266540853061434443185867697514566140680070023787765913440171274947042056223053899456131407112700040785473326993908145466464588079727082668306343285878569830523580893306575740679545716377525420211495576158140025012622859413021647155097925923099079654737612551765675135751782966645477917450112996148903046399471329621073404375189573596145890193897131117904297828564750320319869151402870808599048010941214722131794764777262241425485454033215718530614228813758504306332175182979866223717215916077166925474873898665494945011465406284336639379003976926567214638530673609657120918076383271664162748888007869256029022847210403172118608204190004229661711963779213375751149595015660496318629472654736425230817703675159067350235072835405670403867435136222247715891504953098444893330963408780769325993978054193414473774418426312986080998886874132604721569516239658645730216315981931951673538129741677294786724229246543668009806769282382806899640048243540370141631496589794092432378969070697794223625082216889573837986230015937764716512289357860158816175578297352334460428151262720373431465319777741603199066554187639792933441952154134189948544473456738316249934191318148092777710386387734317720754565453220777092120190516609628049092636019759882816133231666365286193266863360627356763035447762803504507772355471058595487027908143562401451718062464362679456127531813407833033625423278394497538243720583531147711992606381334677687969597030983391307710987040859133746414428227726346594704745878477872019277152807317679077071572134447306057007334924369311383504931631284042512192565179806941135280131470130478164378851852909285452011658393419656213491434159562586586557055269049652098580338507224264829397285847831630577775606888764462482468579260395352773480304802900587607582510474709164396136267604492562742042083208566119062545433721315359584506877246029016187667952406163425225771954291629919306455377991403734043287526288896399587947572917464263574552540790914513571113694109119393251910760208252026187985318877058429725916778131496990090192116971737278476847268608490033770242429165130050051683233643503895170298939223345172201381280696501178440874519601212285993716231301711444846409038906449544400619869075485160263275052983491874078668088183385102283345085048608250393021332197155184306354550076682829493041377655279397517546139539846833936383047461199665385815384205685338621867252334028308711232827892125077126294632295639898989358211674562701021835646220134967151881909730381198004973407239610368540664319395097901906996395524530054505806855019567302292191393391856803449039820595510022635353619204199474553859381023439554495977837790237421617271117236434354394782218185286240851400666044332588856986705431547069657474585503323233421073015459405165537906866273337995851156257843229882737231989875714159578111963583300594087306812160287649628674460477464915995054973742562690104903778198683593814657412680492564879855614537234786733039046883834363465537949864192705638729317487233208376011230299113679386270894387993620162951541337142489283072201269014754668476535761647737946752004907571555278196536213239264061601363581559074220202031872776052772190055614842555187925303435139844253223415762336106425063904975008656271095359194658975141310348227693062474353632569160781547818115284366795706110861533150445212747392454494542368288606134084148637767009612071512491404302725386076482363414334623518975766452164137679690314950191085759844239198629164219399490723623464684411739403265918404437805133389452574239950829659122850855582157250310712570126683024029295252201187267675622041542051618416348475651699981161410100299607838690929160302884002691041407928862150784245167090870006992821206604183718065355672525325675328612910424877618258297651579598470356222629348600341587229805349896502262917487882027342092222453398562647669149055628425039127577102840279980663658254889264880254566101729670266407655904290994568150652653053718294127033693137851786090407086671149655834343476933857817113864558736781230145876871266034891390956200993936103102916161528813843790990423174733639480457593149314052976347574811935670911013775172100803155902485309066920376719220332290943346768514221447737939375170344366199104033751117354719185504644902636551281622882446257591633303910722538374218214088350865739177150968288747826569959957449066175834413752239709683408005355984917541738188399944697486762655165827658483588453142775687900290951702835297163445621296404352311760066510124120065975585127617858382920419748442360800719304576189323492292796501987518721272675079812554709589045563579212210333466974992356302549478024901141952123828153091140790738602515227429958180724716259166854513331239480494707911915326734302824418604142636395480004480026704962482017928964766975831832713142517029692348896276684403232609275249603579964692565049368183609003238092934595889706953653494060340216654437558900456328822505452556405644824651518754711962184439658253375438856909411303150952617937800297412076651479394259029896959469955657612186561967337862362561252163208628692221032748892186543648022967807057656151446320469279068212073883778142335628236089632080682224680122482611771858963814091839036736722208883215137556003727983940041529700287830766709444745601345564172543709069793961225714298946715435784687886144458123145935719849225284716050492212424701412147805734551050080190869960330276347870810817545011930714122339086639383395294257869050764310063835198343893415961318543475464955697810382930971646514384070070736041123735998434522516105070270562352660127648483084076118301305279320542746286540360367453286510570658748822569815793678976697422057505968344086973502014102067235850200724522563265134105592401902742162484391403599895353945909440704691209140938700126456001623742880210927645793106579229552498872758461012648369998922569596881592056001016552563756785667227966198857827948488558343975187445455129656344348039664205579829368043522027709842942325330225763418070394769941597915945300697521482933665556615678736400536665641654732170439035213295435291694145990416087532018683793702348886894791510716378529023452924407736594956305100742108714261349745956151384987137570471017879573104229690666702144986374645952808243694457897723300487647652413390759204340196340391147320233807150952220106825634274716460243354400515212669324934196739770415956837535551667302739007497297363549645332888698440611964961627734495182736955882207573551766515898551909866653935494810688732068599075407923424023009259007017319603622547564789406475483466477604114632339056513433068449539790709030234604614709616968868850140834704054607429586991382966824681857103188790652870366508324319744047718556789348230894310682870272280973624809399627060747264553992539944280811373694338872940630792615959954626246297070625948455690347119729964090894180595343932512362355081349490043642785271383159125689892951964272875739469142725343669415323610045373048819855170659412173524625895487301676002988659257866285612496655235338294287854253404830833070165372285635591525347844598183134112900199920598135220511733658564078264849427644113763938669248031183644536985891754426473998822846218449008777697763127957226726555625962825427653183001340709223343657791601280931794017185985999338492354956400570995585611349802524990669842330173503580440811685526531170995708994273287092584878944364600504108922669178352587078595129834417295351953788553457374260859029081765155780390594640873506123226112009373108048548526357228257682034160504846627750450031262008007998049254853469414697751649327095049346393824322271885159740547021482897111777923761225788734771881968254629812686858170507402725502633290449762778944236216741191862694396506715157795867564823993917604260176338704549901761436412046921823707648878341968968611815581587360629386038101712158552726683008238340465647588040513808016336388742163714064354955618689641122821407533026551004241048967835285882902436709048871181909094945331442182876618103100735477054981596807720094746961343609286148494178501718077930681085469000944589952794243981392135055864221964834915126390128038320010977386806628779239718014613432445726400973742570073592100315415089367930081699805365202760072774967458400283624053460372634165542590276018348403068113818551059797056640075094260878857357960373245141467867036880988060971642584975951380693094494015154222219432913021739125383559150310033303251117491569691745027149433151558854039221640972291011290355218157628232831823425483261119128009282525619020526301639114772473314857391077758744253876117465786711694147764214411112635835538713610110232679877564102468240322648346417663698066378576813492045302240819727856471983963087815432211669122464159117767322532643356861461865452226812688726844596844241610785401676814208088502800541436131462308210259417375623899420757136275167457318918945628352570441335437585753426986994725470316566139919996826282472706413362221789239031760854289437339356188916512504244040089527198378738648058472689546243882343751788520143956005710481194988423906061369573423155907967034614914344788636041031823507365027785908975782727313050488939890099239135033732508559826558670892426124294736701939077271307068691709264625484232407485503660801360466895118400936686095463250021458529309500009071510582362672932645373821049387249966993394246855164832611341461106802674466373343753407642940266829738652209357016263846485285149036293201991996882851718395366913452224447080459239660281715655156566611135982311225062890585491450971575539002439315351909021071194573002438801766150352708626025378817975194780610137150044899172100222013350131060163915415895780371177927752259787428919179155224171895853616805947412341933984202187456492564434623925319531351033114763949119950728584306583619353693296992898379149419394060857248639688369032655643642166442576079147108699843157337496488352927693282207629472823815374099615455987982598910937171262182830258481123890119682214294576675807186538065064870261338928229949725745303328389638184394477077940228435988341003583854238973542439564755568409522484455413923941000162076936368467764130178196593799715574685419463348937484391297423914336593604100352343777065888677811394986164787471407932638587386247328896456435987746676384794665040741118256583788784548581489629612739984134427260860618724554523606431537101127468097787044640947582803487697589483282412392929605829486191966709189580898332012103184303401284951162035342801441276172858302435598300320420245120728725355811958401491809692533950757784000674655260314461670508276827722235341911026341631571474061238504258459884199076112872580591139356896014316682831763235673254170734208173322304629879928049085140947903688786878949305469557030726190095020764334933591060245450864536289354568629585313153371838682656178622736371697577418302398600659148161640494496501173213138957470620884748023653710311508984279927544268532779743113951435741722197597993596852522857452637962896126915723579866205734083757668738842664059909935050008133754324546359675048442352848747014435454195762584735642161981340734685411176688311865448937769795665172796623267148103386439137518659467300244345005449953997423723287124948347060440634716063258306498297955101095418362350303094530973358344628394763047756450150085075789495489313939448992161255255977014368589435858775263796255970816776438001254365023714127834679261019955852247172201777237004178084194239487254068015560359983905489857235467456423905858502167190313952629445543913166313453089390620467843877850542393905247313620129476918749751910114723152893267725339181466073000890277689631148109022097245207591672970078505807171863810549679731001678708506942070922329080703832634534520380278609905569001341371823683709919495164896007550493412678764367463849020639640197666855923356546391383631857456981471962108410809618846054560390384553437291414465134749407848844237721751543342603066988317683310011331086904219390310801437843341513709243530136776310849135161564226984750743032971674696406665315270353254671126675224605511995818319637637076179919192035795820075956053023462677579439363074630569010801149427141009391369138107258137813578940055995001835425118417213605572752210352680373572652792241737360575112788721819084490061780138897107708229310027976659358387589093956881485602632243937265624727760378908144588378550197028437793624078250527048758164703245812908783952324532378960298416692254896497156069811921865849267704039564812781021799132174163058105545988013004845629976511212415363745150056350701278159267142413421033015661653560247338078430286552572227530499988370153487930080626018096238151613669033411113865385109193673938352293458883225508870645075394739520439680790670868064450969865488016828743437861264538158342807530618454859037982179945996811544197425363443996029025100158882721647450068207041937615845471231834600726293395505482395571372568402322682130124767945226448209102356477527230820810635188991526928891084555711266039650343978962782500161101532351605196559042118449499077899920073294769058685778787209829013529566139788848605097860859570177312981553149516814671769597609942100361835591387778176984587581044662839988060061622984861693533738657877359833616133841338536842119789389001852956919678045544828584837011709672125353387586215823101331038776682721157269495181795897546939926421979155233857662316762754757035469941489290413018638611943919628388705436777432242768091323654494853667680000010652624854730558615989991401707698385483188750142938908995068545307651168033373222651756622075269517914422528081651716677667279303548515420402381746089232839170327542575086765511785939500279338959205766827896776445318404041855401043513483895312013263783692835808271937831265496174599705674507183320650345566440344904536275600112501843356073612227659492783937064784264567633881880756561216896050416113903906396016202215368494109260538768871483798955999911209916464644119185682770045742434340216722764455893301277815868695250694993646101756850601671453543158148010545886056455013320375864548584032402987170934809105562116715468484778039447569798042631809917564228098739987669732376957370158080682290459921236616890259627304306793165311494017647376938735140933618332161428021497633991898354848756252987524238730775595559554651963944018218409984124898262367377146722606163364329640633572810707887581640438148501884114318859882769449011932129682715888413386943468285900666408063140777577257056307294004929403024204984165654797367054855804458657202276378404668233798528271057843197535417950113472736257740802134768260450228515797957976474670228409995616015691089038458245026792659420555039587922981852648007068376504183656209455543461351341525700659748819163413595567196496540321872716026485930490397874895890661272507948282769389535217536218507962977851461884327192232238101587444505286652380225328438913752738458923844225354726530981715784478342158223270206902872323300538621634798850946954720047952311201504329322662827276321779088400878614802214753765781058197022263097174950721272484794781695729614236585957820908307332335603484653187302930266596450137183754288975579714499246540386817992138934692447419850973346267933210726868707680626399193619650440995421676278409146698569257150743157407938053239252394775574415918458215625181921552337096074833292349210345146264374498055961033079941453477845746999921285999993996122816152193148887693880222810830019860165494165426169685867883726095877456761825072759929508931805218729246108676399589161458550583972742098090978172932393010676638682404011130402470073508578287246271349463685318154696904669686939254725194139929146524238577625500474852954768147954670070503479995888676950161249722820403039954632788306959762493615101024365553522306906129493885990157346610237122354789112925476961760050479749280607212680392269110277722610254414922157650450812067717357120271802429681062037765788371669091094180744878140490755178203856539099104775941413215432844062503018027571696508209642734841469572639788425600845312140659358090412711359200419759851362547961606322887361813673732445060792441176399759746193835845749159880976674470930065463424234606342374746660804317012600520559284936959414340814685298150539471789004518357551541252235905906872648786357525419112888773717663748602766063496035367947026923229718683277173932361920077745221262475186983349515101986426988784717193966497690708252174233656627259284406204302141137199227852699846988477023238238400556555178890876613601304770984386116870523105531491625172837327286760072481729876375698163354150746088386636406934704372066886512756882661497307886570156850169186474885416791545965072342877306998537139043002665307839877638503238182155355973235306860430106757608389086270498418885951380910304235957824951439885901131858358406674723702971497850841458530857813391562707603563907639473114554958322669457024941398316343323789759556808568362972538679132750555425244919435891284050452269538121791319145135009938463117740179715122837854601160359554028644059024964669307077690554810288502080858008781157738171917417760173307385547580060560143377432990127286772530431825197579167929699650414607066457125888346979796429316229655201687973000356463045793088403274807718115553309098870255052076804630346086581653948769519600440848206596737947316808641564565053004988161649057883115434548505266006982309315777650037807046612647060214575057932709620478256152471459189652236083966456241051955105223572397395128818164059785914279148165426328920042816091369377737222999833270820829699557377273756676155271139225880552018988762011416800546873655806334716037342917039079863965229613128017826797172898229360702880690877686605932527463784053976918480820410219447197138692560841624511239806201131845412447820501107987607171556831540788654390412108730324020106853419472304766667217498698685470767812051247367924791931508564447753798537997322344561227858432968466475133365736923872014647236794278700425032555899268843495928761240075587569464137056251400117971331662071537154360068764773186755871487839890810742953094106059694431584775397009439883949144323536685392099468796450665339857388878661476294434140104988899316005120767810358861166020296119363968213496075011164983278563531614516845769568710900299976984126326650234771672865737857908574664607722834154031144152941880478254387617707904300015669867767957609099669360755949651527363498118964130433116627747123388174060373174397054067031096767657486953587896700319258662594105105335843846560233917967492678447637084749783336555790073841914731988627135259546251816043422537299628632674968240580602964211463864368642247248872834341704415734824818333016405669596688667695634914163284264149745333499994800026699875888159350735781519588990053951208535103572613736403436753471410483601754648830040784641674521673719048310967671134434948192626811107399482506073949507350316901973185211955263563258433909982249862406703107683184466072912487475403161796994113973877658998685541703188477886759290260700432126661791922352093822787888098863359911608192353555704646349113208591897961327913197564909760001399623444553501434642686046449586247690943470482932941404111465409239883444351591332010773944111840741076849810663472410482393582740194493566516108846312567852977697346843030614624180358529331597345830384554103370109167677637427621021370135485445092630719011473184857492331816720721372793556795284439254815609137281284063330393735624200160456645574145881660521666087387480472433912129558777639069690370788285277538940524607584962315743691711317613478388271941686066257210368513215664780014767523103935786068961112599602818393095487090590738613519145918195102973278755710497290114871718971800469616977700179139196137914171627070189584692143436967629274591099400600849835684252019155937037010110497473394938778859894174330317853487076032219829705797511914405109942358830345463534923498268836240433272674155403016195056806541809394099820206099941402168909007082133072308966211977553066591881411915778362729274615618571037217247100952142369648308641025928874579993223749551912219519034244523075351338068568073544649951272031744871954039761073080602699062580760202927314552520780799141842906388443734996814582733720726639176702011830046481900024130835088465841521489912761065137415394356572113903285749187690944137020905170314877734616528798482353382972601361109845148418238081205409961252745808810994869722161285248974255555160763716750548961730168096138038119143611439921063800508321409876045993093248510251682944672606661381517457125597549535802399831469822036133808284993567055755247129027453977621404931820146580080215665360677655087838043041343105918046068008345911366408348874080057412725867047922583191274157390809143831384564241509408491339180968402511639919368532255573389669537490266209232613188558915808324555719484538756287861288590041060060737465014026278240273469625282171749415823317492396835301361786536737606421667781377399510065895288774276626368418306801908046098498094697636673356622829151323527888061577682781595886691802389403330764419124034120223163685778603572769415417788264352381319050280870185750470463129333537572853866058889045831114507739429352019943219711716422350056440429798920815943071670198574692738486538334361457946341759225738985880016980147574205429958012429581054565108310462972829375841611625325625165724980784920998979906200359365099347215829651741357984910471116607915874369865412223483418877229294463351786538567319625598520260729476740726167671455736498121056777168934849176607717052771876011999081441130586455779105256843048114402619384023224709392498029335507318458903553971330884461741079591625117148648744686112476054286734367090466784686702740918810142497111496578177242793470702166882956108777944050484375284433751088282647719785400065097040330218625561473321177711744133502816088403517814525419643203095760186946490886815452856213469883554445602495566684366029221951248309106053772019802183101032704178386654471812603971906884623708575180800353270471856594994761242481109992886791589690495639476246084240659309486215076903149870206735338483495508363660178487710608098042692471324100094640143736032656451845667924566695510015022983307984960799498824970617236744936122622296179081431141466094123415935930958540791390872083227335495720807571651718765994498569379562387555161757543809178052802946420044721539628074636021132942559160025707356281263873310600589106524570802447493754318414940148211999627645310680066311838237616396631809314446712986155275982014514102756006892975024630401735148919457636078935285550531733141645705049964438909363084387448478396168405184527328840323452024705685164657164771393237755172947951261323982296023945485797545865174587877133181387529598094121742273003522965080891777050682592488223221549380483714547816472139768209633205083056479204820859204754998573203888763916019952409189389455767687497308569559580106595265030362661597506622250840674288982659075106375635699682115109496697445805472886936310203678232501823237084597901115484720876182124778132663304120762165873129708112307581598212486398072124078688781145016558251361789030708608701989758898074566439551574153631931919810705753366337380382721527988493503974800158905194208797113080512339332219034662499171691509485414018710603546037946433790058909577211808044657439628061867178610171567409676620802957665770512912099079443046328929473061595104309022214393718495606340561893425130572682914657832933405246350289291754708725648426003496296116541382300773133272983050016025672401418515204189070115428857992081219844931569990591820118197335001261877280368124819958770702075324063612593134385955425477819611429351635612234966615226147353996740515849986035529533292457523888101362023476246690558164389678630976273655047243486430712184943734853006063876445662721866617012381277156213797461498613287441177145524447089971445228856629424402301847912054784985745216346964489738920624019435183100882834802492490854030778638751659113028739587870981007727182718745290139728366148421428717055317965430765045343246005363614726181809699769334862640774351999286863238350887566835950972655748154319401955768504372480010204137498318722596773871549583997184449072791419658459300839426370208756353982169620553248032122674989114026785285996734052420310917978999057188219493913207534317079800237365909853755202389116434671855829068537118979526262344924833924963424497146568465912489185566295893299090352392333336474352037077010108438800329075983421701855422838616172104176030116459187805393674474720599850235828918336929223373239994804371084196594731626548257480994825099918330069765693671596893644933488647442135008407006608835972350395323401795825570360169369909886711321097988970705172807558551912699306730992507040702455685077867906947661262980822516331363995211709845280926303759224267425755998928927837047444521893632034894155210445972618838003006776179313813991620580627016510244588692476492468919246121253102757313908404700071435613623169923716948481325542009145304103713545329662063921054798243921251725401323149027405858920632175894943454890684639931375709103463327141531622328055229729795380188016285907357295541627886764982741861642187898857410716490691918511628152854867941736389066538857642291583425006736124538491606741373401735727799563410433268835695078149313780073623541800706191802673285511919426760912210359874692411728374931261633950012395992405084543756985079570462226646190001035004901830341535458428337643781119885563187777925372011667185395418359844383052037628194407615941068207169703022851522505731260930468984234331527321313612165828080752126315477306044237747535059522871744026663891488171730864361113890694202790881431194487994171540421034121908470940802540239329429454938786402305129271190975135360009219711054120966831115163287054230284700731206580326264171161659576132723515666625366727189985341998952368848309993027574199164638414270779887088742292770538912271724863220288984251252872178260305009945108247835729056919885554678860794628053712270424665431921452817607414824038278358297193010178883456741678113989547504483393146896307633966572267270433932167454218245570625247972199786685427989779923395790575818906225254735822052364248507834071101449804787266919901864388229323053823185597328697809222535295910173414073348847610055640182423921926950620831838145469839236646136398910121021770959767049083050818547041946643713122996923588953849301363565761861060622287055994233716310212784574464639897381885667462608794820186474876727272220626764653380998019668836809941590757768526398651462533363124505364026105696055131838131742611844201890888531963569869627950367384243130113317533053298020166888174813429886815855778103432317530647849832106297184251843855344276201282345707169885305183261796411785796088881503296022907056144762209150947390359466469162353968092013945781758910889319921122600739281491694816152738427362642980982340632002440244958944561291670495082358124873917996
3.14159265358979323846264338327950288419716939937510582097494459230781640628620899862803482534211706798214808651328230664709384460955058223172535940812848111745028410270193852110555964462294895493038196442881097566593344612847564823378678316527120190914564856692346034861045432664821339360726024914127372458700660631558817488152092096282925409171536436789259036001133053054882046652138414695194151160943305727036575959195309218611738193261179310511854807446237996274956735188575272489122793818301194912983367336244065664308602139494639522473719070217986094370277053921717629317675238467481846766940513200056812714526356082778577134275778960917363717872146844090122495343014654958537105079227968925892354201995611212902196086403441815981362977477130996051870721134999999837297804995105973173281609631859502445945534690830264252230825334468503526193118817101000313783875288658753320838142061717766914730359825349042875546873115956286388235378759375195778185778053217122680661300192787661119590921642019893809525720106548586327886593615338182796823030195203530185296899577362259941389124972177528347913151557485724245415069595082953311686172785588907509838175463746493931925506040092770167113900984882401285836160356370766010471018194295559619894676783744944825537977472684710404753464620804668425906949129331367702898915210475216205696602405803815019351125338243003558764024749647326391419927260426992279678235478163600934172164121992458631503028618297455570674983850549458858692699569092721079750930295532116534498720275596023648066549911988183479775356636980742654252786255181841757467289097777279380008164706001614524919217321721477235014144197356854816136115735255213347574184946843852332390739414333454776241686251898356948556209921922218427255025425688767179049460165346680498862723279178608578438382796797668145410095388378636095068006422512520511739298489608412848862694560424196528502221066118630674427862203919494504712371378696095636437191728746776465757396241389086583264599581339047802759009946576407895126946839835259570982582262052248940772671947826848260147699090264013639443745530506820349625245174939965143142980919065925093722169646151570985838741059788595977297549893016175392846813826868386894277415599185592524595395943104997252468084598727364469584865383673622262609912460805124388439045124413654976278079771569143599770012961608944169486855584840635342207222582848864815845602850601684273945226746767889525213852254995466672782398645659611635488623057745649803559363456817432411251507606947945109659609402522887971089314566913686722874894056010150330861792868092087476091782493858900971490967598526136554978189312978482168299894872265880485756401427047755513237964145152374623436454285844479526586782105114135473573952311342716610213596953623144295248493718711014576540359027993440374200731057853906219838744780847848968332144571386875194350643021845319104848100537061468067491927819119793995206141966342875444064374512371819217999839101591956181467514269123974894090718649423196156794520809514655022523160388193014209376213785595663893778708303906979207734672218256259966150142150306803844773454920260541466592520149744285073251866600213243408819071048633173464965145390579626856100550810665879699816357473638405257145910289706414011097120628043903975951567715770042033786993600723055876317635942187312514712053292819182618612586732157919841484882916447060957527069572209175671167229109816909152801735067127485832228718352093539657251210835791513698820914442100675103346711031412671113699086585163983150197016515116851714376576183515565088490998985998238734552833163550764791853589322618548963213293308985706420467525907091548141654985946163718027098199430992448895757128289059232332609729971208443357326548938239119325974636673058360414281388303203824903758985243744170291327656180937734440307074692112019130203303801976211011004492932151608424448596376698389522868478312355265821314495768572624334418930396864262434107732269780280731891544110104468232527162010526522721116603966655730925471105578537634668206531098965269186205647693125705863566201855810072936065987648611791045334885034611365768675324944166803962657978771855608455296541266540853061434443185867697514566140680070023787765913440171274947042056223053899456131407112700040785473326993908145466464588079727082668306343285878569830523580893306575740679545716377525420211495576158140025012622859413021647155097925923099079654737612551765675135751782966645477917450112996148903046399471329621073404375189573596145890193897131117904297828564750320319869151402870808599048010941214722131794764777262241425485454033215718530614228813758504306332175182979866223717215916077166925474873898665494945011465406284336639379003976926567214638530673609657120918076383271664162748888007869256029022847210403172118608204190004229661711963779213375751149595015660496318629472654736425230817703675159067350235072835405670403867435136222247715891504953098444893330963408780769325993978054193414473774418426312986080998886874132604721569516239658645730216315981931951673538129741677294786724229246543668009806769282382806899640048243540370141631496589794092432378969070697794223625082216889573837986230015937764716512289357860158816175578297352334460428151262720373431465319777741603199066554187639792933441952154134189948544473456738316249934191318148092777710386387734317720754565453220777092120190516609628049092636019759882816133231666365286193266863360627356763035447762803504507772355471058595487027908143562401451718062464362679456127531813407833033625423278394497538243720583531147711992606381334677687969597030983391307710987040859133746414428227726346594704745878477872019277152807317679077071572134447306057007334924369311383504931631284042512192565179806941135280131470130478164378851852909285452011658393419656213491434159562586586557055269049652098580338507224264829397285847831630577775606888764462482468579260395352773480304802900587607582510474709164396136267604492562742042083208566119062545433721315359584506877246029016187667952406163425225771954291629919306455377991403734043287526288896399587947572917464263574552540790914513571113694109119393251910760208252026187985318877058429725916778131496990090192116971737278476847268608490033770242429165130050051683233643503895170298939223345172201381280696501178440874519601212285993716231301711444846409038906449544400619869075485160263275052983491874078668088183385102283345085048608250393021332197155184306354550076682829493041377655279397517546139539846833936383047461199665385815384205685338621867252334028308711232827892125077126294632295639898989358211674562701021835646220134967151881909730381198004973407239610368540664319395097901906996395524530054505806855019567302292191393391856803449039820595510022635353619204199474553859381023439554495977837790237421617271117236434354394782218185286240851400666044332588856986705431547069657474585503323233421073015459405165537906866273337995851156257843229882737231989875714159578111963583300594087306812160287649628674460477464915995054973742562690104903778198683593814657412680492564879855614537234786733039046883834363465537949864192705638729317487233208376011230299113679386270894387993620162951541337142489283072201269014754668476535761647737946752004907571555278196536213239264061601363581559074220202031872776052772190055614842555187925303435139844253223415762336106425063904975008656271095359194658975141310348227693062474353632569160781547818115284366795706110861533150445212747392454494542368288606134084148637767009612071512491404302725386076482363414334623518975766452164137679690314950191085759844239198629164219399490723623464684411739403265918404437805133389452574239950829659122850855582157250310712570126683024029295252201187267675622041542051618416348475651699981161410100299607838690929160302884002691041407928862150784245167090870006992821206604183718065355672525325675328612910424877618258297651579598470356222629348600341587229805349896502262917487882027342092222453398562647669149055628425039127577102840279980663658254889264880254566101729670266407655904290994568150652653053718294127033693137851786090407086671149655834343476933857817113864558736781230145876871266034891390956200993936103102916161528813843790990423174733639480457593149314052976347574811935670911013775172100803155902485309066920376719220332290943346768514221447737939375170344366199104033751117354719185504644902636551281622882446257591633303910722538374218214088350865739177150968288747826569959957449066175834413752239709683408005355984917541738188399944697486762655165827658483588453142775687900290951702835297163445621296404352311760066510124120065975585127617858382920419748442360800719304576189323492292796501987518721272675079812554709589045563579212210333466974992356302549478024901141952123828153091140790738602515227429958180724716259166854513331239480494707911915326734302824418604142636395480004480026704962482017928964766975831832713142517029692348896276684403232609275249603579964692565049368183609003238092934595889706953653494060340216654437558900456328822505452556405644824651518754711962184439658253375438856909411303150952617937800297412076651479394259029896959469955657612186561967337862362561252163208628692221032748892186543648022967807057656151446320469279068212073883778142335628236089632080682224680122482611771858963814091839036736722208883215137556003727983940041529700287830766709444745601345564172543709069793961225714298946715435784687886144458123145935719849225284716050492212424701412147805734551050080190869960330276347870810817545011930714122339086639383395294257869050764310063835198343893415961318543475464955697810382930971646514384070070736041123735998434522516105070270562352660127648483084076118301305279320542746286540360367453286510570658748822569815793678976697422057505968344086973502014102067235850200724522563265134105592401902742162484391403599895353945909440704691209140938700126456001623742880210927645793106579229552498872758461012648369998922569596881592056001016552563756785667227966198857827948488558343975187445455129656344348039664205579829368043522027709842942325330225763418070394769941597915945300697521482933665556615678736400536665641654732170439035213295435291694145990416087532018683793702348886894791510716378529023452924407736594956305100742108714261349745956151384987137570471017879573104229690666702144986374645952808243694457897723300487647652413390759204340196340391147320233807150952220106825634274716460243354400515212669324934196739770415956837535551667302739007497297363549645332888698440611964961627734495182736955882207573551766515898551909866653935494810688732068599075407923424023009259007017319603622547564789406475483466477604114632339056513433068449539790709030234604614709616968868850140834704054607429586991382966824681857103188790652870366508324319744047718556789348230894310682870272280973624809399627060747264553992539944280811373694338872940630792615959954626246297070625948455690347119729964090894180595343932512362355081349490043642785271383159125689892951964272875739469142725343669415323610045373048819855170659412173524625895487301676002988659257866285612496655235338294287854253404830833070165372285635591525347844598183134112900199920598135220511733658564078264849427644113763938669248031183644536985891754426473998822846218449008777697763127957226726555625962825427653183001340709223343657791601280931794017185985999338492354956400570995585611349802524990669842330173503580440811685526531170995708994273287092584878944364600504108922669178352587078595129834417295351953788553457374260859029081765155780390594640873506123226112009373108048548526357228257682034160504846627750450031262008007998049254853469414697751649327095049346393824322271885159740547021482897111777923761225788734771881968254629812686858170507402725502633290449762778944236216741191862694396506715157795867564823993917604260176338704549901761436412046921823707648878341968968611815581587360629386038101712158552726683008238340465647588040513808016336388742163714064354955618689641122821407533026551004241048967835285882902436709048871181909094945331442182876618103100735477054981596807720094746961343609286148494178501718077930681085469000944589952794243981392135055864221964834915126390128038320010977386806628779239718014613432445726400973742570073592100315415089367930081699805365202760072774967458400283624053460372634165542590276018348403068113818551059797056640075094260878857357
3.14159265358979323846264338327950288419716939937510582097494459230781640628620899862803482534211706798214808651328230664709384460955058223172535940812848111745028410270193852110555964462294895493038196442881097566593344612847564823378678316527120190914564856692346034861045432664821339360726024914127372458700660631558817488152092096282925409171536436789259036001133053054882046652138414695194151160943305727036575959195309218611738193261179310511854807446237996274956735188575272489122793818301194912983367336244065664308602139494639522473719070217986094370277053921717629317675238467481846766940513200056812714526356082778577134275778960917363717872146844090122495343014654958537105079227968925892354201995611212902196086403441815981362977477130996051870721134999999837297804995105973173281609631859502445945534690830264252230825334468503526193118817101000313783875288658753320838142061717766914730359825349042875546873115956286388235378759375195778185778053217122680661300192787661119590921642019893809525720106548586327886593615338182796823030195203530185296899577362259941389124972177528347913151557485724245415069595082953311686172785588907509838175463746493931925506040092770167113900984882401285836160356370766010471018194295559619894676783744944825537977472684710404753464620804668425906949129331367702898915210475216205696602405803815019351125338243003558764024749647326391419927260426992279678235478163600934172164121992458631503028618297455570674983850549458858692699569092721079750930295532116534498720275596023648066549911988183479775356636980742654252786255181841757467289097
```
Looks like those are 30973, 12424, and 1591 digits, respectively.

Did some cleanup, then deployed a Round 2:
```
Defaulted container "pi" out of: pi, init-dp (init)
Defaulted container "pi" out of: pi, init-dp (init)
Defaulted container "pi" out of: pi, init-dp (init)
3.141592653589793238462643383279502884197169399375105820974944592307816406286208998628034825342117067982148086513282306647093844609550582231725359408128481117450284102701938521105559644622948954930381964428810975665933446128475648233786783165271201909145648566923460348610454326648213393607260249141273724587006606315588174881520920962829254091715364367892590360011330530548820466521384146951941511609433057270365759591953092186117381932611793105118548074462379962749567351885752724891227938183011949129833673362440656643086021394946395224737190702179860943702770539217176293176752384674818467669405132000568127145263560827785771342757789609173637178721468440901224953430146549585371050792279689258923542019956112129021960864034418159813629774771309960518707211349999998372978049951059731732816096318595024459455346908302642522308253344685035261931188171010003137838752886587533208381420617177669147303598253490428755468731159562863882353787593751957781857780532171226806613001927876611195909216420198938095257201065485863278865936153381827968230301952035301852968995773622599413891249721775283479131515574857242454150695950829533116861727855889075098381754637464939319255060400927701671139009848824012858361603563707660104710181942955596198946767837449448255379774726847104047534646208046684259069491293313677028989152104752162056966024058038150193511253382430035587640247496473263914199272604269922796782354781636009341721641219924586315030286182974555706749838505494588586926995690927210797509302955321165344987202755960236480665499119881834797753566369807426542527862551818417574672890977772793800081647060016145249192173217214772350141441973568548161361157352552133475741849468438523323907394143334547762416862518983569485562099219222184272550254256887671790494601653466804988627232791786085784383827967976681454100953883786360950680064225125205117392984896084128488626945604241965285022210661186306744278622039194945047123713786960956364371917287467764657573962413890865832645995813390478027590099465764078951269468398352595709825822620522489407726719478268482601476990902640136394437455305068203496252451749399651431429809190659250937221696461515709858387410597885959772975498930161753928468138268683868942774155991855925245953959431049972524680845987273644695848653836736222626099124608051243884390451244136549762780797715691435997700129616089441694868555848406353422072225828488648158456028506016842739452267467678895252138522549954666727823986456596116354886230577456498035593634568174324112515076069479451096596094025228879710893145669136867228748940560101503308617928680920874760917824938589009714909675985261365549781893129784821682998948722658804857564014270477555132379641451523746234364542858444795265867821051141354735739523113427166102135969536231442952484937187110145765403590279934403742007310578539062198387447808478489683321445713868751943506430218453191048481005370614680674919278191197939952061419663428754440643745123718192179998391015919561814675142691239748940907186494231961567945208095146550225231603881930142093762137855956638937787083039069792077346722182562599661501421503068038447734549202605414665925201497442850732518666002132434088190710486331734649651453905796268561005508106658796998163574736384052571459102897064140110971206280439039759515677157700420337869936007230558763176359421873125147120532928191826186125867321579198414848829164470609575270695722091756711672291098169091528017350671274858322287183520935396572512108357915136988209144421006751033467110314126711136990865851639831501970165151168517143765761835155650884909989859982387345528331635507647918535893226185489632132933089857064204675259070915481416549859461637180270981994309924488957571282890592323326097299712084433573265489382391193259746366730583604142813883032038249037589852437441702913276561809377344403070746921120191302033038019762110110044929321516084244485963766983895228684783123552658213144957685726243344189303968642624341077322697802807318915441101044682325271620105265227211166039666557309254711055785376346682065310989652691862056476931257058635662018558100729360659876486117910453348850346113657686753249441668039626579787718556084552965412665408530614344431858676975145661406800700237877659134401712749470420562230538994561314071127000407854733269939081454664645880797270826683063432858785698305235808933065757406795457163775254202114955761581400250126228594130216471550979259230990796547376125517656751357517829666454779174501129961489030463994713296210734043751895735961458901938971311179042978285647503203198691514028708085990480109412147221317947647772622414254854540332157
3.141592653589793238462643383279502884197169399375105820974944592307816406286208998628034825342117067982148086513282306647093844609550582231725359408128481117450284102701938521105559644622948954930381964428810975665933446128475648233786783165271201909145648566923460348610454326648213393607260249141273724587006606315588174881520920962829254091715364367892590360011330530548820466521384146951941511609433057270365759591953092186117381932611793105118548074462379962749567351885752724891227938183011949129833673362440656643086021394946395224737190702179860943702770539217176293176752384674818467669405132000568127145263560827785771342757789609173637178721468440901224953430146549585371050792279689258923542019956112129021960864034418159813629774771309960518707211349999998372978049951059731732816096318595024459455346908302642522308253344685035261931188171010003137838752886587533208381420617177669147303598253490428755468731159562863882353787593751957781857780532171226806613001927876611195909216420198938095257201065485863278865936153381827968230301952035301852968995773622599413891249721775283479131515574857242454150695950829533116861727855889075098381754637464939319255060400927701671139009848824012858361603563707660104710181942955596198946767837449448255379774726847104047534646208046684259069491293313677028989152104752162056966024058038150193511253382430035587640247496473263914199272604269922796782354781636009341721641219924586315030286182974555706749838505494588586926995690927210797509302955321165344987202755960236480665499119881834797753566369807426542527862551818417574672890977772793800081647060016145249192173217214772350141441973568548161361157352552133475741849468438523323907394143334547762416862518983569485562099219222184272550254256887671790494601653466804988627232791786085784383827967976681454100953883786360950680064225125205117392984896084128488626945604241965285022210661186306744278622039194945047123713786960956364371917287467764657573962413890865832645995813390478027590099465764078951269468398352595709825822620522489407726719478268482601476990902640136394437455305068203496252451749399651431429809190659250937221696461515709858387410597885959772975498930161753928468138268683868942774155991855925245953959431049972524680845987273644695848653836736222626099124608051243884390451244136549762780797715691435997700129616089441694868555848406353422072225828488648158456028506016842739452267467678895252138522549954666727823986456596116354886230577456498035593634568174324112515076069479451096596094025228879710893145669136867228748940560101503308617928680920874760917824938589009714909675985261365549781893129784821682998948722658804857564014270477555132379641451523746234364542858444795265867821051141354735739523113427166102135969536231442952484937187110145765403590279934403742007310578539062198387447808478489683321445713868751943506430218453191048481005370614680674919278191197939952061419663428754440643745123718192179998391015919561814675142691239748940907186494231961567945208095146550225231603881930142093762137855956638937787083039069792077346722182562599661501421503068038447734549202605414665925201497442850732518666002132434088190710486331734649651453905796268561005508106658796998163574736384052571459102897064140110971206280439039759515677157700420337869936007230558763176359421873125147120532928191826186125867321579198414848829164470609575270695722091756711672291098169091528017350671274858322287183520935396572512108357915136988209144421006751033467110314126711136990865851639831501970165151168517143765761835155650884909989859982387345528331635507647918535893226185489632132933089857064204675259070915481416549859461637180270981994309924488957571282890592323326097299712084433573265489382391193259746366730583604142813883032038249037589852437441702913276561809377344403070746921120191302033038019762110110044929321516084244485963766983895228684783123552658213144957685726243344189303968642624341077322697802807318915441101044682325271620105265227211166039666557309254711055785376346682065310989652691862056476931257058635662018558100729360659876486117910453348850346113657686753249441668039626579787718556084552965412665408530614344431858676975145661406800700237877659134401712749470420562230538994561314071127000407854733269939081454664645880797270826683063432858785698305235808933065757406795457163775254202114955761581400250126228594130216471550979259230990796547376125517656751357517829666454779174501129961489030463994713296210734043751895735961458901938971311179042978285647503203198691514028708085990480109412147221317947647772622414254854540332157185306142288137585043063321751829798662237172159160771669254748738986654949450114654062843366393790039769265672146385306736096571209180763832716641627488880078692560290228472104031721186082041900042296617119637792133757511495950156604963186294726547364252308177036751590673502350728354056704038674351362222477158915049530984448933309634087807693259939780541934144737744184263129860809988868741326047215695162396586457302163159819319516735381297416772947867242292465436680098067692823828068996400482435403701416314965897940924323789690706977942236250822168895738379862300159377647165122893578601588161755782973523344604281512627203734314653197777416031990665541876397929334419521541341899485444734567383162499341913181480927777103863877343177207545654532207770921201905166096280490926360197598828161332316663652861932668633606273567630354477628035045077723554710585954870279081435624014517180624643626794561275318134078330336254232783944975382437205835311477119926063813346776879695970309833913077109870408591337464144282277263465947047458784778720192771528073176790770715721344473060570073349243693113835049316312840425121925651798069411352801314701304781643788518529092854520116583934196562134914341595625865865570552690496520985803385072242648293972858478316305777756068887644624824685792603953527734803048029005876075825104747091643961362676044925627420420832085661190625454337213153595845068772460290161876679524061634252257719542916299193064553779914037340432875262888963995879475729174642635745525407909145135711136941091193932519107602082520261879853188770584297259167781314969900901921169717372784768472686084900337702424291651300500516832336435038951702989392233451722013812806965011784408745196012122859937162313017114448464090389064495444006198690754851602632750529834918740786680881833851022833450850486082503930213321971551843063545500766828294930413776552793975175461395398468339363830474611996653858153842056853386218672523340283087112328278921250771262946322956398989893582116745627010218356462201349671518819097303811980049734072396103685406643193950979019069963955245300545058068550195673022921913933918568034490398205955100226353536192041994745538593810234395544959778377902374216172711172364343543947822181852862408514006660443325888569867054315470696574745855033232334210730154594051655379068662733379958511562578432298827372319898757141595781119635833005940873068121602876496286744604774649159950549737425626901049037781986835938146574126804925648798556145372347867330390468838343634655379498641927056387293174872332083760112302991136793862708943879936201629515413371424892830722012690147546684765357616477379467520049075715552781965362132392640616013635815590742202020318727760527721900556148425551879253034351398442532234157623361064250639049750086562710953591946589751413103482276930624743536325691607815478181152843667957061108615331504452127473924544945423682886061340841486377670096120715124914043027253860764823634143346235189757664521641376796903149501910857598442391986291642193994907236234646844117394032659184044378051333894525742399508296591228508555821572503107125701266830240292952522011872676756220415420516184163484756516999811614101002996078386909291603028840026910414079288621507842451670908700069928212066041837180653556725253256753286129104248776182582976515795984703562226293486003415872298053498965022629174878820273420922224533985626476691490556284250391275771028402799806636582548892648802545661017296702664076559042909945681506526530537182941270336931378517860904070866711496558343434769338578171138645587367812301458768712660348913909562009939361031029161615288138437909904231747336394804575931493140529763475748119356709110137751721008031559024853090669203767192203322909433467685142214477379393751703443661991040337511173547191855046449026365512816228824462575916333039107225383742182140883508657391771509682887478265699599574490661758344137522397096834080053559849175417381883999446974867626551658276584835884531427756879002909517028352971634456212964043523117600665101241200659755851276178583829204197484423608007193045761893234922927965019875187212726750798125547095890455635792122103334669749923563025494780249011419521238281530911407907386025152274299581807247162591668545133312394804947079119153267343028244186041426363954800044800267049624820179289647669758318327131425170296923488962766844032326092752496035799646925650493681836090032380929345958897069536534940603402166544375589004563288225054525564056448246515187547119621844396582533754388569094113031509526179378002974120766514793942590298969594699556576121865619673378623625612521632086286922210327488921865436480229678070576561514463204692790682120738837781423356282360896320806822246801224826117718589638140918390367367222088832151375560037279839400415297002878307667094447456013455641725437090697939612257142989467154357846878861444581231459357198492252847160504922124247014121478057345510500801908699603302763478708108175450119307141223390866393833952942578690507643100638351983438934159613185434754649556978103829309716465143840700707360411237359984345225161050702705623526601276484830840761183013052793205427462865403603674532865105706587488225698157936789766974220575059683440869735020141020672358502007245225632651341055924019027421624843914035998953539459094407046912091409387001264560016237428802109276457931065792295524988727584610126483699989225695968815920560010165525637567856672279661988578279484885583439751874454551296563443480396642055798293680435220277098429423253302257634180703947699415979159453006975214829336655566156787364005366656416547321704390352132954352916941459904160875320186837937023488868947915107163785290234529244077365949563051007421087142613497459561513849871375704710178795731042296906667021449863746459528082436944578977233004876476524133907592043401963403911473202338071509522201068256342747164602433544005152126693249341967397704159568375355516673027390074972973635496453328886984406119649616277344951827369558822075735517665158985519098666539354948106887320685990754079234240230092590070173196036225475647894064754834664776041146323390565134330684495397907090302346046147096169688688501408347040546074295869913829668246818571031887906528703665083243197440477185567893482308943106828702722809736248093996270607472645539925399442808113736943388729406307926159599546262462970706259484556903471197299640908941805953439325123623550813494900436427852713831591256898929519642728757394691427253436694153236100453730488198551706594121735246258954873016760029886592578662856124966552353382942878542534048308330701653722856355915253478445981831341129001999205981352205117336585640782648494276441137639386692480311836445369858917544264739988228462184490087776977631279572267265556259628254276531830013407092233436577916012809317940171859859993384923549564005709955856113498025249906698423301735035804408116855265311709957089942732870925848789443646005041089226691783525870785951298344172953519537885534573742608590290817651557803905946408735061232261120093731080485485263572282576820341605048466277504500312620080079980492548534694146977516493270950493463938243222718851597405470214828971117779237612257887347718819682546298126868581705074027255026332904497627789442362167411918626943965067151577958675648239939176042601763387045499017614364120469218237076488783419689686118155815873606293860381017121585527266830082383404656475880405138080163363887421637140643549556186896411228214075330265510042410489678352858829024367090488711819090949453314421828766181031007354770549815968077200947469613436092861484941785017180779306810854690009445899527942439813921350558642219648349151263901280383200109773868066287792397180146134324457264009737425700735921003154150893679300816998053652027600727749674584002836240534603726341655425902760183484030681138185510597970566400750942608788573579603732451414678670368809880609716425849759513806930944940151542222194329130217391253835591503100333032511174915696917450271494331515588540392216409722910112903552181576282328318234254832611191280092825256190205263016391147724733148573910777587442538761174657867116941477642144111126358355387136101102326798775641024682403226483464176636980663785768134920453022408197278564719839630878154322116691224641591177673225326433568614618654522268126887268445968442416107854016768142080885028005414361314623082102594173756238994207571362751674573189189456283525704413354375857534269869947254703165661399199968262824727064133622217892390317608542894373393561889165125042440400895271983787386480584726895462438823437517885201439560057104811949884239060613695734231559079670346149143447886360410318235073650277859089757827273130504889398900992391350337325085598265586708924261242947367019390772713070686917092646254842324074855036608013604668951184009366860954632500214585293095000090715105823626729326453738210493872499669933942468551648326113414611068026744663733437534076429402668297386522093570162638464852851490362932019919968828517183953669134522244470804592396602817156551565666111359823112250628905854914509715755390024393153519090210711945730024388017661503527086260253788179751947806101371500448991721002220133501310601639154158957803711779277522597874289191791552241718958536168059474123419339842021874564925644346239253195313510331147639491199507285843065836193536932969928983791494193940608572486396883690326556436421664425760791471086998431573374964883529276932822076294728238153740996154559879825989109371712621828302584811238901196822142945766758071865380650648702613389282299497257453033283896381843944770779402284359883410035838542389735424395647555684095224844554139239410001620769363684677641301781965937997155746854194633489374843912974239143365936041003523437770658886778113949861647874714079326385873862473288964564359877466763847946650407411182565837887845485814896296127399841344272608606187245545236064315371011274680977870446409475828034876975894832824123929296058294861919667091895808983320121031843034012849511620353428014412761728583024355983003204202451207287253558119584014918096925339507577840006746552603144616705082768277222353419110263416315714740612385042584598841990761128725805911393568960143166828317632356732541707342081733223046298799280490851409479036887868789493054695570307261900950207643349335910602454508645362893545686295853131533718386826561786227363716975774183023986006591481616404944965011732131389574706208847480236537103115089842799275442685327797431139514357417221975979935968525228574526379628961269157235798662057340837576687388426640599099350500081337543245463596750484423528487470144354541957625847356421619813407346854111766883118654489377697956651727966232671481033864391375186594673002443450054499539974237232871249483470604406347160632583064982979551010954183623503030945309733583446283947630477564501500850757894954893139394489921612552559770143685894358587752637962559708167764380012543650237141278346792610199558522471722017772370041780841942394872540680155603599839054898572354674564239058585021671903139526294455439131663134530893906204678438778505423939052473136201294769187497519101147231528932677253391814660730008902776896311481090220972452075916729700785058071718638105496797310016787085069420709223290807038326345345203802786099055690013413718236837099194951648960075504934126787643674638490206396401976668559233565463913836318574569814719621084108096188460545603903845534372914144651347494078488442377217515433426030669883176833100113310869042193903108014378433415137092435301367763108491351615642269847507430329716746964066653152703532546711266752246055119958183196376370761799191920357958200759560530234626775794393630746305690108011494271410093913691381072581378135789400559950018354251184172136055727522103526803735726527922417373605751127887218190844900617801388971077082293100279766593583875890939568814856026322439372656247277603789081445883785501970284377936240782505270487581647032458129087839523245323789602984166922548964971560698119218658492677040395648127810217991321741630581055459880130048456299765112124153637451500563507012781592671424134210330156616535602473380784302865525722275304999883701534879300806260180962381516136690334111138653851091936739383522934588832255088706450753947395204396807906708680644509698654880168287434378612645381583428075306184548590379821799459968115441974253634439960290251001588827216474500682070419376158454712318346007262933955054823955713725684023226821301247679452264482091023564775272308208106351889915269288910845557112660396503439789627825001611015323516051965590421184494990778999200732947690586857787872098290135295661397888486050978608595701773129815531495168146717695976099421003618355913877781769845875810446628399880600616229848616935337386578773598336161338413385368421197893890018529569196780455448285848370117096721253533875862158231013310387766827211572694951817958975469399264219791552338576623167627547570354699414892904130186386119439196283887054367774322427680913236544948536676800000106526248547305586159899914017076983854831887501429389089950685453076511680333732226517566220752695179144225280816517166776672793035485154204023817460892328391703275425750867655117859395002793389592057668278967764453184040418554010435134838953120132637836928358082719378312654961745997056745071833206503455664403449045362756001125018433560736122276594927839370647842645676338818807565612168960504161139039063960162022153684941092605387688714837989559999112099164646441191856827700457424343402167227644558933012778158686952506949936461017568506016714535431581480105458860564550133203758645485840324029871709348091055621167154684847780394475697980426318099175642280987399876697323769573701580806822904599212366168902596273043067931653114940176473769387351409336183321614280214976339918983548487562529875242387307755955595546519639440182184099841248982623673771467226061633643296406335728107078875816404381485018841143188598827694490119321296827158884133869434682859006664080631407775772570563072940049294030242049841656547973670548558044586572022763784046682337985282710578431975354179501134727362577408021347682604502285157979579764746702284099956160156910890384582450267926594205550395879229818526480070683765041836562094555434613513415257006597488191634135955671964965403218727160264859304903978748958906612725079482827693895352175362185079629778514618843271922322381015874445052866523802253284389137527384589238442253547265309817157844783421582232702069028723233005386216347988509469547200479523112015043293226628272763217790884008786148022147537657810581970222630971749507212724847947816957296142365859578209083073323356034846531873029302665964501371837542889755797144992465403868179921389346924474198509733462679332107268687076806263991936196504409954216762784091466985692571507431574079380532392523947755744159184582156251819215523370960748332923492103451462643744980559610330799414534778457469999212859999939961228161521931488876938802228108300198601654941654261696858678837260958774567618250727599295089318052187292461086763995891614585505839727420980909781729323930106766386824040111304024700735085782872462713494636853181546969046696869392547251941399291465242385776255004748529547681479546700705034799958886769501612497228204030399546327883069597624936151010243655535223069061294938859901573466102371223547891129254769617600504797492806072126803922691102777226102544149221576504508120677173571202718024296810620377657883716690910941807448781404907551782038565390991047759414132154328440625030180275716965082096427348414695726397884256008453121406593580904127113592004197598513625479616063228873618136737324450607924411763997597461938358457491598809766744709300654634242346063423747466608043170126005205592849369594143408146852981505394717890045183575515412522359059068726487863575254191128887737176637486027660634960353679470269232297186832771739323619200777452212624751869833495151019864269887847171939664976907082521742336566272592844062043021411371992278526998469884770232382384005565551788908766136013047709843861168705231055314916251728373272867600724817298763756981633541507460883866364069347043720668865127568826614973078865701568501691864748854167915459650723428773069985371390430026653078398776385032381821553559732353068604301067576083890862704984188859513809103042359578249514398859011318583584066747237029714978508414585308578133915627076035639076394731145549583226694570249413983163433237897595568085683629725386791327505554252449194358912840504522695381217913191451350099384631177401797151228378546011603595540286440590249646693070776905548102885020808580087811577381719174177601733073855475800605601433774329901272867725304318251975791679296996504146070664571258883469797964293162296552016879730003564630457930884032748077181155533090988702550520768046303460865816539487695196004408482065967379473168086415645650530049881616490578831154345485052660069823093157776500378070466126470602145750579327096204782561524714591896522360839664562410519551052235723973951288181640597859142791481654263289200428160913693777372229998332708208296995573772737566761552711392258805520189887620114168005468736558063347160373429170390798639652296131280178267971728982293607028806908776866059325274637840539769184808204102194471971386925608416245112398062011318454124478205011079876071715568315407886543904121087303240201068534194723047666672174986986854707678120512473679247919315085644477537985379973223445612278584329684664751333657369238720146472367942787004250325558992688434959287612400755875694641370562514001179713316620715371543600687647731867558714878398908107429530941060596944315847753970094398839491443235366853920994687964506653398573888786614762944341401049888993160051207678103588611660202961193639682134960750111649832785635316145168457695687109002999769841263266502347716728657378579085746646077228341540311441529418804782543876177079043000156698677679576090996693607559496515273634981189641304331166277471233881740603731743970540670310967676574869535878967003192586625941051053358438465602339179674926784476370847497833365557900738419147319886271352595462518160434225372996286326749682405806029642114638643686422472488728343417044157348248183330164056695966886676956349141632842641497453334999948000266998758881593507357815195889900539512085351035726137364034367534714104836017546488300407846416745216737190483109676711344349481926268111073994825060739495073503169019731852119552635632584339099822498624067031076831844660729124874754031617969941139738776589986855417031884778867592902607004321266617919223520938227878880988633599116081923535557046463491132085918979613279131975649097600013996234445535014346426860464495862476909434704829329414041114654092398834443515913320107739441118407410768498106634724104823935827401944935665161088463125678529776973468430306146241803585293315973458303845541033701091676776374276210213701354854450926307190114731848574923318167207213727935567952844392548156091372812840633303937356242001604566455741458816605216660873874804724339121295587776390696903707882852775389405246075849623157436917113176134783882719416860662572103685132156647800147675231039357860689611125996028183930954870905907386135191459181951029732787557104972901148717189718004696169777001791391961379141716270701895846921434369676292745910994006008498356842520191559370370101104974733949387788598941743303178534870760322198297057975119144051099423588303454635349234982688362404332726741554030161950568065418093940998202060999414021689090070821330723089662119775530665918814119157783627292746156185710372172471009521423696483086410259288745799932237495519122195190342445230753513380685680735446499512720317448719540397610730806026990625807602029273145525207807991418429063884437349968145827337207266391767020118300464819000241308350884658415214899127610651374153943565721139032857491876909441370209051703148777346165287984823533829726013611098451484182380812054099612527458088109948697221612852489742555551607637167505489617301680961380381191436114399210638005083214098760459930932485102516829446726066613815174571255975495358023998314698220361338082849935670557552471290274539776214049318201465800802156653606776550878380430413431059180460680083459113664083488740800574127258670479225831912741573908091438313845642415094084913391809684025116399193685322555733896695374902662092326131885589158083245557194845387562878612885900410600607374650140262782402734696252821717494158233174923968353013617865367376064216677813773995100658952887742766263684183068019080460984980946976366733566228291513235278880615776827815958866918023894033307644191240341202231636857786035727694154177882643523813190502808701857504704631293335375728538660588890458311145077394293520199432197117164223500564404297989208159430716701985746927384865383343614579463417592257389858800169801475742054299580124295810545651083104629728293758416116253256251657249807849209989799062003593650993472158296517413579849104711166079158743698654122234834188772292944633517865385673196255985202607294767407261676714557364981210567771689348491766077170527718760119990814411305864557791052568430481144026193840232247093924980293355073184589035539713308844617410795916251171486487446861124760542867343670904667846867027409188101424971114965781772427934707021668829561087779440504843752844337510882826477197854000650970403302186255614733211777117441335028160884035178145254196432030957601869464908868154528562134698835544456024955666843660292219512483091060537720198021831010327041783866544718126039719068846237085751808003532704718565949947612424811099928867915896904956394762460842406593094862150769031498702067353384834955083636601784877106080980426924713241000946401437360326564518456679245666955100150229833079849607994988249706172367449361226222961790814311414660941234159359309585407913908720832273354957208075716517187659944985693795623875551617575438091780528029464200447215396280746360211329425591600257073562812638733106005891065245708024474937543184149401482119996276453106800663118382376163966318093144467129861552759820145141027560068929750246304017351489194576360789352855505317331416457050499644389093630843874484783961684051845273288403234520247056851646571647713932377551729479512613239822960239454857975458651745878771331813875295980941217422730035229650808917770506825924882232215493804837145478164721397682096332050830564792048208592047549985732038887639160199524091893894557676874973085695595801065952650303626615975066222508406742889826590751063756356996821151094966974458054728869363102036782325018232370845979011154847208761821247781326633041207621658731297081123075815982124863980721240786887811450165582513617890307086087019897588980745664395515741536319319198107057533663373803827215279884935039748001589051942087971130805123393322190346624991716915094854140187106035460379464337900589095772118080446574396280618671786101715674096766208029576657705129120990794430463289294730615951043090222143937184956063405618934251305726829146578329334052463502892917547087256484260034962961165413823007731332729830500160256724014185152041890701154288579920812198449315699905918201181973350012618772803681248199587707020753240636125931343859554254778196114293516356122349666152261473539967405158499860355295332924575238881013620234762466905581643896786309762736550472434864307121849437348530060638764456627218666170123812771562137974614986132874411771455244470899714452288566294244023018479120547849857452163469644897389206240194351831008828348024924908540307786387516591130287395878709810077271827187452901397283661484214287170553179654307650453432460053636147261818096997693348626407743519992868632383508875668359509726557481543194019557685043724800102041374983187225967738715495839971844490727914196584593008394263702087563539821696205532480321226749891140267852859967340524203109179789990571882194939132075343170798002373659098537552023891164346718558290685371189795262623449248339249634244971465684659124891855662958932990903523923333364743520370770101084388003290759834217018554228386161721041760301164591878053936744747205998502358289183369292233732399948043710841965947316265482574809948250999183300697656936715968936449334886474421350084070066088359723503953234017958255703601693699098867113210979889707051728075585519126993067309925070407024556850778679069476612629808225163313639952117098452809263037592242674257559989289278370474445218936320348941552104459726188380030067761793138139916205806270165102445886924764924689192461212531027573139084047000714356136231699237169484813255420091453041037135453296620639210547982439212517254013231490274058589206321758949434548906846399313757091034633271415316223280552297297953801880162859073572955416278867649827418616421878988574107164906919185116281528548679417363890665388576422915834250067361245384916067413734017357277995634104332688356950781493137800736235418007061918026732855119194267609122103598746924117283749312616339500123959924050845437569850795704622266461900010350049018303415354584283376437811198855631877779253720116671853954183598443830520376281944076159410682071697030228515225057312609304689842343315273213136121658280807521263154773060442377475350595228717440266638914881717308643611138906942027908814311944879941715404210341219084709408025402393294294549387864023051292711909751353600092197110541209668311151632870542302847007312065803262641711616595761327235156666253667271899853419989523688483099930275741991646384142707798870887422927705389122717248632202889842512528721782603050099451082478357290569198855546788607946280537122704246654319214528176074148240382783582971930101788834567416781139895475044833931468963076339665722672704339321674542182455706252479721997866854279897799233957905758189062252547358220523642485078340711014498047872669199018643882293230538231855973286978092225352959101734140733488476100556401824239219269506208318381454698392366461363989101210217709597670490830508185470419466437131229969235889538493013635657618610606222870559942337163102127845744646398973818856674626087948201864748767272722206267646533809980196688368099415907577685263986514625333631245053640261056960551318381317426118442018908885319635698696279503673842431301133175330532980201668881748134298868158557781034323175306478498321062971842518438553442762012823457071698853051832617964117857960888815032960229070561447622091509473903594664691623539680920139457817589108893199211226007392814916948161527384273626429809823406320024402449589445612916704950823581248739179964864113348032475777521970893277226234948601504665268143987705161531702669692970492831628550421289814670619533197026950721437823047687528028735412616639170824592517001071418085480063692325946201900227808740985977192180515853214739265325155903541020928466592529991435379182531454529059841581763705892790690989691116438118780943537152133226144362531449012745477269573939348154691631162492887357471882407150399500944673195431619385548520766573882513963916357672315100555603726339486720820780865373494244011579966750736071115935133195919712094896471755302453136477094209463569698222667377520994516845064362382421185353488798939567318780660610788544000550827657030558744854180577889171920788142335113866292966717964346876007704799953788338787034871802184243734211227394025571769081960309201824018842705704609262256417837526526335832424066125331152942345796556950250681001831090041124537901533296615697052237921032570693705109083078947999900499939532215362274847660361367769797856738658467093667958858378879562594646489137665219958828693380183601193236857855855819555604215625088365020332202451376215820461810670519533065306060650105488716724537794283133887163139559690583208341689847606560711834713621812324622725884199028614208728495687963932546428534307530110528571382964370999035694888528519040295604734613113826387889755178856042499874831638280404684861893818959054203988987265069762020199554841265000539442820393012748163815853039643992547020167275932857436666164411096256633730540921951967514832873480895747777527834422109107311135182804603634719818565557295714474768255285786334934285842311874944000322969069775831590385803935352135886007960034209754739229673331064939560181223781285458431760556173386
3.14159265358979323846264338327950288419716939937510582097494459230781640628620899862803482534211706798214808651328230664709384460955058223172535940812848111745028410270193852110555964462294895493038196442881097566593344612847564823378678316527120190914564856692346034861045432664821339360726024914127372458700660631558817488152092096282925409171536436789259036001133053054882046652138414695194151160943305727036575959195309218611738193261179310511854807446237996274956735188575272489122793818301194912983367336244065664308602139494639522473719070217986094370277053921717629317675238467481846766940513200056812714526356082778577134275778960917363717872146844090122495343014654958537105079227968925892354201995611212902196086403441815981362977477130996051870721134999999837297804995105973173281609631859502445945534690830264252230825334468503526193118817101000313783875288658753320838142061717766914730359825349042875546873115956286388235378759375195778185778053217122680661300192787661119590921642019893809525720106548586327886593615338182796823030195203530185296899577362259941389124972177528347913151557485724245415069595082953311686172785588907509838175463746493931925506040092770167113900984882401285836160356370766010471018194295559619894676783744944825537977472684710404753464620804668425906949129331367702898915210475216205696602405803815019351125338243003558764024749647326391419927260426992279678235478163600934172164121992458631503028618297455570674983850549458858692699569092721079750930295532116534498720275596023648066549911988183479775356636980742654252786255181841757467289097777279380008164706001614524919217321721477235014144197356854816136115735255213347574184946843852332390739414333454776241686251898356948556209921922218427255025425688767179049460165346680498862723279178608578438382796797668145410095388378636095068006422512520511739298489608412848862694560424196528502221066118630674427862203919494504712371378696095636437191728746776465757396241389086583264599581339047802759009946576407895126946839835259570982582262052248940772671947826848260147699090264013639443745530506820349625245174939965143142980919065925093722169646151570985838741059788595977297549893016175392846813826868386894277415599185592524595395943104997252468084598727364469584865383673622262609912460805124388439045124413654976278079771569143599770012961608944169486855584840635342207222582848864815845602850601684273945226746767889525213852254995466672782398645659611635488623057745649803559363456817432411251507606947945109659609402522887971089314566913686722874894056010150330861792868092087476091782493858900971490967598526136554978189312978482168299894872265880485756401427047755513237964145152374623436454285844479526586782105114135473573952311342716610213596953623144295248493718711014576540359027993440374200731057853906219838744780847848968332144571386875194350643021845319104848100537061468067491927819119793995206141966342875444064374512371819217999839101591956181467514269123974894090718649423196156794520809514655022523160388193014209376213785595663893778708303906979207734672218256259966150142150306803844773454920260541466592520149744285073251866600213243408819071048633173464965145390579626856100550810665879699816357473638405257145910289706414011097120628043903975951567715770042033786993600723055876317635942187312514712053292819182618612586732157919841484882916447060957527069572209175671167229109816909152801735067127485832228718352093539657251210835791513698820914442100675103346711031412671113699086585163983150197016515116851714376576183515565088490998985998238734552833163550764791853589322618548963213293308985706420467525907091548141654985946163718027098199430992448895757128289059232332609729971208443357326548938239119325974636673058360414281388303203824903758985243744170291327656180937734440307074692112019130203303801976211011004492932151608424448596376698389522868478312355265821314495768572624334418930396864262434107732269780280731891544110104468232527162010526522721116603966655730925471105578537634668206531098965269186205647693125705863566201855810072936065987648611791045334885034611365768675324944166803962657978771855608455296541266540853061434443185867697514566140680070023787765913440171274947042056223053899456131407112700040785473326993908145466464588079727082668306343285878569830523580893306575740679545716377525420211495576158140025012622859413021647155097925923099079654737612551765675135751782966645477917450112996148903046399471329621073404375189573596145890193897131117904297828564750320319869151402870808599048010941214722131794764777262241425485454033215718530614228813758504306332175182979866223717215916077166925474873898665494945011465406284336639379003976926567214638530673609657120918076383271664162748888007869256029022847210403172118608204190004229661711963779213375751149595015660496318629472654736425230817703675159067350235072835405670403867435136222247715891504953098444893330963408780769325993978054193414473774418426312986080998886874132604721569516239658645730216315981931951673538129741677294786724229246543668009806769282382806899640048243540370141631496589794092432378969070697794223625082216889573837986230015937764716512289357860158816175578297352334460428151262720373431465319777741603199066554187639792933441952154134189948544473456738316249934191318148092777710386387734317720754565453220777092120190516609628049092636019759882816133231666365286193266863360627356763035447762803504507772355471058595487027908143562401451718062464362679456127531813407833033625423278394497538243720583531147711992606381334677687969597030983391307710987040859133746414428227726346594704745878477872019277152807317679077071572134447306057007334924369311383504931631284042512192565179806941135280131470130478164378851852909285452011658393419656213491434159562586586557055269049652098580338507224264829397285847831630577775606888764462482468579260395352773480304802900587607582510474709164396136267604492562742042083208566119062545433721315359584506877246029016187667952406163425225771954291629919306455377991403734043287526288896399587947572917464263574552540790914513571113694109119393251910760208252026187985318877058429725916778131496990090192116971737278476847268608490033770242429165130050051683233643503895170298939223345172201381280696501178440874519601212285993716231301711444846409038906449544400619869075485160263275052983491874078668088183385102283345085048608250393021332197155184306354550076682829493041377655279397517546139539846833936383047461199665385815384205685338621867252334028308711232827892125077126294632295639898989358211674562701021835646220134967151881909730381198004973407239610368540664319395097901906996395524530054505806855019567302292191393391856803449039820595510022635353619204199474553859381023439554495977837790237421617271117236434354394782218185286240851400666044332588856986705431547069657474585503323233421073015459405165537906866273337995851156257843229882737231989875714159578111963583300594087306812160287649628674460477464915995054973742562690104903778198683593814657412680492564879855614537234786733039046883834363465537949864192705638729317487233208376011230299113679386270894387993620162951541337142489283072201269014754668476535761647737946752004907571555278196536213239264061601363581559074220202031872776052772190055614842555187925303435139844253223415762336106425063904975008656271095359194658975141310348227693062474353632569160781547818115284366795706110861533150445212747392454494542368288606134084148637767009612071512491404302725386076482363414334623518975766452164137679690314950191085759844239198629164219399490723623464684411739403265918404437805133389452574239950829659122850855582157250310712570126683024029295252201187267675622041542051618416348475651699981161410100299607838690929160302884002691041407928862150784245167090870006992821206604183718065355672525325675328612910424877618258297651579598470356222629348600341587229805349896502262917487882027342092222453398562647669149055628425039127577102840279980663658254889264880254566101729670266407655904290994568150652653053718294127033693137851786090407086671149655834343476933857817113864558736781230145876871266034891390956200993936103102916161528813843790990423174733639480457593149314052976347574811935670911013775172100803155902485309066920376719220332290943346768514221447737939375170344366199104033751117354719185504644902636551281622882446257591633303910722538374218214088350865739177150968288747826569959957449066175834413752239709683408005355984917541738188399944697486762655165827658483588453142775687900290951702835297163445621296404352311760066510124120065975585127617858382920419748442360800719304576189323492292796501987518721272675079812554709589045563579212210333466974992356302549478024901141952123828153091140790738602515227429958180724716259166854513331239480494707911915326734302824418604142636395480004480026704962482017928964766975831832713142517029692348896276684403232609275249603579964692565049368183609003238092934595889706953653494060340216654437558900456328822505452556405644824651518754711962184439658253375438856909411303150952617937800297412076651479394259029896959469955657612186561967337862362561252163208628692221032748892186543648022967807057656151446320469279068212073883778142335628236089632080682224680122482611771858963814091839036736722208883215137556003727983940041529700287830766709444745601345564172543709069793961225714298946715435784687886144458123145935719849225284716050492212424701412147805734551050080190869960330276347870810817545011930714122339086639383395294257869050764310063835198343893415961318543475464955697810382930971646514384070070736041123735998434522516105070270562352660127648483084076118301305279320542746286540360367453286510570658748822569815793678976697422057505968344086973502014102067235850200724522563265134105592401902742162484391403599895353945909440704691209140938700126456001623742880210927645793106579229552498872758461012648369998922569596881592056001016552563756785667227966198857827948488558343975187445455129656344348039664205579829368043522027709842942325330225763418070394769941597915945300697521482933665556615678736400536665641654732170439035213295435291694145990416087532018683793702348886894791510716378529023452924407736594956305100742108714261349745956151384987137570471017879573104229690666702144986374645952808243694457897723300487647652413390759204340196340391147320233807150952220106825634274716460243354400515212669324934196739770415956837535551667302739007497297363549645332888698440611964961627734495182736955882207573551766515898551909866653935494810688732068599075407923424023009259007017319603622547564789406475483466477604114632339056513433068449539790709030234604614709616968868850140834704054607429586991382966824681857103188790652870366508324319744047718556789348230894310682870272280973624809399627060747264553992539944280811373694338872940630792615959954626246297070625948455690347119729964090894180595343932512362355081349490043642785271383159125689892951964272875739469142725343669415323610045373048819855170659412173524625895487301676002988659257866285612496655235338294287854253404830833070165372285635591525347844598183134112900199920598135220511733658564078264849427644113763938669248031183644536985891754426473998822846218449008777697763127957226726555625962825427653183001340709223343657791601280931794017185985999338492354956400570995585611349802524990669842330173503580440811685526531170995708994273287092584878944364600504108922669178352587078595129834417295351953788553457374260859029081765155780390594640873506123226112009373108048548526357228257682034160504846627750450031262008007998049254853469414697751649327095049346393824322271885159740547021482897111777923761225788734771881968254629812686858170507402725502633290449762778944236216741191862694396506715157795867564823993917604260176338704549901761436412046921823707648878341968968611815581587360629386038101712158552726683008238340465647588040513808016336388742163714064354955618689641122821407533026551004241048967835285882902436709048871181909094945331442182876618103100735477054981596807720094746961
```

This second output has 4601, 32675, and 12136 digits.

Just for "funzies" I copied random portions (up to 13161 characters) off https://paulkaefer.com/pi and found matches above.

### deploy the CronJob and target PVC for backup files:
`kubectl apply -f todo-list/db/backup/`
Got:
```
error: resource mapping not found for name: "todo-db-backup" namespace: "" from "todo-list/db/backup/todo-db-backup-cronjob.yaml": no matches for kind "CronJob" in version "batch/v1beta1"
ensure CRDs are installed first
```
Tried selectinvely running commands from earlier in the chapter, to no avail. Tried restarting Kube cluster via Docker Desktop, too.

### wait for the Job to run--this is a good time to make tea:
`sleep 150`

### check the CronJob status:
`kubectl get cronjob todo-db-backup`
```
Error from server (NotFound): cronjobs.batch "todo-db-backup" not found
```

### now run a sleep Pod that mounts the backup PVC:
`kubectl apply -f sleep/sleep-with-db-backup-mount.yaml`
```
deployment.apps/sleep created
```

### check if the CronJob Pod created the backup:
`kubectl exec deploy/sleep -- ls -l /backup`
```
total 0
```


## Section 8.5: Choosing your platform for stateful apps

### delete all the owning objects:
`kubectl delete all -l kiamol=ch08`
```
deployment.apps "sleep" deleted
```

### delete the PVCs
`kubectl delete pvc -l kiamol=ch08`
```
persistentvolumeclaim "todo-db-backup-pvc" deleted

λ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   19h
```

## Section 8.6: Lab

`kubectl apply -f lab/nginx/`
```
service/nginx created
pod/nginx created
```

`kubectl get svc nginx -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8088'`
http://localhost:8088
* "If you see this page, the nginx web server is successfully installed and working. Further configuration is required."

`kubectl apply -f lab/solution/nginx-statefulset.yaml`
```
service/nginx configured
statefulset.apps/nginx-stateful created
```

`kubectl apply -f lab/solution/disk-calc-job.yaml`
```
job.batch/disk-calc created
```

`kubectl logs -l job-name=disk-calc`
```
0 /nginx0/access.log
4.0K  /nginx1/access.log
4.0K  /nginx2/access.log
```

Based on book code, but adapted from [here](https://stackoverflow.com/questions/12409519/curl-command-to-repeat-url-request):
```bash
for i in `seq 1 100`; do curl http://localhost:8088 > /dev/null; done
```

# Chapter 9: Managing app releases with rollouts and rollbacks

## Section 9.1: How Kubernetes manages rollouts

### change to the exercise directory:
`cd ch09`

### deploy a simple web app:
`kubectl apply -f vweb/`
```
service/vweb created
deployment.apps/vweb created
```

### check the ReplicaSets:
`kubectl get rs -l app=vweb`
```
NAME              DESIRED   CURRENT   READY   AGE
vweb-6bd7464487   2         2         0       0s
```

### now increase the scale:
`kubectl apply -f vweb/update/vweb-v1-scale.yaml`
```
deployment.apps/vweb configured
```

### check the ReplicaSets:
`kubectl get rs -l app=vweb`
```
NAME              DESIRED   CURRENT   READY   AGE
vweb-6bd7464487   3         3         0       0s
```

### check the deployment history:
`kubectl rollout history deploy/vweb`
```
deployment.apps/vweb 
REVISION  CHANGE-CAUSE
1         <none>
```
This would have been interesting to run in previous chapters!

### update the image for the web app:
`kubectl set image deployment/vweb web=kiamol/ch09-vweb:v2`
```
deployment.apps/vweb image updated
```

### check the ReplicaSets again:
`kubectl get rs -l app=vweb`
```
NAME              DESIRED   CURRENT   READY   AGE
vweb-6bd7464487   0         0         0       2m
vweb-7c675fdd96   3         3         3       9s
```

### check the rollouts:
`kubectl rollout history deploy/vweb`
```
deployment.apps/vweb 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```
Hopefully they will talk about change-causes.

## Section 9.2: Updating Deployments with rollouts and rollbacks

### apply the change using the record flag:
`kubectl apply -f vweb/update/vweb-v11.yaml --record`
```
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/vweb configured
```

### check the ReplicaSets and their labels:
`kubectl get rs -l app=vweb --show-labels`
```
NAME              DESIRED   CURRENT   READY   AGE     LABELS
vweb-6bd7464487   0         0         0       4m17s   app=vweb,pod-template-hash=6bd7464487,version=v1
vweb-7c675fdd96   0         0         0       2m26s   app=vweb,pod-template-hash=7c675fdd96,version=v1
vweb-7f67d94964   3         3         3       10s     app=vweb,pod-template-hash=7f67d94964,version=v1.1
```

### check the current rollout status:
`kubectl rollout status deploy/vweb`
```
deployment "vweb" successfully rolled out
```

### check the rollout history:
`kubectl rollout history deploy/vweb`
```
deployment.apps/vweb 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         kubectl apply --filename=vweb/update/vweb-v11.yaml --record=true
```

### show the rollout revision for the ReplicaSets:
`kubectl get rs -l app=vweb -o=custom-columns=NAME:.metadata.name,REPLICAS:.status.replicas,REVISION:.metadata.annotations.deployment\.kubernetes\.io/revision`
```
NAME              REPLICAS   REVISION
vweb-6bd7464487   0          <none>
vweb-7c675fdd96   0          <none>
vweb-7f67d94964   3          <none>
```

### we’ll use the app URL a lot, so save it to a local file:
`kubectl get svc vweb -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8090/v.txt' > url.txt`

### then use the contents of the file to make an HTTP request:
`curl $(cat url.txt)`
```bash
paulkaefer ~/GitHub/kiamol/ch09 λ curl $(cat url.txt)
v1
Mon Nov 13 11:04:52
~/GitHub/kiamol/ch09
paulkaefer ~/GitHub/kiamol/ch09 λ cat url.txt 
http://localhost:8090/v.txt
```

### deploy the v2 update:
`kubectl apply -f vweb/update/vweb-v2.yaml --record`
```
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/vweb configured
```

### check the response again:
`curl $(cat url.txt)`
```
v2
```

### check the ReplicaSet details:
`kubectl get rs -l app=vweb --show-labels`
```
NAME              DESIRED   CURRENT   READY   AGE     LABELS
vweb-675f546687   3         3         3       22s     app=vweb,pod-template-hash=675f546687,version=v2
vweb-6bd7464487   0         0         0       12m     app=vweb,pod-template-hash=6bd7464487,version=v1
vweb-7c675fdd96   0         0         0       10m     app=vweb,pod-template-hash=7c675fdd96,version=v1
vweb-7f67d94964   0         0         0       7m58s   app=vweb,pod-template-hash=7f67d94964,version=v1.1
```

### look at the revisions:
`kubectl rollout history deploy/vweb`
Hah -- Ran this just a minute ago because I was curious!
```
λ kubectl rollout history deploy/vweb
deployment.apps/vweb 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         kubectl apply --filename=vweb/update/vweb-v11.yaml --record=true
4         kubectl apply --filename=vweb/update/vweb-v2.yaml --record=true
```

### list ReplicaSets with their revisions:
`kubectl get rs -l app=vweb -o=custom-columns=NAME:.metadata.name,REPLICAS:.status.replicas,VERSION:.metadata.labels.version,REVISION:.metadata.annotations.deployment\.kubernetes\.io/revision`
```
NAME              REPLICAS   VERSION   REVISION
vweb-675f546687   3          v2        <none>
vweb-6bd7464487   0          v1        <none>
vweb-7c675fdd96   0          v1        <none>
vweb-7f67d94964   0          v1.1      <none>
```

### see what would happen with a rollback:
`kubectl rollout undo deploy/vweb --dry-run`
```
W1113 11:08:23.984322    5303 helpers.go:692] --dry-run is deprecated and can be replaced with --dry-run=client.
deployment.apps/vweb Pod Template:
  Labels: app=vweb
  pod-template-hash=7f67d94964
  version=v1.1
  Containers:
   web:
    Image:  kiamol/ch09-vweb:v1
    Port: 80/TCP
    Host Port:  0/TCP
    Environment:  <none>
    Mounts: <none>
  Volumes:  <none>
 (dry run)
```

### then start a rollback to revision 2:
`kubectl rollout undo deploy/vweb --to-revision=2`
```
deployment.apps/vweb rolled back
```

### check the app--this should surprise you:
`curl $(cat url.txt)`
```
v2
```
It does surprise me.


### remove the existing app:
`kubectl delete deploy vweb`
```
deployment.apps "vweb" deleted
```

### deploy a new version that stores content in config:
`kubectl apply -f vweb/update/vweb-v3-with-configMap.yaml --record`
```
Flag --record has been deprecated, --record will be removed in the future
configmap/vweb-config created
deployment.apps/vweb created
```

### check the response:
`curl $(cat url.txt)`
```
v3-from-config
```

### update the ConfigMap, and wait for the change to propagate:
`kubectl apply -f vweb/update/vweb-configMap-v31.yaml --record`
```
Flag --record has been deprecated, --record will be removed in the future
configmap/vweb-config configured
```
`sleep 120`

### check the app again:
`curl $(cat url.txt)`
```
v3.1
```

### check the rollout history:
`kubectl rollout history deploy/vweb`
```
deployment.apps/vweb 
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=vweb/update/vweb-v3-with-configMap.yaml --record=true
```

Deploy a new version of the app with an immutable config, so you can compare the release process:
### remove the old Deployment:
`kubectl delete deploy vweb`
```
deployment.apps "vweb" deleted
```

### create a new Deployment using an immutable config:
`kubectl apply -f vweb/update/vweb-v4-with-configMap.yaml --record`
```
Flag --record has been deprecated, --record will be removed in the future
configmap/vweb-config-v4 created
deployment.apps/vweb created
```

### check the output:
`curl $(cat url.txt)`
```
v4-from-config
```

### release a new ConfigMap and updated Deployment:
`kubectl apply -f vweb/update/vweb-v41-with-configMap.yaml --record`
```
Flag --record has been deprecated, --record will be removed in the future
configmap/vweb-config-v41 created
deployment.apps/vweb configured
```

### check the output again:
`curl $(cat url.txt)`
```
v4.1-from-config
```

### the update is a full rollout:
`kubectl rollout history deploy/vweb`
```
deployment.apps/vweb 
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=vweb/update/vweb-v4-with-configMap.yaml --record=true
2         kubectl apply --filename=vweb/update/vweb-v41-with-configMap.yaml --record=true
```
This is helpful to compare, too:
```
λ diff vweb/update/vweb-v4-with-configMap.yaml vweb/update/vweb-v41-with-configMap.yaml
4c4
<   name: vweb-config-v4
---
>   name: vweb-config-v41
9c9
<     v4-from-config
---
>     v4.1-from-config
26c26
<         version: v4
---
>         version: v4.1
41c41
<             name: vweb-config-v4
---
>             name: vweb-config-v41
```

### so you can rollback:
`kubectl rollout undo deploy/vweb`
curl $(cat url.txt)
```
deployment.apps/vweb rolled back
v4-from-config
````

## Section 9.3: Configuring rolling updates for Deployments

### delete the existing app:
`kubectl delete deploy vweb`
```
deployment.apps "vweb" deleted
```

### deploy with the Recreate strategy:
`kubectl apply -f vweb-strategies/vweb-recreate-v2.yaml`
```
deployment.apps/vweb created
```

### check the ReplicaSets:
`kubectl get rs -l app=vweb`
```
NAME              DESIRED   CURRENT   READY   AGE
vweb-675f546687   3         3         3       7s
```

### test the app:
`curl $(cat url.txt)`
```
v2
```

### look at the details of the Deployment:
`kubectl describe deploy vweb`
```
Name:               vweb
Namespace:          default
CreationTimestamp:  Mon, 13 Nov 2023 11:24:24 -0600
Labels:             kiamol=ch09
Annotations:        deployment.kubernetes.io/revision: 1
Selector:           app=vweb
Replicas:           3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:       Recreate
MinReadySeconds:    0
Pod Template:
  Labels:  app=vweb
           version=v2
  Containers:
   web:
    Image:        kiamol/ch09-vweb:v2
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   vweb-675f546687 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set vweb-675f546687 to 3
```

### deploy the updated Pod spec:
`kubectl apply -f vweb-strategies/vweb-recreate-v3.yaml`
```
deployment.apps/vweb configured
```

### check the status, with a time limit for updates:
`kubectl rollout status deploy/vweb --timeout=2s`
```
Waiting for deployment "vweb" rollout to finish: 0 of 3 updated replicas are available...
error: timed out waiting for the condition
```

### check the ReplicaSets:
`kubectl get rs -l app=vweb`
```
NAME              DESIRED   CURRENT   READY   AGE
vweb-675f546687   0         0         0       2m24s
vweb-7cbbbdf58c   3         3         0       23s
```

### check the Pods:
`kubectl get pods -l app=vweb`
```
NAME                    READY   STATUS   RESTARTS      AGE
vweb-7cbbbdf58c-c6lkc   0/1     Error    2 (23s ago)   30s
vweb-7cbbbdf58c-cv5lp   0/1     Error    2 (22s ago)   30s
vweb-7cbbbdf58c-g7tzv   0/1     Error    2 (25s ago)   30s
```

### test the app-this will fail:
`curl $(cat url.txt)`
```
curl: (52) Empty reply from server
```
I don't think I've seen that `curl` error before!

### update the Deployment to use rolling updates and the v2 image:
`kubectl apply -f vweb-strategies/vweb-rollingUpdate-v2.yaml`
```
deployment.apps/vweb configured
```

### check the Pods for the app:
`kubectl get po -l app=vweb`
```
NAME                    READY   STATUS    RESTARTS   AGE
vweb-675f546687-dt2bv   1/1     Running   0          7s
vweb-675f546687-hjskd   1/1     Running   0          6s
vweb-675f546687-hsrb2   1/1     Running   0          5s
```

### check the rollout status:
`kubectl rollout status deploy/vweb`
```
deployment "vweb" successfully rolled out
```

### check the ReplicaSets:
`kubectl get rs -l app=vweb`
```
NAME              DESIRED   CURRENT   READY   AGE
vweb-675f546687   3         3         3       10m
vweb-7cbbbdf58c   0         0         0       8m26s
```

### test the app:
`curl $(cat url.txt)`
```
v2
```
I wasn't fast enough to catch what the author illustrates in Figure 9.12, but I get it.


### update to the failing container image:
`kubectl apply -f vweb-strategies/vweb-rollingUpdate-v3.yaml`
```
deployment.apps/vweb configured
```

### check the Pods:
`kubectl get po -l app=vweb`
```
Mon Nov 13 11:37:04
NAME                    READY   STATUS    RESTARTS     AGE
vweb-675f546687-hjskd   1/1     Running   0            2m42s
vweb-675f546687-hsrb2   1/1     Running   0            2m41s
vweb-7cbbbdf58c-gclnr   0/1     Error     1 (6s ago)   7s
vweb-7cbbbdf58c-lbcqg   0/1     Error     1 (6s ago)   7s

Mon Nov 13 11:37:11
~/GitHub/kiamol/ch09
paulkaefer ~/GitHub/kiamol/ch09 λ kubectl get po -l app=vweb
NAME                    READY   STATUS    RESTARTS      AGE
vweb-675f546687-hjskd   1/1     Running   0             2m49s
vweb-675f546687-hsrb2   1/1     Running   0             2m48s
vweb-7cbbbdf58c-gclnr   0/1     Error     1 (13s ago)   14s
vweb-7cbbbdf58c-lbcqg   0/1     Error     2 (13s ago)   14s

(re-ran the command a few times)

Mon Nov 13 11:37:28
~/GitHub/kiamol/ch09
paulkaefer ~/GitHub/kiamol/ch09 λ kubectl get po -l app=vweb
NAME                    READY   STATUS    RESTARTS      AGE
vweb-675f546687-hjskd   1/1     Running   0             3m
vweb-675f546687-hsrb2   1/1     Running   0             2m59s
vweb-7cbbbdf58c-gclnr   0/1     Error     2 (24s ago)   25s
vweb-7cbbbdf58c-lbcqg   0/1     Error     2 (24s ago)   25s

Mon Nov 13 11:37:29
~/GitHub/kiamol/ch09
paulkaefer ~/GitHub/kiamol/ch09 λ kubectl get po -l app=vweb
NAME                    READY   STATUS             RESTARTS      AGE
vweb-675f546687-hjskd   1/1     Running            0             3m4s
vweb-675f546687-hsrb2   1/1     Running            0             3m3s
vweb-7cbbbdf58c-gclnr   0/1     CrashLoopBackOff   2 (14s ago)   29s
vweb-7cbbbdf58c-lbcqg   0/1     CrashLoopBackOff   2 (16s ago)   29s

λ kubectl get po -l app=vweb
NAME                    READY   STATUS             RESTARTS      AGE
vweb-675f546687-hjskd   1/1     Running            0             3m44s
vweb-675f546687-hsrb2   1/1     Running            0             3m43s
vweb-7cbbbdf58c-gclnr   0/1     CrashLoopBackOff   3 (29s ago)   69s
vweb-7cbbbdf58c-lbcqg   0/1     CrashLoopBackOff   3 (30s ago)   69s

Mon Nov 13 11:38:13

```

### check the rollout:
`kubectl rollout status deploy/vweb`
```
Waiting for deployment "vweb" rollout to finish: 2 out of 3 new replicas have been updated...
error: deployment "vweb" exceeded its progress deadline
```

### see the scale in the ReplicaSets:
`kubectl get rs -l app=vweb`
```
NAME              DESIRED   CURRENT   READY   AGE
vweb-675f546687   2         2         2       15m
vweb-7cbbbdf58c   2         2         0       13m
```

### test the app:
`curl $(cat url.txt)`
```
v2
```

Ran this again:
```
Mon Nov 13 11:39:55
~/GitHub/kiamol/ch09
paulkaefer ~/GitHub/kiamol/ch09 λ kubectl get po -l app=vweb
NAME                    READY   STATUS             RESTARTS      AGE
vweb-675f546687-hjskd   1/1     Running            0             5m37s
vweb-675f546687-hsrb2   1/1     Running            0             5m36s
vweb-7cbbbdf58c-gclnr   0/1     CrashLoopBackOff   5 (17s ago)   3m2s
vweb-7cbbbdf58c-lbcqg   0/1     Error              5 (91s ago)   3m2s
```

## Section 9.4: Rolling updates in DaemonSets and StatefulSets

### remove all of this chapter’s apps:
`kubectl delete all -l kiamol=ch09`
```
service "vweb" deleted
deployment.apps "vweb" deleted
```

### deploy the to-do app, database, and proxy:
`kubectl apply -f todo-list/db/ -f todo-list/web/ -f todo-list/proxy/`
```
configmap/todo-db-config created
configmap/todo-db-env created
configmap/todo-db-scripts created
secret/todo-db-secret created
service/todo-db created
statefulset.apps/todo-db created
configmap/todo-web-config created
secret/todo-web-secret created
service/todo-web created
deployment.apps/todo-web created
configmap/todo-proxy-configmap created
service/todo-proxy created
daemonset.apps/todo-proxy created
```

### get the app URL:
`kubectl get svc todo-proxy -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8091'`
```
http://localhost:8091
```

### browse to the app, add an item, and check that it’s in the list
Still doesn't work for me.

# deploy the DaemonSet update:
kubectl apply -f todo-list/proxy/update/nginx-rollingUpdate.yaml

# watch the Pod update:
kubectl get po -l app=todo-proxy --watch
```
NAME               READY   STATUS        RESTARTS   AGE
todo-proxy-qcrr6   1/1     Terminating   0          66s
todo-proxy-qcrr6   0/1     Terminating   0          66s
todo-proxy-bjtfd   0/1     Pending       0          0s
todo-proxy-bjtfd   0/1     Pending       0          0s
todo-proxy-bjtfd   0/1     ContainerCreating   0          0s
todo-proxy-qcrr6   0/1     Terminating         0          67s
todo-proxy-qcrr6   0/1     Terminating         0          67s
todo-proxy-bjtfd   1/1     Running             0          6s
```

# Press ctrl-c when the update completes
Ah cool. Ctrl-c'd and tried again:
```
λ kubectl get po -l app=todo-proxy --watch
NAME               READY   STATUS    RESTARTS   AGE
todo-proxy-bjtfd   1/1     Running   0          44s
```
This does seem super useful & interesting for logging.

### deploy the update:
`kubectl apply -f todo-list/db/update/todo-db-rollingUpdate-partition.yaml`
```
statefulset.apps/todo-db configured
```

### check the rollout status:
`kubectl rollout status statefulset/todo-db`
```
Waiting for partitioned roll out to finish: 0 out of 1 new pods have been updated...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...
partitioned roll out complete: 1 new pods have been updated...
```

### list the Pods, showing the image name and start time:
`kubectl get pods -l app=todo-db -o=custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[0].image,START_TIME:.status.startTime`
```
NAME        IMAGE                  START_TIME
todo-db-0   postgres:11.6-alpine   2023-11-13T17:51:01Z
todo-db-1   postgres:11.8-alpine   2023-11-13T18:05:37Z
```

### switch the web app to read-only mode, so it uses the secondary database:
`kubectl apply -f todo-list/web/update/todo-web-readonly.yaml`
```
deployment.apps/todo-web configured
```

### test the app--the data is there, but now it’s read-only
This app still doesn't let me add to the list, so I see an empty list.


### apply the update:
`kubectl apply -f todo-list/db/update/todo-db-rollingUpdate.yaml`
```
statefulset.apps/todo-db configured
```

### check its progress:
`kubectl rollout status statefulset/todo-db`
```
waiting for statefulset rolling update to complete 1 pods at revision todo-db-d77d4cdc5...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...
Waiting for 1 pods to be ready...
statefulset rolling update complete 2 pods at revision todo-db-d77d4cdc5...
```

### Pods should now all have the same spec:
`kubectl get pods -l app=todo-db -o=custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[0].image,START_TIME:.status.startTime`
```
NAME        IMAGE                  START_TIME
todo-db-0   postgres:11.8-alpine   2023-11-13T18:09:02Z
todo-db-1   postgres:11.8-alpine   2023-11-13T18:05:37Z
```

### reset the web app back to read-write mode:
`kubectl apply -f todo-list/web/todo-web.yaml`
```
deployment.apps/todo-web configured
```

### test that the app works and is connected to the updated primary Pod
Adding items still doesn't work for me.
The diagnostics page:
```
Hostname  .NET Version  OS Architecture OS Description
todo-web-5d57859767-xzlns .NET 6.0.8  Arm64 Linux 6.4.16-linuxkit #1 SMP PREEMPT Wed Oct 25 16:32:24 UTC 2023
```

## Section 9.5: Understanding release strategies

### Cleanup
```
λ kubectl get all
NAME                            READY   STATUS      RESTARTS        AGE
pod/disk-calc-szgxt             0/1     Completed   0               4d3h
pod/nginx                       1/1     Running     1 (2d21h ago)   4d3h
pod/nginx-stateful-0            1/1     Running     1 (2d21h ago)   4d3h
pod/nginx-stateful-1            1/1     Running     1 (2d21h ago)   4d3h
pod/nginx-stateful-2            1/1     Running     1 (2d21h ago)   4d3h
pod/todo-db-0                   1/1     Running     0               4m31s
pod/todo-db-1                   1/1     Running     0               7m56s
pod/todo-proxy-bjtfd            1/1     Running     0               21m
pod/todo-web-5d57859767-xzlns   1/1     Running     0               3m57s

NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          4d22h
service/nginx        LoadBalancer   10.104.234.22   localhost     8088:30782/TCP   4d3h
service/todo-db      ClusterIP      None            <none>        5432/TCP         22m
service/todo-proxy   LoadBalancer   10.101.26.125   localhost     8091:30942/TCP   22m
service/todo-web     ClusterIP      10.108.94.221   <none>        80/TCP           22m

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/todo-proxy   1         1         1       1            1           <none>          22m

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/todo-web   1/1     1            1           22m

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/todo-web-5d57859767   1         1         1       22m
replicaset.apps/todo-web-7ffcbf8ff9   0         0         0       7m10s

NAME                              READY   AGE
statefulset.apps/nginx-stateful   3/3     4d3h
statefulset.apps/todo-db          2/2     22m

NAME                  COMPLETIONS   DURATION   AGE
job.batch/disk-calc   1/1           4s         4d3h

λ kubectl delete all -l kiamol=ch09
service "todo-db" deleted
service "todo-proxy" deleted
service "todo-web" deleted
daemonset.apps "todo-proxy" deleted
deployment.apps "todo-web" deleted
statefulset.apps "todo-db" deleted

λ kubectl delete cm -l kiamol=ch09
configmap "todo-db-config" deleted
configmap "todo-db-env" deleted
configmap "todo-db-scripts" deleted
configmap "todo-proxy-configmap" deleted
configmap "todo-web-config" deleted
configmap "vweb-config" deleted
configmap "vweb-config-v4" deleted
configmap "vweb-config-v41" deleted

λ kubectl delete pvc -l kiamol=ch09
persistentvolumeclaim "data-todo-db-0" deleted
persistentvolumeclaim "data-todo-db-1" deleted

λ kubectl get all
NAME                   READY   STATUS      RESTARTS        AGE
pod/disk-calc-szgxt    0/1     Completed   0               4d3h
pod/nginx              1/1     Running     1 (2d21h ago)   4d3h
pod/nginx-stateful-0   1/1     Running     1 (2d21h ago)   4d3h
pod/nginx-stateful-1   1/1     Running     1 (2d21h ago)   4d3h
pod/nginx-stateful-2   1/1     Running     1 (2d21h ago)   4d3h

NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          4d22h
service/nginx        LoadBalancer   10.104.234.22   localhost     8088:30782/TCP   4d3h

NAME                              READY   AGE
statefulset.apps/nginx-stateful   3/3     4d3h

NAME                  COMPLETIONS   DURATION   AGE
job.batch/disk-calc   1/1           4s         4d3h
```
I believe all that's left relates to that dashboard I deployed, from the kubernetes documentation.


## Chapter 9 lab
```
λ kubectl apply -f lab/v1/
service/vweb created
deployment.apps/vweb-v1 created

λ kubectl get svc vweb -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8090'
http://localhost:8090
```
> v1
with "fresh air" background

```
λ kubectl apply -f lab/solution/
service/vweb configured
deployment.apps/vweb-v2 created
```

Now I see v2, with magic mint background.

Switching between the two works, but yeah, Chrome caches the page, so Incognito mode helps~
```
# for v1
kubectl apply -f lab/v1/vweb-service-v1.yaml

# for v2
kubectl apply -f lab/solution/vweb-service-v2.yaml
```

### Teardown
```
λ kubectl delete all -l kiamol=ch09-lab
service "vweb" deleted
deployment.apps "vweb-v1" deleted
deployment.apps "vweb-v2" deleted
```


# Chapter 10: Packaging and managing apps with Helm
## Section 10.1: What Helm adds to Kubernetes

Installing `helm`:
```bash
λ brew install helm
...
λ helm version
version.BuildInfo{Version:"v3.13.2", GitCommit:"2a2fb3b98829f1e0be6fb18af2f6599e0f4e8243", GitTreeState:"clean", GoVersion:"go1.21.4"}
λ helm repo add kiamol https://kiamol.net
"kiamol" has been added to your repositories
λ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "kiamol" chart repository
...Successfully got an update from the "airflow-stable" chart repository
Update Complete. ⎈Happy Helming!⎈
λ helm search repo vweb --versions
NAME        CHART VERSION APP VERSION DESCRIPTION             
kiamol/vweb 2.0.0         2.0.0       Simple versioned web app
kiamol/vweb 1.0.0         1.0.0       Simple versioned web app
```

### inspect the default values stored in the chart:
`helm show values kiamol/vweb --version 1.0.0`
```
servicePort: 8090
replicaCount: 2
```

### install the chart, overriding the default values:
`helm install --set servicePort=8010 --set replicaCount=1 ch10-vweb kiamol/vweb --version 1.0.0`
```
NAME: ch10-vweb
LAST DEPLOYED: Wed Nov 15 08:43:54 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### check the releases you have installed:
`helm ls`
```
NAME      NAMESPACE REVISION  UPDATED                               STATUS    CHART       APP VERSION
ch10-vweb default   1         2023-11-15 08:43:54.226961 -0600 CST  deployed  vweb-1.0.0  1.0.0  
```

### show the details of the Deployment:
`kubectl get deploy -l app.kubernetes.io/instance=ch10-vweb --show-labels`
```
NAME        READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
ch10-vweb   1/1     1            1           87s   app.kubernetes.io/instance=ch10-vweb,app.kubernetes.io/managed-by=Helm,app.kubernetes.io/name=vweb,kiamol=ch10
```

### update the release to increase the replica count:
`helm upgrade --set servicePort=8010 --set replicaCount=3 ch10-vweb kiamol/vweb --version 1.0.0`
```
Release "ch10-vweb" has been upgraded. Happy Helming!
NAME: ch10-vweb
LAST DEPLOYED: Wed Nov 15 08:45:32 2023
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
```

### check the ReplicaSet:
`kubectl get rs -l app.kubernetes.io/instance=ch10-vweb`
```
NAME                   DESIRED   CURRENT   READY   AGE
ch10-vweb-5f87499974   3         3         3       108s
```

### get the URL for the app:
`kubectl get svc ch10-vweb -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8010'`
```
http://localhost:8010
```

### browse to the app URL
```html
<!DOCTYPE html>
<html>
    <head>
        <title>v1</title>
        <style>
            body {
                background-color: #a8ebff;
                font-family: "Lucida Console", Courier, monospace;
            }
            h1 {
                font-weight: 800;
                font-size: 80px;
                color: #020c66;
            }
        </style>
    </head>
    <body>
        <h1>v1</h1>
    </body>
</html>
```

## Section 10.2: Packaging your own apps with Helm

### switch to this chapter’s source:
`cd ch10`

### validate the chart contents:
`helm lint web-ping`
```
==> Linting web-ping
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

### install a release from the chart folder:
`helm install wp1 web-ping/`
```
NAME: wp1
LAST DEPLOYED: Wed Nov 15 08:51:14 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### check the installed releases:
`helm ls`
```
NAME      NAMESPACE REVISION  UPDATED                               STATUS    CHART           APP VERSION
ch10-vweb default   2         2023-11-15 08:45:32.551091 -0600 CST  deployed  vweb-1.0.0      1.0.0      
wp1       default   1         2023-11-15 08:51:14.357695 -0600 CST  deployed  web-ping-0.1.0  1.0.0      
```

### check the available settings for the chart:
`helm show values web-ping/`
```
# targetUrl - URL of the website to ping
targetUrl: blog.sixeyed.com
# httpMethod - HTTP method to use for pings
httpMethod: HEAD
# pingIntervalMilliseconds - interval between pings in ms
pingIntervalMilliseconds: 30000
# chapter where this exercise is used
kiamolChapter: ch10
```

### install a new release named wp2, using a different target:
`helm install --set targetUrl=paulkaefer.com wp2 web-ping/`
```
NAME: wp2
LAST DEPLOYED: Wed Nov 15 08:53:56 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### wait a minute or so for the pings to fire, then check the logs:
`kubectl logs -l app=web-ping --tail 1`
```
** web-ping ** Pinging: paulkaefer.com; method: HEAD; 30000ms intervals

λ helm install --set targetUrl=google.com wp3 web-ping/
NAME: wp3
LAST DEPLOYED: Wed Nov 15 08:55:02 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

```
Getting some certificate errors, but I get the point.

### add the official Helm repository:
`helm repo add stable https://charts.helm.sh/stable`
```
"stable" has been added to your repositories
```

### install ChartMuseum--the repo flag fetches details from

### the repository so you don’t need to update your local cache:
`helm install --set service.type=LoadBalancer --set service.externalPort=8008 --set env.open.DISABLE_API=false repo stable/chartmuseum --version 2.13.0 --wait`
```
NAME: repo
LAST DEPLOYED: Wed Nov 15 08:59:09 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Get the ChartMuseum URL by running:

** Please ensure an external IP is associated to the repo-chartmuseum service before proceeding **
** Watch the status using: kubectl get svc --namespace default -w repo-chartmuseum **

  export SERVICE_IP=$(kubectl get svc --namespace default repo-chartmuseum -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
  echo http://$SERVICE_IP:8008/

OR

  export SERVICE_HOST=$(kubectl get svc --namespace default repo-chartmuseum -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
  echo http://$SERVICE_HOST:8008/
```

### get the URL for your local ChartMuseum app:
`kubectl get svc repo-chartmuseum -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8008'`
```
http://localhost:8008

Welcome to ChartMuseum!
If you see this page, the ChartMuseum web server is successfully installed and working.

For online documentation and support please refer to [the GitHub project.](https://github.com/helm/chartmuseum)
Thank you for using ChartMuseum.
```

### add it as a repository called local:
`helm repo add local $(kubectl get svc repo-chartmuseum -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8008')`
```
"local" has been added to your repositories
```


### package the local chart:
`helm package web-ping`
```
Successfully packaged chart and saved it to: /Users/paulkaefer/GitHub/kiamol/ch10/web-ping-0.1.0.tgz
```

### *on Windows 10* remove the PowerShell alias to use the real curl:
`Remove-Item Alias:curl -ErrorAction Ignore`

### upload the chart zip archive to ChartMuseum:
`curl --data-binary "@web-ping-0.1.0.tgz" $(kubectl get svc repo-chartmuseum -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8008/api/charts')`
```
{"saved":true}
```

### check that ChartMuseum has updated its index:
`curl $(kubectl get svc repo-chartmuseum -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8008/index.yaml')`
```
apiVersion: v1
entries:
  web-ping:
  - apiVersion: v2
    appVersion: 1.0.0
    created: "2023-11-15T15:02:56.846719Z"
    description: A simple web pinger
    digest: 1c2f19f49021e0334fc98684c1f6da0e18bb1667077b02c2b7f1ba29e6587ee2
    name: web-ping
    type: application
    urls:
    - charts/web-ping-0.1.0.tgz
    version: 0.1.0
generated: "2023-11-15T15:03:07Z"
serverInfo: {}
```


### update your repository cache:
`helm repo update`
```
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "airflow-stable" chart repository
...Successfully got an update from the "kiamol" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```

### verify that Helm can find your chart:
`helm search repo web-ping`
```
NAME            CHART VERSION APP VERSION DESCRIPTION        
local/web-ping  0.1.0         1.0.0       A simple web pinger
```

### check the local values file:
`cat web-ping-values.yaml`
```
pingIntervalMilliseconds: 15000
```

### install from the repository using the values file:
Note I tried another version with `wp3` above, so I renamed this:
`helm install -f web-ping-values.yaml wp4 local/web-ping`
```
NAME: wp4
LAST DEPLOYED: Wed Nov 15 09:04:55 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### list all the Pods running the web-ping apps:
`kubectl get pod -l app=web-ping -o custom-columns='NAME:.metadata.name,ENV:.spec.containers[0].env[*].value'`
```
NAME                   ENV
wp1-b7b69b4dd-9m82w    blog.sixeyed.com,HEAD,30000
wp2-7568f9c5d-hhrm8    paulkaefer.com,HEAD,30000
wp3-68946c8856-m2djj   google.com,HEAD,30000
wp4-6495b98586-qfv9z   blog.sixeyed.com,HEAD,15000`
```

## Section 10.3: Modeling dependencies in charts

### install a release from the local chart folder:
`helm install --set upstreamToProxy=ch10-vweb:8010 vweb-proxy proxy/`
```
NAME: vweb-proxy
LAST DEPLOYED: Wed Nov 15 09:09:41 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### get the URL for the new proxy service:
`kubectl get svc vweb-proxy-proxy -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8080'`
```
http://localhost:8080
```

### browse to the URL
Same `v1` app (HTML even) as above.

### build dependencies:
`helm dependency build pi`
```
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "kiamol" chart repository
...Successfully got an update from the "airflow-stable" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 2 charts
Downloading vweb from repo https://kiamol.net
Deleting outdated charts
```

### check that the dependencies have been downloaded:
`ls ./pi/charts`
```
proxy-0.1.0.tgz vweb-2.0.0.tgz
```

### print the YAML Helm would deploy with default values:
`helm install pi1 ./pi --dry-run`
```yaml
NAME: pi1
LAST DEPLOYED: Wed Nov 15 09:15:23 2023
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: pi/templates/web-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: pi1-web
  labels:
    kiamol: ch10
spec:
  ports:
    - port: 80
      name: http
  selector:
    app: pi1
    component: web
  type: LoadBalancer
---
# Source: pi/templates/web.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pi1-web
  labels:
    kiamol: ch10
spec:
  replicas: 2
  selector:
    matchLabels:
      app: pi1
      component: web
  template:
    metadata:
      labels:
        app: pi1
        component: web
    spec:
      containers:
        - image: kiamol/ch05-pi
          command: ["dotnet", "Pi.Web.dll", "-m", "web"]
          name: web
          ports:
            - containerPort: 80
              name: http
```

### install with custom settings to add the proxy:
`helm install --set serviceType=ClusterIP --set proxy.enabled=true pi2 ./pi`
```
NAME: pi2
LAST DEPLOYED: Wed Nov 15 09:15:41 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### get the URL for the proxied app:
`kubectl get svc pi2-proxy -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8030'`
```
http://localhost:8030
```

### browse to it
```
To: 50 d.p.
in: 0 ms.
from: pi2-web-7c7b4fc8d4-8c9jv
3.14159265358979323846264338327950288419716939937510
```

## Section 10.4: Upgrading and rolling back Helm releases

### list all releases:
`helm ls -q`
```
ch10-vweb
pi2
repo
vweb-proxy
wp1
wp2
wp3
wp4
```

### check the values for the new chart version:
`helm show values kiamol/vweb --version 2.0.0`
```
# port for the Service to listen on
servicePort: 8090
# type of the Service:
serviceType: LoadBalancer
# number of replicas for the web Pod
replicaCount: 2
```

### deploy a new release using an internal Service type:
`helm install --set servicePort=8020 --set replicaCount=1 --set serviceType=ClusterIP ch10-vweb-v2 kiamol/vweb --version 2.0.0`
```
NAME: ch10-vweb-v2
LAST DEPLOYED: Wed Nov 15 11:36:11 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### use a port-forward so you can test the app:
`kubectl port-forward svc/ch10-vweb-v2 8020:8020`
```
Forwarding from 127.0.0.1:8020 -> 80
Forwarding from [::1]:8020 -> 80
```

### browse to localhost:8020, then exit the port-forward with Ctrl-C or Cmd-C
```html
<!DOCTYPE html>
<html>
    <head>
        <title>v2</title>
        <style>
            body {
                background-color: #b3ffe6;
                font-family: "Times New Roman", Times, serif;
            }
            h1 {
                font-size: 90px;
                color: #02422d;
            }
        </style>
    </head>
    <body>
        <h1>v2</h1>
    </body>
</html>
```

```
Handling connection for 8020
^C
```

### remove the test release:
`helm uninstall ch10-vweb-v2`
```
release "ch10-vweb-v2" uninstalled
```

### check the values used in the current version 1 release:
`helm get values ch10-vweb`
```
USER-SUPPLIED VALUES:
replicaCount: 3
servicePort: 8010
```

### upgrade to version 2 using the same values--this will fail:
`helm upgrade --reuse-values --atomic ch10-vweb kiamol/vweb --version 2.0.0`
```
Release "ch10-vweb" has been upgraded. Happy Helming!
NAME: ch10-vweb
LAST DEPLOYED: Wed Nov 15 12:07:47 2023
NAMESPACE: default
STATUS: deployed
REVISION: 3
TEST SUITE: None
```
Hmm, it worked for me. Trying again increments the REVISION to 4.

### show the history of the vweb release:
`helm history ch10-vweb`
```
REVISION  UPDATED                   STATUS      CHART       APP VERSION DESCRIPTION     
1         Wed Nov 15 08:43:54 2023  superseded  vweb-1.0.0  1.0.0       Install complete
2         Wed Nov 15 08:45:32 2023  superseded  vweb-1.0.0  1.0.0       Upgrade complete
3         Wed Nov 15 12:07:47 2023  superseded  vweb-2.0.0  2.0.0       Upgrade complete
4         Wed Nov 15 12:10:07 2023  deployed    vweb-2.0.0  2.0.0       Upgrade complete
```

### save the values of the current release to a YAML file:
`helm get values ch10-vweb -o yaml > vweb-values.yaml`
```
λ cat vweb-values.yaml 
replicaCount: 3
servicePort: 8010
```

### upgrade to version 2 using the values file and the atomic flag:
`helm upgrade -f vweb-values.yaml --atomic ch10-vweb kiamol/vweb --version 2.0.0`
```
Release "ch10-vweb" has been upgraded. Happy Helming!
NAME: ch10-vweb
LAST DEPLOYED: Wed Nov 15 12:15:36 2023
NAMESPACE: default
STATUS: deployed
REVISION: 5
TEST SUITE: None
```

### check the Service and ReplicaSet configuration:
`kubectl get svc,rs -l app.kubernetes.io/instance=ch10-vweb`
```
NAME                TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/ch10-vweb   LoadBalancer   10.103.142.148   localhost     8010:30299/TCP   3h32m

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/ch10-vweb-5f87499974   0         0         0       3h32m
replicaset.apps/ch10-vweb-69774b76bf   3         3         3       8m8s
```
`http://localhost:8010/` now has the v2 again.


### confirm the values used in revision 2:
`helm get values ch10-vweb --revision 2`
```
USER-SUPPLIED VALUES:
replicaCount: 3
servicePort: 8010
```
 
### roll back to that revision:
`helm rollback ch10-vweb 2`
```
Rollback was a success! Happy Helming!
```
 
### check the latest two revisions:
`helm history ch10-vweb --max 2 -o yaml`
```
- app_version: 2.0.0
  chart: vweb-2.0.0
  description: Upgrade complete
  revision: 5
  status: superseded
  updated: "2023-11-15T12:15:36.344275-06:00"
- app_version: 1.0.0
  chart: vweb-1.0.0
  description: Rollback to 2
  revision: 6
  status: deployed
  updated: "2023-11-15T12:17:31.631021-06:00"
```

## Section 10.5: Understanding where Helm fits in

### uninstall all the releases:
`helm uninstall $(helm ls -q)`
```
release "ch10-vweb" uninstalled
release "pi2" uninstalled
release "repo" uninstalled
release "vweb-proxy" uninstalled
release "wp1" uninstalled
release "wp2" uninstalled
release "wp3" uninstalled
release "wp4" uninstalled
```

## Chapter 10 lab
```bash
λ kubectl apply -f ./todo-list/
configmap/todo-web-config created
deployment.apps/todo-web created
service/todo-web created

λ kubectl get svc todo-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8080'
http://localhost:8080
```
Does not load...

```bash
λ helm lint ./ch10-lab-solution
==> Linting ./ch10-lab-solution
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed

λ helm install lab-test ./ch10-lab-solution
NAME: lab-test
LAST DEPLOYED: Wed Nov 15 12:20:33 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
browsing to localhost:8080:
```
This page isn’t working
localhost is currently unable to handle this request.
HTTP ERROR 500
```
`/config` does return `Status Code: 404; Not Found`.
(actually a bunch of spaces afterwards -- 500, to be exact!)

```bash
λ helm install -f dev-values.yaml lab-dev ./ch10-lab-solution
NAME: lab-dev
LAST DEPLOYED: Wed Nov 15 12:23:12 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
Hmm, `/config` now has an nginx/1.17.10 404 page.

### Lab teardown
```bash
λ helm uninstall lab-test lab-dev
release "lab-test" uninstalled
release "lab-dev" uninstalled

λ kubectl delete all -l kiamol=ch10-lab
service "todo-web" deleted
deployment.apps "todo-web" deleted
```


# Chapter 11: App development-Developer workflows and CI/CD

## Section 11.1: The Docker developer workflow

### switch to this chapter’s folder in the source code:
`cd ch11`

### build the app:
`docker-compose -f bulletin-board/docker-compose.yml build`
```
[+] Building 5.4s (10/12)                                  docker:desktop-linux
...
 => ERROR [bulletin-board builder 4/4] RUN npm install                     0.8s
------
 > [bulletin-board builder 4/4] RUN npm install:
0.786 npm ERR! code SELF_SIGNED_CERT_IN_CHAIN
0.786 npm ERR! errno SELF_SIGNED_CERT_IN_CHAIN
0.786 npm ERR! request to https://registry.npmjs.org/morgan failed, reason: self signed certificate in certificate chain
0.788 
0.788 npm ERR! A complete log of this run can be found in:
0.788 npm ERR!     /root/.npm/_logs/2023-11-28T14_48_53_327Z-debug.log
------
failed to solve: process "/bin/sh -c npm install" did not complete successfully: exit code: 1
```

### run the app:
`docker-compose -f bulletin-board/docker-compose.yml up -d`
Also fails...

### check the running  containers:
`docker ps`

### browse to the app at http://localhost:8010/

### stop the app in Compose:
`docker-compose -f bulletin-board/docker-compose.yml down`
No output.

### deploy in Kubernetes:
`kubectl apply -f bulletin-board/kubernetes/`
```
deployment.apps/bulletin-board created
service/bulletin-board created
```

### get the new URL:
`kubectl get svc bulletin-board -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8011'`
```
http://localhost:8011
```

### browse
```
This page isn’t working
localhost didn’t send any data.
ERR_EMPTY_RESPONSE
```

Started next code sample (`rm bulletin...`) and it errored out again. I will likely just read the rest of the chapter.

## Section 11.2: The Kubernetes-as-a-Service developer workflow

"**TRY IT NOW** Gogs is a simple but powerful Git server that is published as an image on Docker Hub. It’s a great way to run a private Git server in your organization or to quickly spin up a backup if your online service goes offline. Run Gogs in your cluster to push a local copy of the book’s source code."

### deploy the Git server:
`kubectl apply -f infrastructure/gogs.yaml`
```
service/gogs created
deployment.apps/gogs created
```

### wait for it to spin up:
`kubectl wait --for=condition=ContainersReady pod -l app=gogs`
```
pod/gogs-76d6d4846f-thqjl condition met
```

### add your local Git server to the book’s repository--this grabs the URL from the Service to use as the target:
`git remote add gogs $(kubectl get svc gogs -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:3000/kiamol/kiamol.git')`

### push the code to your server--authenticate with username kiamol and password kiamol:
`git push gogs`
```
Enumerating objects: 3445, done.
Counting objects: 100% (3445/3445), done.
Delta compression using up to 8 threads
Compressing objects: 100% (1560/1560), done.
Username for 'http://localhost:3000': kiamol
Password for 'http://kiamol@localhost:3000': 
Writing objects: 100% (3445/3445), 10.53 MiB | 1.40 MiB/s, done.
Total 3445 (delta 1687), reused 3422 (delta 1676), pack-reused 0
remote: Resolving deltas: 100% (1687/1687), done.
To http://localhost:3000/kiamol/kiamol.git
 * [new branch]      master -> master
```

### find the server URL:
`kubectl get svc gogs -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:3000'`
```
http://localhost:3000
```
I figured, based on the prior output.

### browse and sign in with the same kiamol credentials
![](./ch11/gogs_up_and_running.png)
![](./ch11/gogs_logged_in.png)

Output of these next four commands is reflected in "Figure 11.6 BuildKit running as a container image—building service, without requiring Docker".
### deploy BuildKit:
`kubectl apply -f infrastructure/buildkitd.yaml`
```
service/buildkitd created
deployment.apps/buildkitd created
```

### wait for it to spin up:
`kubectl wait --for=condition=ContainersReady pod -l app=buildkitd`
```
pod/buildkitd-6794789b55-h992m condition met
```

### verify that Git and BuildKit are available:
`kubectl exec deploy/buildkitd -- sh -c 'git version && buildctl --version'`
```
git version 2.24.1
buildctl github.com/moby/buildkit v0.7.1 ddd175c5a2cc24530ea8ff427887c22939ca4289
```

### check that Docker isn’t installed--this command will fail:
`kubectl exec deploy/buildkitd -- sh -c 'docker version'`
```
sh: docker: not found
command terminated with exit code 127
```
I do like when they warn us it will fail!


### connect to a session on the BuildKit Pod:
`kubectl exec -it deploy/buildkitd -- sh`

### clone the source code from your Gogs server:
```bash
cd ~
git clone http://gogs:3000/kiamol/kiamol.git
```

```
λ kubectl exec -it deploy/buildkitd -- sh
/ # cd ~
~ # git clone http://gogs:3000/kiamol/kiamol.git
Cloning into 'kiamol'...
remote: Enumerating objects: 3445, done.
remote: Counting objects: 100% (3445/3445), done.
remote: Compressing objects: 100% (1549/1549), done.
remote: Total 3445 (delta 1687), reused 3445 (delta 1687), pack-reused 0
Receiving objects: 100% (3445/3445), 10.53 MiB | 77.00 MiB/s, done.
Resolving deltas: 100% (1687/1687), done.
~ # 
```

### switch to the app directory:
`cd kiamol/ch11/bulletin-board/`

### build the app using BuildKit; the options tell BuildKit to use Buildpacks instead of a Dockerfile as input and to produce an image as the output:
`buildctl build --frontend=gateway.v0  --opt source=kiamol/buildkit-buildpacks --local context=src --output type=image,name=kiamol/ch11-bulletin-board:buildkit`
Fails...

### leave the session when the build completes
`exit`


## Section 11.3: Isolating workloads with contexts and namespaces

### create a new namespace:
`kubectl create namespace kiamol-ch11-test`
```
namespace/kiamol-ch11-test created
```

### deploy a sleep Pod in the new namespace:
`kubectl apply -f sleep.yaml --namespace kiamol-ch11-test`
```
deployment.apps/sleep created
```

### list sleep Pods--this won’t return anything:
`kubectl get pods -l app=sleep`
```
No resources found in default namespace.
```

### now list the Pods in the namespace:
`kubectl get pods -l app=sleep -n kiamol-ch11-test`
```
NAME                     READY   STATUS    RESTARTS   AGE
sleep-568fb49bb7-fc99b   1/1     Running   0          19s
```


### create the namespace and Deployment:
`kubectl apply -f sleep-uat.yaml`
```
namespace/kiamol-ch11-uat created
deployment.apps/sleep created
```

### list the sleep Deployments in all namespaces:
`kubectl get deploy -l app=sleep --all-namespaces`
```
NAMESPACE          NAME    READY   UP-TO-DATE   AVAILABLE   AGE
kiamol-ch11-test   sleep   1/1     1            1           153m
kiamol-ch11-uat    sleep   1/1     1            1           10s
```

### delete the new UAT namespace:
`kubectl delete namespace kiamol-ch11-uat`
```
namespace "kiamol-ch11-uat" deleted
```

### list Deployments again:
`kubectl get deploy -l app=sleep --all-namespaces`
```
NAMESPACE          NAME    READY   UP-TO-DATE   AVAILABLE   AGE
kiamol-ch11-test   sleep   1/1     1            1           154m
```

### list all contexts:
`kubectl config get-contexts`
```
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         docker-desktop   docker-desktop   docker-desktop   
```

### update the default namespace for the current context:
`kubectl config set-context --current --namespace=kiamol-ch11-test`
```
Context "docker-desktop" modified.
```

### list the Pods in the default namespace:
`kubectl get pods`
```
NAME                     READY   STATUS    RESTARTS   AGE
sleep-568fb49bb7-fc99b   1/1     Running   0          156m

λ kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         docker-desktop   docker-desktop   docker-desktop   kiamol-ch11-test
```

### setting the namespace to blank resets the default:
`kubectl config set-context --current --namespace=`
```
Context "docker-desktop" modified.
```

### printing out the config file shows your cluster connection:
`kubectl config view`
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://kubernetes.docker.internal:6443
  name: docker-desktop
contexts:
- context:
    cluster: docker-desktop
    user: docker-desktop
  name: docker-desktop
current-context: docker-desktop
kind: Config
preferences: {}
users:
- name: docker-desktop
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
- name: iam-root-account@kiamol.us-west-2.eksctl.io
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      - token
      - -i
      - kiamol
      command: aws-iam-authenticator
      env:
      - name: AWS_STS_REGIONAL_ENDPOINTS
        value: regional
      - name: AWS_DEFAULT_REGION
        value: us-west-2
      interactiveMode: IfAvailable
      provideClusterInfo: false
```


## Section 11.4: Continuous delivery in Kubernetes without Docker

Not running since I know I wasn't able to build the Node app earlier. But I might try on a different machine sometime.

## Section 11.5: Evaluating developer workflows on Kubernetes

Teardown:
### uninstall the Helm release from the pipeline:
`helm -n kiamol-ch11-test uninstall bulletin-board`
```
Error: uninstall: Release not loaded: bulletin-board: release: not found
```
...as expected.

### delete the manual deployment:
`kubectl delete all -l app=bulletin-board`
```
pod "bulletin-board-7ccbb48465-b7hd4" deleted
service "bulletin-board" deleted
deployment.apps "bulletin-board" deleted
```

## Chapter 11 lab
Skipping this for now. Let's see if Yong deployed it; I can revisit on a different machine, once I have that setup.


# Chapter 12: Empowering self-healing apps

## Section 12.1 Routing traffic to healthy Pods using readiness probes


### switch to the chapter’s directory:
`cd ch12`

### deploy the random-number API:
kubectl apply -f numbers/
```
service/numbers-api created
deployment.apps/numbers-api created
```

### wait for it to be ready:
`kubectl wait --for=condition=ContainersReady pod -l app=numbers-api`
```
pod/numbers-api-757d849ff-65tjm condition met
pod/numbers-api-757d849ff-xvrgm condition met
```

### check that the Pod is registered as Service endpoints:
`kubectl get endpoints numbers-api`
```
NAME          ENDPOINTS                     AGE
numbers-api   10.1.1.148:80,10.1.1.149:80   21s
```

### save the URL for the API in a text file:
`kubectl get svc numbers-api -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8013' > api-url.txt`
```bash
λ cat api-url.txt
http://localhost:8013
```

### call the API--after returning, this app server is now unhealthy:
`curl "$(cat api-url.txt)/rng"`
```
81
{"message":"Unhealthy!"}
```

### test the health endpoints to check:
`curl "$(cat api-url.txt)/healthz"; curl "$(cat api-url.txt)/healthz"`
```
Ok{"message":"Unhealthy"}
```
Additional attempts gave me `OkOk` and `{"message":"Unhealthy"}{"message":"Unhealthy"}`.

### confirm the Pods used by the service:
`kubectl get endpoints numbers-api`
```
NAME          ENDPOINTS                     AGE
numbers-api   10.1.1.148:80,10.1.1.149:80   88s
```

### check the Service endpoints:
`kubectl get endpoints numbers-api`
```
NAME          ENDPOINTS                     AGE
numbers-api   10.1.1.148:80,10.1.1.149:80   3m48s
```
 
### call the API, triggering it to go unhealthy:
`curl "$(cat api-url.txt)/rng"`
Two calls:
```
14
{"message":"Unhealthy!"}
```

### wait for the readiness probe to fire:
`sleep 10`

### check the endpoints again:
`kubectl get endpoints numbers-api`
```
NAME          ENDPOINTS                     AGE
numbers-api   10.1.1.148:80,10.1.1.149:80   4m22s
```

### check the Pod status:
`kubectl get pods -l app=numbers-api`
```
NAME                          READY   STATUS    RESTARTS   AGE
numbers-api-757d849ff-65tjm   1/1     Running   0          4m33s
numbers-api-757d849ff-xvrgm   1/1     Running   0          4m33s
```

### we could reset the API... but there are no Pods ready to receive traffic so this will fail:
`curl "$(cat api-url.txt)/reset"`
```
Ok
```
`curl "$(cat api-url.txt)/rng"`
```
8
```
Another `sleep 10` results in a successful call, which returns `96`.

## Section 12.2: Restarting unhealthy Pods with liveness probes

### update the Pod spec from listing 12.2:
`kubectl apply -f numbers/update/api-with-readiness-and-liveness.yaml`
```
deployment.apps/numbers-api configured
```

### wait for the new Pods:
`kubectl wait --for=condition=ContainersReady pod -l app=numbers-api,version=v3`
```
pod/numbers-api-7bf6c69dfb-4m4mq condition met
pod/numbers-api-7bf6c69dfb-r58wg condition met
```

### check the Pod status:
`kubectl get pods -l app=numbers-api -o wide`
```
NAME                           READY   STATUS    RESTARTS   AGE   IP           NODE             NOMINATED NODE   READINESS GATES
numbers-api-7bf6c69dfb-4m4mq   1/1     Running   0          19s   10.1.1.150   docker-desktop   <none>           <none>
numbers-api-7bf6c69dfb-r58wg   1/1     Running   0          16s   10.1.1.151   docker-desktop   <none>           <none>
```

### check the Servivce endpoints:
`kubectl get endpoints numbers-api  # two`
```
NAME          ENDPOINTS                     AGE
numbers-api   10.1.1.150:80,10.1.1.151:80   8m39s
```

### cause one application to become unhealthy:
`curl "$(cat api-url.txt)/rng"`
```
81
{"message":"Unhealthy!"}
95
88
{"message":"Unhealthy!"}
```

### wait for the probes to fire, and check the Pods again:
`sleep 20`
`kubectl get pods -l app=numbers-api`
```
NAME                           READY   STATUS    RESTARTS      AGE
numbers-api-7bf6c69dfb-4m4mq   1/1     Running   1 (26s ago)   107s
numbers-api-7bf6c69dfb-r58wg   1/1     Running   2 (14s ago)   104s
```


### deploy the web and database:
`kubectl apply -f todo-list/db/ -f todo-list/web/`
```
secret/todo-db-secret configured
service/todo-db created
deployment.apps/todo-db created
configmap/todo-web-config configured
secret/todo-web-secret configured
service/todo-web created
deployment.apps/todo-web created
```

### wait for the app to be ready:
`kubectl wait --for=condition=ContainersReady pod -l app=todo-web`
```
pod/todo-web-7b8d778988-snnjb condition met
pod/todo-web-7b8d778988-sxdv9 condition met
```

### get the URL for the service:
`kubectl get svc todo-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8081'`
```
http://localhost:8081
```

### browse to the app, and add a new item
Can browse; can't add any items.


### apply the update:
`kubectl apply -f todo-list/db/update/todo-db-bad-command.yaml`
```
deployment.apps/todo-db configured
```

### watch the Pod status changing:
`kubectl get pods -l app=todo-db --watch`
```bash
Thu Dec 07 12:46:29
~/GitHub/kiamol/ch12
paulkaefer ~/GitHub/kiamol/ch12 λ kubectl get pods -l app=todo-db --watch
NAME                       READY   STATUS    RESTARTS   AGE
todo-db-5d4994db89-lzsp6   0/1     Running   0          8s
todo-db-975596846-fvj6q    1/1     Running   0          2m28s
todo-db-5d4994db89-lzsp6   0/1     Running   1 (1s ago)   62s
todo-db-5d4994db89-lzsp6   0/1     Running   2 (0s ago)   2m1s
^C
Thu Dec 07 12:48:34
~/GitHub/kiamol/ch12
paulkaefer ~/GitHub/kiamol/ch12 λ 
```

### refresh the app to check that it still works ctrl-c or cmd-c to exit the Kubectl watch

Now `http://localhost:8081/new` shows me `Status Code: 400; Bad Request`.

## Section 12.3: Deploying upgrades safely with Helm

### remove all existing apps from the chapter:
`kubectl delete all -l kiamol=ch12`
```
service "numbers-api" deleted
service "todo-db" deleted
service "todo-web" deleted
deployment.apps "numbers-api" deleted
deployment.apps "todo-db" deleted
deployment.apps "todo-web" deleted
```

### install the Helm release:
`helm install --atomic todo-list todo-list/helm/v1/todo-list/`
```
NAME: todo-list
LAST DEPLOYED: Thu Dec  7 12:55:19 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Installed Kiamol to-do list 0.1.0. This is how to get the URL:
 $ kubectl get svc todo-list-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8012'
```

```
http://localhost:8012
```
 
### browse to the app, and add a new item
Still unable.


# Chapter 13: Centralizing logs with Fluentd and Elasticsearch

## Section 13.1: How Kubernetes stores log entries

### switch to the chapter’s folder:
`cd ch13`

### deploy the timecheck app in development and test namespaces:
`kubectl apply -f timecheck/`
```
namespace/kiamol-ch13-dev created
deployment.apps/timecheck created
namespace/kiamol-ch13-test created
configmap/timecheck-config created
deployment.apps/timecheck created
```

### wait for the development namespace to spin up:
`kubectl wait --for=condition=ContainersReady pod -l app=timecheck -n kiamol-ch13-dev`
```
pod/timecheck-7589456574-bcjxl condition met
```

### check the logs:
`kubectl logs -l app=timecheck --all-containers -n kiamol-ch13-dev --tail 1`
```
2023-12-14 17:23:18.014 +00:00 [INF] Environment: DEV; version: 1.0; time check: 17:23.17
```

### wait for the test namespace to spin up:
`kubectl wait --for=condition=ContainersReady pod -l app=timecheck -n kiamol-ch13-test`
```
pod/timecheck-76bc55f9c5-66lh4 condition met
pod/timecheck-76bc55f9c5-pcgnj condition met
```

### check those logs:
`kubectl logs -l app=timecheck --all-containers -n kiamol-ch13-test --tail 1`
```
2023-12-14 17:23:35.114 +00:00 [INF] Environment: TEST; version: 1.1; time check: 17:23.35
2023-12-14 17:23:35.636 +00:00 [INF] Environment: TEST; version: 1.1; time check: 17:23.35
```

### run the Deployment:
`kubectl apply -f sleep.yaml`
```
deployment.apps/sleep created
```

### connect to a session in the Pod container:
`kubectl exec -it deploy/sleep -- sh`

### browse to the host log mount:
`cd /var/log/containers/`

### list the timecheck log files:
`ls timecheck*kiamol-ch13*_logger*`

### view the contents of the dev log file:
`cat $(ls timecheck*kiamol-ch13-dev_logger*) | tail -n 1`

```bash
/ # cd /var/log/containers/
/var/log/containers # ls timecheck*kiamol-ch13*_logger*
timecheck-7589456574-bcjxl_kiamol-ch13-dev_logger-0809b18330c364c441f6b8b1c3cb0875f6e9a61b19404e9f932bfe36806faf8f.log
timecheck-76bc55f9c5-66lh4_kiamol-ch13-test_logger-5b9631a231eb427ca135b4ff92447e1782f660d8e02a2bc935073199ebe38e33.log
timecheck-76bc55f9c5-pcgnj_kiamol-ch13-test_logger-8ee3c1212b31728d9b8e9e3c14b92f517b9214e5d759cdc3bbfc73e9dd99b034.log
/var/log/containers # cat $(ls timecheck*kiamol-ch13-dev_logger*) | tail -n 1
{"log":"2023-12-14 17:25:17.990 +00:00 [INF] Environment: DEV; version: 1.0; time check: 17:25.17\n","stream":"stdout","time":"2023-12-14T17:25:18.674626084Z"}
/var/log/containers # exit
```

### exit from the session:
`exit`


### connect to a session in the Pod:
`kubectl exec -it deploy/sleep -- sh`

### browse to the host path volume:
`cd /var/log/containers/`

### the network proxy runs on every node:
`cat $(ls kube-proxy*) | tail -n 1`

### if your cluster uses Core DNS, you’ll see logs here:
`cat $(ls coredns*) | tail -n 1`

### if your node is running the API server, you’ll see these logs:
`cat $(ls kube-apiserver*) | tail -n 1`

### leave the session:
`exit`

```bash
/ # cd /var/log/containers/
/var/log/containers # cat $(ls kube-proxy*) | tail -n 1
{"log":"I1214 17:22:56.194561       1 shared_informer.go:318] Caches are synced for node config\n","stream":"stderr","time":"2023-12-14T17:22:56.194779254Z"}
/var/log/containers # cat $(ls coredns*) | tail -n 1
{"log":"[INFO] plugin/ready: Still waiting on: \"kubernetes\"\n","stream":"stdout","time":"2023-12-14T17:23:18.035776125Z"}
/var/log/containers # cat $(ls kube-apiserver*) | tail -n 1
{"log":"I1214 17:22:58.298751       1 controller.go:624] quota admission added evaluator for: endpoints\n","stream":"stderr","time":"2023-12-14T17:22:58.300372588Z"}
/var/log/containers # exit
```

## Section 13.2: Collecting logs from nodes with Fluentd

### deploy the DaemonSet and ConfigMap:
`kubectl apply -f fluentbit/`
```
namespace/kiamol-ch13-logging created
configmap/fluent-bit-config created
daemonset.apps/fluent-bit created
serviceaccount/fluent-bit created
clusterrole.rbac.authorization.k8s.io/fluent-bit created
error: resource mapping not found for name: "fluent-bit" namespace: "" from "fluentbit/fluentbit.yaml": no matches for kind "ClusterRoleBinding" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first
```

### wait for Fluent Bit to start up:
`kubectl wait --for=condition=ContainersReady pod -l app=fluent-bit -n kiamol-ch13-logging`
```
pod/fluent-bit-hwk86 condition met
```

### check the logs of the Fluent Bit Pod:
`kubectl logs  -l app=fluent-bit -n kiamol-ch13-logging --tail 2`
```
{"date":1702575206.262565,"log":"2023-12-14 17:33:25.632 +00:00 [INF] Environment: TEST; version: 1.1; time check: 17:33.25\n","stream":"stdout","time":"2023-12-14T17:33:26.262565046Z"}
{"date":1702575208.170758,"log":"2023-12-14 17:33:27.985 +00:00 [INF] Environment: DEV; version: 1.0; time check: 17:33.27\n","stream":"stdout","time":"2023-12-14T17:33:28.170757963Z"}
```

### update the data pipeline configuration files:
`kubectl apply -f fluentbit/update/fluentbit-config-match.yaml`
```
configmap/fluent-bit-config configured
```

### restart the DaemonSet so a new Pod gets the changed configuration:
`kubectl rollout restart ds/fluent-bit -n kiamol-ch13-logging`
```
daemonset.apps/fluent-bit restarted
```

### wait for the new logging Pod:
`kubectl wait --for=condition=ContainersReady pod -l app=fluent-bit -n kiamol-ch13-logging`
```
pod/fluent-bit-tqxhl condition met
```

### print the last log entry:
`kubectl logs  -l app=fluent-bit -n kiamol-ch13-logging --tail 1`
```
{"date":1702575515.125852,"log":"2023-12-14 17:38:35.101 +00:00 [INF] Environment: TEST; version: 1.1; time check: 17:38.35\n","stream":"stdout","time":"2023-12-14T17:38:35.1258523Z"}
```


### update the configuration and restart Fluent Bit:
`kubectl apply -f fluentbit/update/fluentbit-config-match-multiple.yaml`
```
configmap/fluent-bit-config configured
```

`kubectl rollout restart ds/fluent-bit -n kiamol-ch13-logging`
```
daemonset.apps/fluent-bit restarted
```

`kubectl wait --for=condition=ContainersReady pod -l app=fluent-bit -n kiamol-ch13-logging`
```
pod/fluent-bit-xv27c condition met
```
 
### print the last two log lines:
`kubectl logs  -l app=fluent-bit -n kiamol-ch13-logging --tail 2`
```
1702575610.166681,2 (total = 2)
{"date":1702575605.63383,"log":"2023-12-14 17:40:05.627 +00:00 [INF] Environment: TEST; version: 1.1; time check: 17:40.05\n","stream":"stdout","time":"2023-12-14T17:40:05.6338303Z"}
```

Ran again:
```
1702575615.165694,1 (total = 3)
{"date":1702575615.642952,"log":"2023-12-14 17:40:15.626 +00:00 [INF] Environment: TEST; version: 1.1; time check: 17:40.15\n","stream":"stdout","time":"2023-12-14T17:40:15.642951763Z"}
```

...and again!
```
1702575635.164616,1 (total = 7)
{"date":1702575635.661916,"log":"2023-12-14 17:40:35.625 +00:00 [INF] Environment: TEST; version: 1.1; time check: 17:40.35\n","stream":"stdout","time":"2023-12-14T17:40:35.661915967Z"}
```

## Section 13.3: Shipping logs to Elasticsearch

### create the Elasticsearch deployment, and wait for the Pod:
`kubectl apply -f elasticsearch/`
```
service/elasticsearch created
deployment.apps/elasticsearch created
```

`kubectl wait --for=condition=ContainersReady pod -l app=elasticsearch -n kiamol-ch13-logging`
```
pod/elasticsearch-6f45f77487-6484p condition met
```

### create the Kibana deployment, and wait for it to start:
`kubectl apply -f kibana/`
```
service/kibana created
deployment.apps/kibana created
```

`kubectl wait --for=condition=ContainersReady pod -l app=kibana -n kiamol-ch13-logging`
```
pod/kibana-844b4c58d4-xpjjv condition met
```

### get the URL for Kibana:
`kubectl get svc kibana -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:5601' -n kiamol-ch13-logging`
```
http://localhost:5601
```


### deploy the updated configuration from listing 13.3
`kubectl apply -f fluentbit/update/fluentbit-config-elasticsearch.yaml`
```
configmap/fluent-bit-config configured
```

### update Fluent Bit, and wait for it to restart
`kubectl rollout restart ds/fluent-bit -n kiamol-ch13-logging`
```
daemonset.apps/fluent-bit restarted
```

`kubectl wait --for=condition=ContainersReady pod -l app=fluent-bit -n kiamol-ch13-logging`
```
pod/fluent-bit-dxjf2 condition met
```

### now browse to Kibana and set up the search:
* click Discover on the left navigation panel 
* create a new index pattern
* enter "test" as the index pattern
* in the next step, select @timestamp as the time filter field
* click Create Index Pattern 
* click Discover again on the left navigation panel to see the logs
See two screenshots!

### deploy the API and proxy:
`kubectl apply -f numbers/`
```
service/numbers-api created
deployment.apps/numbers-api created
service/numbers-api-proxy created
configmap/numbers-api-proxy-config created
deployment.apps/numbers-api-proxy created
```
 
### wait for the app to start up:
`kubectl wait --for=condition=ContainersReady pod -l app=numbers-api -n kiamol-ch13-test`
```
pod/numbers-api-66dbcbc888-96gjm condition met
```
 
### get the URL to use the API via the proxy:
`kubectl get svc numbers-api-proxy -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8080/rng' -n kiamol-ch13-test`
```
http://localhost:8080/rng
```
 
### browse to the API, wait 30 seconds, and refresh until you get an error
### browse to Kibana, and enter this query in the search bar:
### kubernetes.labels.app:numbers-api AND log:<failure-ID-from-the-API>
In browser:
```
{"message":"Unhealthy! Failure ID: 358d0430-b810-4b9d-b5c5-7169a531a077"}
```

Didn't see results with `kubernetes.labels.app:numbers-api AND log:358d0430-b810-4b9d-b5c5-7169a531a077`, but if I just search that error ID or the next one I get (`ea783aad-7ff6-480c-b3df-b22a71ad1ad0`), I see results.

## Section 13.4: Parsing and filtering log entries

# Chapter 14: didn't run through exercises
# Chapter 15: didn't run through exercises

# Chapter 16: Securing applications with policies, contexts, and admission control

## Section 16.1: Securing communication with network policies

### switch to the chapter folder:
`cd ch16`

### deploy the APOD app:
`kubectl apply -f apod/`
```
service/apod-api created
deployment.apps/apod-api created
service/apod-log created
deployment.apps/apod-log created
service/apod-web created
deployment.apps/apod-web created
```

### wait for it to be ready:
`kubectl wait --for=condition=ContainersReady pod -l app=apod-web`
Took maybe five seconds or so?
```
pod/apod-web-66c5b9d8c4-dwbxs condition met
```

### browse to the Service on port 8016 if you want to see today's picture
No image...s
```
Image Gallery

©
```
 
### now run a sleep Pod:
`kubectl apply -f sleep.yaml`
```
deployment.apps/sleep configured
```
 
### confirm that the sleep Pod can use the API:
`kubectl exec  deploy/sleep -- curl -s http://apod-api/image`
```
{"timestamp":"2024-01-11T14:37:39.139+00:00","status":500,"error":"Internal Server Error","message":"","path":"/image"}
```
 
### read the metrics from the access log:
`kubectl exec deploy/sleep -- sh -c 'curl -s http://apod-log/metrics | head -n 2'`
```
# HELP access_log_total Access Log - total log requests
# TYPE access_log_total counter
```

### create the policy:
`kubectl apply -f apod/update/networkpolicy-api.yaml`
```
networkpolicy.networking.k8s.io/apod-api created
```

### confirm it is there:
`kubectl get networkpolicy`
```
NAME       POD-SELECTOR   AGE
apod-api   app=apod-api   11s
```

### try to access the API from the sleep Pod--this is not permitted  by the policy:
`kubectl exec deploy/sleep -- curl http://apod-api/image`
```
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current                                 Dload  Upload   Total   Spent    Left  Speed
0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0{"timestamp":"2024-01-11T14:40:06.832+00:00","status":500,"error":"Internal Server100   119    0   119    0     0    329      0 --:--:-- --:--:-- --:--:--   330
```


### install the Kind command line using instructions at https://kind.sigs.k8s.io/docs/user/quick-start/
Done.

### create a new cluster with a custom Kind configuration:
`kind create cluster --image kindest/node:v1.18.4 --name kiamol-ch16 --config kind/kind-calico.yaml`
```
ERROR: failed to create cluster: unknown apiVersion: kind.sigs.k8s.io/v1alpha3
```

### install the Calico network plugin:
`kubectl apply -f kind/calico.yaml`

### wait for Calico to spin up:
`kubectl wait --for=condition=ContainersReady pod -l k8s-app=calico-node -n kube-system`

### confirm your new cluster is ready:
`kubectl get nodes`

...

`kind delete cluster --name kiamol-ch16`
```
Deleting cluster "kiamol-ch16" ...

λ kubectl get nodes
NAME             STATUS   ROLES           AGE   VERSION
docker-desktop   Ready    control-plane   63d   v1.28.2
```

## Section 16.2: Restricting container capabilities with security contexts

### deploy the app:
`kubectl apply -f pi/`
```
deployment.apps/pi-web created
service/pi-web created
```

### wait for the container to start:
`kubectl wait --for=condition=ContainersReady pod -l app=pi-web`
```
pod/pi-web-66544ccb68-mlcbr condition met
```

### print the name of the user in the Pod container:
`kubectl exec deploy/pi-web -- whoami`
```
root
```

### try to access the Kubernetes API server:
`kubectl exec deploy/pi-web -- sh -c 'curl -k -s https://kubernetes.default | grep message'`
```
sh: curl: not found
command terminated with exit code 1
```

### print the API access token:
`kubectl exec deploy/pi-web -- cat /run/secrets/kubernetes.io/serviceaccount/token`
```
eyJhbGc...AYAt84ZDj9w
```


### add the nonroot SecurityContext:
`kubectl apply -f pi/update/deployment-podsecuritycontext.yaml`
```
deployment.apps/pi-web configured
```

### wait for the new Pod:
`kubectl wait --for=condition=ContainersReady pod -l app=pi-web`
```
pod/pi-web-86f86c9fb8-kcfbf condition met
```

### confirm the user:
`kubectl exec deploy/pi-web -- whoami`
```
nobody
```

### list the API token files:
`kubectl exec deploy/pi-web -- ls -l /run/secrets/kubernetes.io/serviceaccount/token`
```
lrwxrwxrwx    1 root     root            12 Jan 11 14:52 /run/secrets/kubernetes.io/serviceaccount/token -> ..data/token
```

### print out the access token:
`kubectl exec deploy/pi-web -- cat /run/secrets/kubernetes.io/serviceaccount/token`
```
eyJhbGciO...TolcEC3E6g
```

### update to the Pod spec in listing 16.3:
`kubectl apply -f pi/update/deployment-no-serviceaccount-token.yaml`
```
deployment.apps/pi-web configured
```

### confirm the API token doesn’t exist:
`kubectl exec deploy/pi-web -- cat /run/secrets/kubernetes.io/serviceaccount/token`
```
cat: can't open '/run/secrets/kubernetes.io/serviceaccount/token': No such file or directory
command terminated with exit code 1
```

### confirm that the API server is still accessible:
`kubectl exec deploy/pi-web -- sh -c 'curl -k -s https://kubernetes.default | grep message'`
```
sh: curl: not found
command terminated with exit code 1
```

### get the URL, and check that the app still works:
`kubectl get svc pi-web -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8031'`
```
http://localhost:8031
```
```
Pi.Web
Source
Pi
To: 6 d.p.
in: 16 ms.
from: pi-web-846c676747-86tg9
3.141592
```
Appending `?dp=50` to the URL returns `3.14159265358979323846264338327950288419716939937510`.

## Section 16.3: Blocking and modifying workloads with webhooks

### run the Pod to generate a certificate:
`kubectl apply -f ./cert-generator.yaml`
```
deployment.apps/cert-generator created
```

### when the container is ready, the certificate is done:
`kubectl wait --for=condition=ContainersReady pod -l app=cert-generator`
```
pod/cert-generator-54b58488d7-w8x5j condition met
```

### the Pod has deployed the cert as a TLS Secret:
`kubectl get secret -l kiamol=ch16`
```
No resources found in default namespace.
```

### deploy the webhook server, using the TLS Secret:
`kubectl apply -f admission-webhook/`
```
service/admission-webhook created
deployment.apps/admission-webhook created
```

### print the CA certificate:
`kubectl exec -it deploy/cert-generator -- cat ca.base64`
```
LS0tLS1CRUdJ...klDQVRFLS0tLS0K
```

### install the configuration object:
`helm install validating-webhook admission-webhook/helm/validating-webhook/ --set caBundle=$(kubectl exec -it deploy/cert-generator -- cat ca.base64)`
```
Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: resource mapping not found for name: "servicetokenpolicy" namespace: "" from "": no matches for kind "ValidatingWebhookConfiguration" in version "admissionregistration.k8s.io/v1beta1"
ensure CRDs are installed first
```

### confirm it’s been created:
`kubectl get validatingwebhookconfiguration`
```
No resources found
```

### try to deploy an app:
`kubectl apply -f vweb/v1.yaml`
```
service/vweb-v1 created
deployment.apps/vweb-v1 created
```

### check the webhook logs:
`kubectl logs -l app=admission-webhook --tail 3`
```
Error from server (BadRequest): container "admission-webhook" in pod "admission-webhook-c7b7769c7-h9r8b" is waiting to start: ContainerCreating
```

### show the ReplicaSet status for the app:
`kubectl get rs -l app=vweb-v1`
```
NAME                 DESIRED   CURRENT   READY   AGE
vweb-v1-66c5db6dd8   1         1         1       28s
```

### show the details:
`kubectl describe rs -l app=vweb-v1`
```
Name:           vweb-v1-66c5db6dd8
Namespace:      default
Selector:       app=vweb-v1,pod-template-hash=66c5db6dd8
Labels:         app=vweb-v1
                pod-template-hash=66c5db6dd8
Annotations:    deployment.kubernetes.io/desired-replicas: 1
                deployment.kubernetes.io/max-replicas: 2
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/vweb-v1
Replicas:       1 current / 1 desired
Pods Status:    1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=vweb-v1
           pod-template-hash=66c5db6dd8
  Containers:
   web:
    Image:        kiamol/ch09-vweb:v1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  35s   replicaset-controller  Created pod: vweb-v1-66c5db6dd8-ms97t
```

### deploy the webhook configuration:
`helm install mutating-webhook admission-webhook/helm/mutating-webhook/ --set caBundle=$(kubectl exec -it deploy/cert-generator -- cat ca.base64)`
```
Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: resource mapping not found for name: "nonrootpolicy" namespace: "" from "": no matches for kind "MutatingWebhookConfiguration" in version "admissionregistration.k8s.io/v1beta1"
ensure CRDs are installed first
```

### confirm it’s been created:
`kubectl get mutatingwebhookconfiguration`
Skipping...

### deploy a new web app:
`kubectl apply -f vweb/v2.yaml`

### print the webhook server logs:
`kubectl logs -l app=admission-webhook --tail 5`

### show the status of the ReplicaSet:
`kubectl get rs -l app=vweb-v2`

### show the details:
`kubectl describe pod -l app=vweb-v2`

## Section 16.4: Controlling admission with Open Policy Agent


# Chapter 17: Securing resources with role-based access control

## Section 17.1: How Kubernetes secures access to resources

### switch to this chapter’s folder:
`cd ch17`

### PowerShell doesn't have a grep command; run this on Windows to add it:
`. .\grep.ps1`
Code:
```PowerShell
Set-Alias -Name grep -Value Select-String
```

### check that the API versions include RBAC:
`kubectl api-versions | grep rbac`
```
rbac.authorization.k8s.io/v1
```

### show the admin cluster roles:
`kubectl get clusterroles | grep admin`
```
admin                                                                  2023-11-08T19:29:56Z
cluster-admin                                                          2023-11-08T19:29:56Z
system:aggregate-to-admin                                              2023-11-08T19:29:56Z
system:kubelet-api-admin                                               2023-11-08T19:29:56Z
```

### show the details of the cluster admin:
`kubectl describe clusterrole cluster-admin`
```
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  *.*        []                 []              [*]
             [*]                []              [*]
```

### run the certificate generator: 
`kubectl apply -f user-cert-generator.yaml`
```
serviceaccount/user-cert-generator created
clusterrole.rbac.authorization.k8s.io/create-approve-csr created
clusterrolebinding.rbac.authorization.k8s.io/user-cert-generator created
pod/user-cert-generator created
```

### wait for the container to start:
`kubectl wait --for=condition=ContainersReady pod user-cert-generator`
```
pod/user-cert-generator condition met
```

### print the logs:
`kubectl logs user-cert-generator --tail 3`
```
----------------
Cert generated: /certs/user.key and /certs/user.crt
----------------
```

### copy the files onto your local disk:
`kubectl cp user-cert-generator:/certs/user.key user.key`
`kubectl cp user-cert-generator:/certs/user.crt user.crt`
```bash
tar: removing leading '/' from member names
tar: removing leading '/' from member names
λ cat user.crt
-----BEGIN CERTIFICATE-----
MIIDQj...xDW7X65sIQ==
-----END CERTIFICATE-----
λ cat user.key 
-----BEGIN RSA PRIVATE KEY-----
MIIEpgI...p6xmBDJ
-----END RSA PRIVATE KEY-----
λ ls -al
...
-rw-r--r--   1 paulkaefer  staff  1192 Jan 23 09:14 user.crt
-rw-r--r--   1 paulkaefer  staff  1679 Jan 23 09:14 user.key
```
Figure 17.3: "The certificate is a standard X.509 certificate, issued by Kubernetes."

### set the credentials for a new context from the client certificate:
`kubectl config set-credentials reader --client-key=./user.key --client-certificate=./user.crt --embed-certs=true`
```
User "reader" set.
```

### set the cluster for the context:
`kubectl config set-context reader --user=reader --cluster $(kubectl config view -o jsonpath='{.clusters[0].name}')`
```
Context "reader" created.
```

### try to deploy a Pod using the new context-- if your cluster is configured with authentication, this won't work, as the provider tries to authenticate:
`kubectl apply -f sleep/ --context reader`
```
Error from server (Forbidden): error when retrieving current configuration of:
Resource: "apps/v1, Resource=deployments", GroupVersionKind: "apps/v1, Kind=Deployment"
Name: "sleep", Namespace: "default"
from server for: "sleep/sleep.yaml": deployments.apps "sleep" is forbidden: User "reader@kiamol.net" cannot get resource "deployments" in API group "apps" in the namespace "default"
```

### impersonate the user to confirm their permissions:
`kubectl get pods --as reader@kiamol.net`
```
Error from server (Forbidden): pods is forbidden: User "reader@kiamol.net" cannot list resource "pods" in API group "" in the namespace "default"
```
Phew; same errors as in the book.


### deploy a sleep Pod as your normal user:
`kubectl apply -f sleep/`
```
deployment.apps/sleep configured
```

### deploy the role binding so the reader can view the Pod:
`kubectl apply -f role-bindings/reader-view-default.yaml`
```
rolebinding.rbac.authorization.k8s.io/reader-view created
```

### confirm the user sees Pods in the default namespace:
`kubectl get pods --as reader@kiamol.net`
```
NAME                                READY   STATUS              RESTARTS       AGE
admission-webhook-c7b7769c7-h9r8b   0/1     ContainerCreating   0              12d
apod-api-5df694f6bb-xp6cs           1/1     Running             1 (10d ago)    12d
apod-log-56455d84bf-zmbx8           1/1     Running             1 (21m ago)    12d
apod-web-66c5b9d8c4-dwbxs           1/1     Running             1 (10d ago)    12d
cert-generator-54b58488d7-w8x5j     1/1     Running             1 (10d ago)    12d
nginx-stateful-0                    1/1     Running             6 (10d ago)    55d
nginx-stateful-1                    1/1     Running             6 (10d ago)    55d
nginx-stateful-2                    1/1     Running             6 (10d ago)    55d
pi-web-846c676747-86tg9             1/1     Running             1 (10d ago)    12d
sleep-568fb49bb7-vhgk2              1/1     Running             1 (10d ago)    12d
todo-list-db-c7566bff7-h499l        1/1     Running             10 (10d ago)   46d
todo-list-web-c659dfbc7-4rg2r       1/1     Running             11 (10d ago)   46d
todo-list-web-c659dfbc7-hnnj6       1/1     Running             10 (10d ago)   46d
user-cert-generator                 1/1     Running             0              11m
vweb-v1-66c5db6dd8-ms97t            1/1     Running             1 (10d ago)    11d
```
Oops, I'm behind on cleanup!

### confirm the user is blocked from the system namespace:
`kubectl get pods -n kube-system --as reader@kiamol.net`
```
Error from server (Forbidden): pods is forbidden: User "reader@kiamol.net" cannot list resource "pods" in API group "" in the namespace "kube-system"
```

### confirm the user can’t delete Pods--this will fail:
`kubectl delete -f sleep/ --as reader@kiamol.net`
```
Error from server (Forbidden): error when deleting "sleep/sleep.yaml": deployments.apps "sleep" is forbidden: User "reader@kiamol.net" cannot delete resource "deployments" in API group "apps" in the namespace "default"
```

## Section 17.2: Securing resource access within the cluster

### on Docker Desktop for Mac run this to fix the RBAC setup:
`kubectl patch clusterrolebinding docker-for-desktop-binding --type=json --patch $'[{"op":"replace", "path":"/subjects/0/name", "value":"system:serviceaccounts:kube-system"}]'`
```
Error from server (NotFound): clusterrolebindings.rbac.authorization.k8s.io "docker-for-desktop-binding" not found
```
I wonder if this bug was fixed since the book was published...

### OR on Docker Desktop for Windows:
`kubectl patch clusterrolebinding docker-for-desktop-binding --type=json --patch '[{\"op\":\"replace\",\"path\":\"/subjects/0/name\", \"value\":\"system:serviceaccounts:kube-system\"}]'`
 
### create the new namespace:
`kubectl apply -f namespace.yaml`
```
namespace/kiamol-ch17 created
```
 
### list service accounts:
`kubectl get serviceaccounts -n kiamol-ch17`
```
NAME      SECRETS   AGE
default   0         9s
```
 
### check permissions for your own account:
`kubectl auth can-i "*" "*"`
```
yes
```
 
### check permissions for the new service account:
`kubectl auth can-i "*" "*" --as system:serviceaccount:kiamol-ch17:default`
```
no
```
 
`kubectl auth can-i get pods -n kiamol-ch17 --as system:serviceaccount:kiamol-ch17:default`
```
no
```

### run a can-i check inside the certificate generator Pod:
`kubectl exec user-cert-generator -- kubectl auth can-i create csr --all-namespaces`
```
yes
```
 
### use impersonation for the same check:
`kubectl auth can-i create csr -A --as system:serviceaccount:default:user-cert-generator`
```
yes
```
 
### confirm the new service account doesn’t have the permission:
`kubectl auth can-i create csr -A --as system:serviceaccount:kiamol-ch17:default`
```
no
```


### deploy the app and the RBAC rules:
`kubectl apply -f kube-explorer/`
```
serviceaccount/kube-explorer created
deployment.apps/kube-explorer created
role.rbac.authorization.k8s.io/default-pod-reader created
rolebinding.rbac.authorization.k8s.io/kube-explorer-default created
service/kube-explorer created
```
 
### wait for the Pod:
`kubectl wait --for=condition=ContainersReady pod -l app=kube-explorer`
```
pod/kube-explorer-5dd5fc5b97-zmzqb condition met
```
 
### get the URL for the app:
`kubectl get svc kube-explorer -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8019'`
```
http://localhost:8019
```
 
### browse to the app, and confirm you can view and delete Pods in the default namespace; then add ?ns=kube-system to the URL, and you’ll see an error.
Yes; `http://localhost:8019/ServiceAccounts` gives an error though, as does `http://localhost:8019/?ns=kube-system` (as expected).

If I delete, e.g., the pod named `apod-api-5df694f6bb-qj2q7` from Image `kiamol/ch14-image-of-the-day`, I see a new one spun up almost immediately: `apod-api-5df694f6bb-4qbsc`.



### apply the new role and binding
`kubectl apply -f kube-explorer/update/rbac-with-kube-system.yaml`
```
role.rbac.authorization.k8s.io/system-pod-reader created
rolebinding.rbac.authorization.k8s.io/kube-explorer-system created
```
 
### refresh the explorer app with the path /?ns=kube-system. you can see the Pods now, but you can’t delete them.
Correct:
![](./ch17/KubeExplorer.png)


### print the label value on the namespace:
`kubectl get ns kiamol-ch17 --show-labels`
```
NAME          STATUS   AGE   LABELS
kiamol-ch17   Active   12m   kiamol=ch17,kubernetes.io/metadata.name=kiamol-ch17
```
 
### deploy the to-do list app:
`kubectl apply -f todo-list/`
```
serviceaccount/todo-web created
deployment.apps/todo-web created
clusterrole.rbac.authorization.k8s.io/ch17-reader created
clusterrolebinding.rbac.authorization.k8s.io/todo-web-reader created
serviceaccount/todo-web unchanged
deployment.apps/todo-web configured
service/todo-web created
```
 
### wait for the Pod to start:
`kubectl wait --for=condition=ContainersReady pod -l app=todo-web -n kiamol-ch17`
```
pod/todo-web-5ccff57bf7-67xdw condition met
```
 
### print the logs of the init container:
`kubectl logs -l app=todo-web -c configurator --tail 1 -n kiamol-ch17`
```
** Namespace label: ch17. Written to config file **
```
 
### get the URL, and browse to the app:
`kubectl get svc todo-web -n kiamol-ch17 -o jsonpath='http://{.status.loadBalancer.ingress[0].*}:8020'`
```
http://localhost:8020
```
> This page isn’t working

## Section 17.3: Binding roles to groups of users and service accounts

### create two users:
`kubectl apply -f user-groups/`
```
pod/sre-user created
pod/test-user created
```
 
### confirm that the SRE can’t delete Pods:
`kubectl exec sre-user -- kubectl auth can-i delete pods`
```
no
command terminated with exit code 1
```
 
### print the username and group in the certificate:
`kubectl exec sre-user -- sh -c 'openssl x509 -text -noout -in /certs/user.crt | grep Subject:'`
```
        Subject: C = UK, ST = LONDON, L = London, O = sre, CN = sre1
```
 
### confirm the test user can’t read Pod logs:
`kubectl exec test-user -- kubectl auth can-i get pod/logs`
```
no
command terminated with exit code 1
```
 
### print this certificate’s details:
`kubectl exec test-user -- sh -c 'openssl x509 -text -noout -in /certs/user.crt | grep Subject:'`
```
        Subject: C = UK, ST = LONDON, L = London, O = test, CN = tester1
```

### apply the roles and bindings:
`kubectl apply -f user-groups/bindings/`
```
clusterrolebinding.rbac.authorization.k8s.io/sre-view-cluster created
rolebinding.rbac.authorization.k8s.io/sre-edit-ch17 created
clusterrole.rbac.authorization.k8s.io/logs-reader created
clusterrolebinding.rbac.authorization.k8s.io/test-logs-cluster created
```

### confirm the SRE cannot delete in the default namespace:
`kubectl exec sre-user -- kubectl auth can-i delete pods`
```
no
command terminated with exit code 1
```
 
### confirm they can delete in the ch17 namespace:
`kubectl exec sre-user -- kubectl auth can-i delete pods -n kiamol-ch17`
```
yes
```
 
### confirm the tester can’t list Pods:
`kubectl exec test-user -- kubectl get pods`
```
Error from server (Forbidden): pods is forbidden: User "tester1" cannot list resource "pods" in API group "" in the namespace "default"
command terminated with exit code 1
```
 
### confirm the tester can read the logs for a known Pod:
`kubectl exec test-user -- kubectl logs test-user --tail 10`
```
certificatesigningrequest.certificates.k8s.io/tester1 created
certificatesigningrequest.certificates.k8s.io/tester1 approved
** Cert approved.
----------------
Cert generated: /certs/user.key and /certs/user.crt
----------------
User "tester1" set.
Context "tester1" created.
Switched to context "tester1".
** Using context for user: tester1; group: test
```

Faking an authentication system with service accounts, tokens, and namespaces:
### create namespaces, service accounts, and tokens:
`kubectl apply -f user-groups/service-accounts/`
```
namespace/kiamol-authn-sre created
serviceaccount/sre2 created
secret/sre2-sa-token created
namespace/kiamol-authn-test created
serviceaccount/tester2 created
secret/tester2-sa-token created
```

### apply bindings to the groups:
`kubectl apply -f user-groups/service-accounts/role-bindings/`
```
clusterrolebinding.rbac.authorization.k8s.io/sre-sa-view-cluster created
rolebinding.rbac.authorization.k8s.io/sre-sa-edit-ch17 created
clusterrolebinding.rbac.authorization.k8s.io/test-sa-logs-cluster created
```

### confirm the SRE group has view access across the cluster:
`kubectl get clusterrolebinding sre-sa-view-cluster -o custom-columns='ROLE:.roleRef.name,SUBJECT KIND:.subjects[0].kind,SUBJECT NAME:.subjects[0].name'`
```
ROLE   SUBJECT KIND   SUBJECT NAME
view   Group          system:serviceaccounts:kiamol-authn-sre
```

Skipping a bit, but wanted to look at this:
### add a Base64 command if you’re using Windows:
`. .\base64.ps1`
```PowerShell
function Convert-ToBase64 {
  param (
    [parameter(ValueFromPipeline)]
    [string] $text,
    [switch] $d
  )
  if ($d){
    [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($text))
  }
  else {
    [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes($text))
  }
}

Set-Alias base64  Convert-ToBase64
```


## Section 17.4: Discovering and auditing permissions with plugins

### run a Pod with kubectl and Krew already installed:
`kubectl apply -f rbac-tools/`
```
serviceaccount/rbac-tools created
deployment.apps/rbac-tools created
clusterrolebinding.rbac.authorization.k8s.io/rbac-tools-admin created
```

### wait for the Pod:
`kubectl wait --for=condition=ContainersReady pod -l app=rbac-tools`
```
pod/rbac-tools-66864c5489-vwrm2 condition met
```

### connect to the container:
`kubectl exec -it deploy/rbac-tools -- sh`
 
### install the who-can plugin:
`kubectl krew install who-can`
```bash
λ kubectl exec -it deploy/rbac-tools -- sh
/ # kubectl krew install who-can
Updated the local copy of plugin index.
  New plugins available:
    * aks
    * autons
    * browse-pvc
    * clog
    * community-images
    * crane
    * ctr
    * debug-pdb
    * execws
    * foreach
    * gke-policy
    * insider
    * klock
    * kluster-capacity
    * kopilot
    * kubescape
    * mapr-ticket
    * marvin
    * mayastor
    * netscaler
    * node-ssm
    * nodegizmo
    * nodepools
    * oomd
    * permissions
    * plogs
    * shell-ctx
    * subm
    * ttsum
    * unlimited
  Upgrades available for installed plugins:
    * krew v0.4.3 -> v0.4.4
Installing plugin: who-can
Installed plugin: who-can
\
 | Use this plugin:
 |  kubectl who-can
 | Documentation:
 |  https://github.com/aquasecurity/kubectl-who-can
 | Caveats:
 | \
 |  | The plugin requires the rights to list (Cluster)Role and (Cluster)RoleBindings.
 | /
/
WARNING: You installed plugin "who-can" from the krew-index plugin repository.
   These plugins are not audited for security by the Krew maintainers.
   Run them at your own risk.
A newer version of krew is available (v0.4.3 -> v0.4.4).
Run "kubectl krew upgrade" to get the newest version!
/ # kubectl krew upgrade
Updated the local copy of plugin index.
Upgrading plugin: krew
Upgraded plugin: krew
Upgrading plugin: who-can
Skipping plugin who-can, it is already on the newest version
/ # 
```
 
### list who has access to the todo-list ConfigMap:
`kubectl who-can get configmap todo-web-config`
```
ROLEBINDING  NAMESPACE  SUBJECT            TYPE  SA-NAMESPACE
reader-view  default    reader@kiamol.net  User  

CLUSTERROLEBINDING                           SUBJECT                                  TYPE            SA-NAMESPACE
cluster-admin                                system:masters                           Group           
rbac-tools-admin                             rbac-tools                               ServiceAccount  default
sre-sa-view-cluster                          system:serviceaccounts:kiamol-authn-sre  Group           
sre-view-cluster                             sre                                      Group           
storage-provisioner                          storage-provisioner                      ServiceAccount  kube-system
system:controller:generic-garbage-collector  generic-garbage-collector                ServiceAccount  kube-system
system:controller:namespace-controller       namespace-controller                     ServiceAccount  kube-system
system:kube-controller-manager               system:kube-controller-manager           User            
```


### install the plugin:
`kubectl krew install access-matrix`
```bash
λ kubectl exec -it deploy/rbac-tools -- sh
/ # kubectl krew install access-matrix
Updated the local copy of plugin index.
Installing plugin: access-matrix
W0125 15:07:52.967485     103 install.go:164] failed to install plugin "access-matrix": plugin "access-matrix" does not offer installation for this platform
failed to install some plugins: [access-matrix]: plugin "access-matrix" does not offer installation for this platform
```

### print the matrix for Pods:
`kubectl access-matrix for pods -n default`
```
error: unknown command "access-matrix" for "kubectl"
```

### the print the matrix for the to-do list ConfigMap:
`kubectl access-matrix for configmap todo-web-config -n default`
Same error.

### install the plugin:
`kubectl krew install rbac-lookup`
Also failed.

### cleanup
`kubectl delete all,ns,rolebinding,clusterrolebinding,role,clusterrole,serviceaccount -l kiamol=ch17`
```
pod "sre-user" deleted
pod "test-user" deleted
pod "user-cert-generator" deleted
namespace "kiamol-authn-sre" deleted
namespace "kiamol-authn-test" deleted
namespace "kiamol-ch17" deleted
rolebinding.rbac.authorization.k8s.io "kube-explorer-default" deleted
rolebinding.rbac.authorization.k8s.io "reader-view" deleted
clusterrolebinding.rbac.authorization.k8s.io "rbac-tools-admin" deleted
clusterrolebinding.rbac.authorization.k8s.io "sre-sa-view-cluster" deleted
clusterrolebinding.rbac.authorization.k8s.io "sre-view-cluster" deleted
clusterrolebinding.rbac.authorization.k8s.io "test-logs-cluster" deleted
clusterrolebinding.rbac.authorization.k8s.io "test-sa-logs-cluster" deleted
clusterrolebinding.rbac.authorization.k8s.io "todo-web-reader" deleted
clusterrolebinding.rbac.authorization.k8s.io "user-cert-generator" deleted
role.rbac.authorization.k8s.io "default-pod-reader" deleted
clusterrole.rbac.authorization.k8s.io "ch17-reader" deleted
clusterrole.rbac.authorization.k8s.io "create-approve-csr" deleted
clusterrole.rbac.authorization.k8s.io "logs-reader" deleted
serviceaccount "kube-explorer" deleted
serviceaccount "rbac-tools" deleted
serviceaccount "user-cert-generator" deleted
```

Ran cleanup steps & then:
```bash
λ kubectl get all
NAME                   READY   STATUS    RESTARTS   AGE
pod/nginx-stateful-0   1/1     Running   0          17m
pod/nginx-stateful-1   1/1     Running   0          17m
pod/nginx-stateful-2   1/1     Running   0          17m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   17m

NAME                              READY   AGE
statefulset.apps/nginx-stateful   3/3     77d
```

# Chapter 18: Deploying Kubernetes: Multinode and multiarchitecture clusters
## Section 18.1: What’s inside a Kubernetes cluster?

### install Vagrant-browse to https://www.vagrantup.com to download OR on Windows you can use: choco install vagrant OR on macOS you can use: brew cask install vagrant
```
Error: `brew cask` is no longer a `brew` command. Use `brew <command> --cask` instead.
```
...so I ran `brew install --cask vagrant` instead!

### Vagrant packages VM images into "boxes"; download a Linux box--this will ask you to choose a provider, and you should select your VM runtime:
`vagrant box add bento/ubuntu-20.04`
box: URL: https://vagrantcloud.com/api/v2/vagrant/bento/ubuntu-20.04; see also https://vagrantcloud.com/bento/boxes/ubuntu-20.04/versions/202309.09.0/providers/parallels/unknown/vagrant.box
Installed Parallels (trial), too.
Unfortunately, getting:
```
SSL certificate problem: self-signed certificate in certificate chain
```

### if you want to add Windows, download a Windows box:
`vagrant box add kiamol/windows-2019`

On Windows:
```bash
$ vagrant box add bento/ubuntu-20.04
==> box: Loading metadata for box 'bento/ubuntu-20.04'
    box: URL: https://vagrantcloud.com/api/v2/vagrant/bento/ubuntu-20.04
This box can work with multiple providers! The providers that it
can work with are listed below. Please review the list and choose
the provider you will be working with.

1) parallels
2) virtualbox
3) vmware_desktop

Enter your choice: 2
==> box: Adding box 'bento/ubuntu-20.04' (v202309.09.0) for provider: virtualbox
    box: Downloading: https://vagrantcloud.com/bento/boxes/ubuntu-20.04/versions/202309.09.0/providers/virtualbox/unknown/vagrant.box
    box:
==> box: Successfully added box 'bento/ubuntu-20.04' (v202309.09.0) for 'virtualbox'!
```





