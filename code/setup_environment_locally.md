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






