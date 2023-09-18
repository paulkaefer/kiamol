cd GitHub

# Section 2.1: How Kubernetes runs and manages containers

## run a Pod with a single container; the restart flag tells Kubernetes
## to create just the Pod and no other resources:
kubectl run hello-kiamol --image=kiamol/ch02-hello-kiamol
## can add --restart=Never if desired; see https://github.com/sixeyed/kiamol/issues/45
 
## wait for the Pod to be ready:
kubectl wait --for=condition=Ready pod hello-kiamol
 
## list all the Pods in the cluster:
kubectl get pods
 
## show detailed information about the Pod:
kubectl describe pod hello-kiamol

## get the basic information about the Pod:
kubectl get pod hello-kiamol
 
## specify custom columns in the output, selecting network details:
kubectl get pod hello-kiamol --output custom-columns=NAME:metadata.name,NODE_IP:status.hostIP,POD_IP:status.podIP 
 
## specify a JSONPath query in the output,
## selecting the ID of the first container in the Pod:
kubectl get pod hello-kiamol -o jsonpath='{.status.containerStatuses[0].containerID}'

## find the Pod’s container:
docker container ls -q --filter label=io.kubernetes.container.name=hello-kiamol
 
## now delete that container:
docker container rm -f $(docker container ls -q --filter label=io.kubernetes.container.name=hello-kiamol)
 
## check the Pod status:
kubectl get pod hello-kiamol
 
## and find the container again:
docker container ls -q --filter label=io.kubernetes.container.name=hello-kiamol

## listen on port 8080 on your machine and send traffic
## to the Pod on port 80:
kubectl port-forward pod/hello-kiamol 8080:80
 
## now browse to http://localhost:8080
 
## when you’re done press ctrl-c to end the port forward

## create a Deployment called "hello-kiamol-2", running the same web app:
kubectl create deployment hello-kiamol-2 --image=kiamol/ch02-hello-kiamol
 
## list all the Pods:
kubectl get pods

## print the labels that the Deployment adds to the Pod:
kubectl get deploy hello-kiamol-2 -o jsonpath='{.spec.template.metadata.labels}'
 
## list the Pods that have that matching label:
kubectl get pods -l app=hello-kiamol-2

## list all Pods, showing the Pod name and labels:
kubectl get pods -o custom-columns=NAME:metadata.name,LABELS:metadata.labels
 
## update the "app" label for the Deployment’s Pod:
kubectl label pods -l app=hello-kiamol-2 --overwrite app=hello-kiamol-x
 
## fetch Pods again:
kubectl get pods -o custom-columns=NAME:metadata.name,LABELS:metadata.labels

# 2.2 Running Pods with controllers


