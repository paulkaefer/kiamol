## Setup/teardown
See `book_code.md` and `cleanup.md`, respectively.

## Chapter 2
* p. 18, my output for the third command:
```bash
λ kubectl get pod hello-kiamol -o jsonpath='{.status.containerStatuses[0].containerID}'
containerd://9e3b015012a1fe9aacf96a4a88fbdc69a66aa39e9ce47935cdf1c52f9d6c2cf0
```
  * So, how do we get it to run via Docker? One day of use is showing me an AWS bill of USD 1.74; wouldn't Docker Desktop be free?

## Chapter 5
- [ ] mentions write-ahead log (WAL)... learn more!

something really clicked at some point... essentially the part where we created a PVC but no PV, so it is `Pending`. I can see how this might be working behind the scenes when we go to provision a new EC2 instance, for example.
Also how cool it is to decouple the storage from the node/app!
Imagine retaining some volumes for archival purposes, or swapping PVs to change how the app works. A test PV with demo data & then a real/prod version. Or different PVs depending on the person/org? Maybe not swapping on-the-fly, but easy to deploy an app and "plug-in" the custom/specific PV required.

Similarly:
> Now you have a custom storage class that your apps can request in a PVC.
Neat that we could build our own in-house cloud/cloud provisioning service this way...

- [ ] This seems worth looking into:
> having your whole stack defined in Kubernetes manifests is pretty tempting, and some modern database servers are designed to run in a container platform; TiDB and CockroachDB are options worth looking at.

## Chapter 6
- [X] run `kubectl get all` at end/before I cleanup; see what's all there.
      I did `kubectl get all -l kiamol=ch06-lab` as part of the lab, at least. See also results within the chapter.
- [X] ~~once I complete this chapter, revisit page 7;~~ relevant segment below:
![](./attachments/page7.png)

Section 6.3: "The DaemonSet takes its name from the Linux daemon, which is usually a system process that runs constantly as a single instance in the background (the equivalent of a Windows Service in the Windows world)."

Still not fully understanding, but https://en.wikipedia.org/wiki/Kubernetes#DaemonSets helps. See also [image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297984/files/OEBPS/Images/6-18.jpg).

## Chapter 7
* "The Pod is a virtual environment that creates a shared networking and filesystem space for one or more containers."
* "...you can’t have some Linux and some Windows containers in the same Pod (yet)..." --> I wonder if you can use `wsl` in a Windows container, though.
* "Another container in the same Pod can provide a REST API, which reports on what the app container is doing."
* "...you shouldn’t be running different apps in the same Pod."
* I like the idea of an init container with `git` installed that pulls the latest code, and then other pods can run it.
* https://bit.ly/376rBcF maps to https://github.com/sixeyed/kiamol/blob/master/ch07/resources/Sune-Keller-Service-Hotel.pdf
* "Perhaps you’ve heard of the service mesh architecture, using technologies like Linkerd and Istio..." No, I haven't.

## Chapter 8
* "Kubernetes is a dynamic environment, and data-heavy apps typically expect to run in a stable environment."
  * Makes sense. In my work, we certainly have data-heavy processes... I'm excited to read to see if there are options or suggestions.

## Once complete:
- [ ] consider a certification. See page 8 for some info
- [ ] Try *Learn Docker in a Month of Lunches* (per p. xiii)
  - [ ] explore some of the docker images from this book in more detail, like [this one](https://hub.docker.com/r/kiamol/ch03-sleep)

