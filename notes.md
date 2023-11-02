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
- [X] once I complete this chapter, revisit page 7
![](./attachments/page7.png)

Section 6.3: "The DaemonSet takes its name from the Linux daemon, which is usually a system process that runs constantly as a single instance in the background (the equivalent of a Windows Service in the Windows world)."

Still not fully understanding, but https://en.wikipedia.org/wiki/Kubernetes#DaemonSets helps. See also [image](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617297984/files/OEBPS/Images/6-18.jpg).

## Chapter 7
* "The Pod is a virtual environment that creates a shared networking and filesystem space for one or more containers."

## Once complete:
- [ ] consider a certification. See page 8 for some info
- [ ] Try *Learn Docker in a Month of Lunches* (per p. xiii)

