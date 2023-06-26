# TROUBLESHOOTING

- To see the endpoint associated with a running pod
	`kubectl get endpoints pod-name`
	

- To check nodes with taints and might be missing a matching toleration
	`kubectl get nodes -o json | jq '.items[].spec.taints'`
	
> ...Here we see that node01 is in a Ready state, but that Scheduling is disabled. It appears as though someone has cordon'd (the node has been made unschedulable) that node so that the scheduler wouldn't deploy resources to it. After talking with teammates you find out that someone was testing and forgot to enable scheduling again.

- Uncordon the node to fix the issue.
	`kubectl uncordon node01`
Now, check to see if the pod is still in a pending state.

- You can run below command to remove the taint from master node and then you should be able to deploy your pod on that node
	`kubectl taint nodes node_name node-role.kubernetes.io/master-`


- You can enter into a pod using the exec
	`kubectl exec -it -n kube-system whatevs /bin/sh`
	`kubectl exec -it -c podname bash`
	
### Common Services Problems (such as denying connections)
	- If the Pods is Ready, you should investigate if the Service can distribute traffic to the Pods.
	- You should also examine the connection between the Service and the Ingress.
	- No healthy endpoints (pod maybe going through a recycle or might have run out of resources like CPU, RAM)
	- Label selector incorrect (you could be searching a label that wasn't stated in the manifest)
	- Pods listening on a port different than configured in the service (for example image has been changed)
	- Too many services defined in the cluster
	- DNS for pod is misconfigured
	- Ensure kube-proxy is running on the node
	
### Here's a quick recap on what ports and labels should match:
	- The Service selector should match the label of the Pod
	- The Service targetPort should match the containerPort of the container inside the Pod
	- The Service port can be any number. Multiple Services can use the same port because they have different IP addresses assigned.
	- The service.port of the Ingress should match the port in the Service
	- The name of the Service should match the field service.name in the Ingress

There are four useful commands to troubleshoot Pods:

```
kubectl logs <pod name> is helpful to retrieve the logs of the containers of the Pod.
kubectl describe pod <pod name> is useful to retrieve a list of events associated with the Pod.
kubectl get pod <pod name> is useful to extract the YAML definition of the Pod as stored in Kubernetes.
kubectl exec -ti <pod name> -- bash is useful to run an interactive command within one of the containers of the Pod.
```
Which one should you use?

There isn't a one-size-fits-all. Instead, you should use a combination of them.

### Troubleshooting Pods

1. ImagePullBackOff
This error appears when Kubernetes isn't able to retrieve the image for one of the containers of the Pod.
There are three common culprits:

- The image name is invalid — as an example, you misspelt the name, or the image does not exist.
- You specified a non-existing tag for the image.
- The image that you're trying to retrieve belongs to a private registry, and Kubernetes doesn't have credentials to access it.
- The first two cases can be solved by correcting the image name and tag.

For the last, you should add the credentials to your private registry in a Secret and reference it in your Pods.

2. CrashLoopBackOff
CrashLoopBackOff is a common error in Kubernetes, indicating a pod constantly crashing in an endless loop. If the container can't start, then Kubernetes shows the CrashLoopBackOff message as a status.
Usually, a container can't start when:
- There's an error in the application that prevents it from starting.
- You misconfigured the container.
- The Liveness probe failed too many times.

### CrashLoopBackOff: Common Causes
The CrashLoopBackOff error can be caused by a variety of issues, including:

- Insufficient resources—lack of resources prevents the container from loading
- Locked file—a file was already locked by another container
- Locked database—the database is being used and locked by other pods
- Failed reference—reference to scripts or binaries that are not present on the container
- Setup error—an issue with the init-container setup in Kubernetes
- Config loading error—a server cannot load the configuration file
- Misconfigurations—a general file system misconfiguration
- Connection issues—DNS or kube-DNS is not able to connect to a third-party service
- Deploying failed services—an attempt to deploy services/applications that have already failed (e.g. due to a lack of access to other services)

You should try and retrieve the logs from that container to investigate why it failed.
If you can't see the logs because your container is restarting too quickly, you can use the following command:

`kubectl logs <pod-name> --previous`

Which prints the error messages from the previous container.

3. RunContainerError

The error appears when the container is unable to start.
That's even before the application inside the container starts.

The issue is usually due to misconfiguration such as:
- Mounting a not-existent volume such as ConfigMap or Secrets.
- Mounting a read-only volume as read-write.

You should use `kubectl describe pod <pod-name>` to inspect and analyse the errors.

4. Pods in a Pending state
When you create a Pod, the Pod stays in the Pending state. Why?

Assuming that your scheduler component is running fine, here are the causes:
- The cluster doesn't have enough resources such as CPU and memory to run the Pod.
- The current Namespace has a ResourceQuota object and creating the Pod will make the Namespace go over the quota.
- The Pod is bound to a Pending PersistentVolumeClaim.
Your best option is to inspect the Events section in the kubectl describe command:

`kubectl describe pod <pod name>`

For errors that are created as a result of ResourceQuotas, you can inspect the logs of the cluster with:

`kubectl get events --sort-by=.metadata.creationTimestamp`

5. Pods in a not Ready state

If a Pod is Running but not Ready it means that the Readiness probe is failing.

When the Readiness probe is failing, the Pod isn't attached to the Service, and no traffic is forwarded to that instance.

A failing Readiness probe is an application-specific error, so you should inspect the Events section in kubectl describe to identify the error.

### Troubleshooting Services
If your Pods are Running and Ready, but you're still unable to receive a response from your app, you should check if the Service is configured correctly.

Services are designed to route the traffic to Pods based on their labels. So the first thing that you should check is how many Pods are targeted by the Service.

You can do so by checking the Endpoints in the Service:

`kubectl describe service my-service`

When the Service targets (at least) a Pod. There should be at an endpoint, which is a pair of <ip address:port>.

If the "Endpoints" section is empty, there are two explanations:

- You don't have any Pod running with the correct label (hint: you should check if you are in the right namespace).
- You have a typo in the selector labels of the Service.

If you see a list of endpoints, but still can't access your application, then the targetPort in your service is the likely culprit.



### Kubernetes PVC Errors: Common Causes and Resolution

In general, PVC errors are related to three broad categories:

- PV creation issue — Kubernetes had a problem creating the persistent volume or enabling access to it, even though the underlying storage resources exist.
- PV provisioning issue — Kubernetes could not create the required persistent volume because storage resources were unavailable.
- Changes in specs — Kubernetes had a problem connecting a pod to the required Persistent Volume because of a configuration change in the PV or PVC.
All of these issues can happen at different stages of the PVC lifecycle. We’ll review a few common errors you might encounter:
1. FailedAttachVolume
2. FailedMount
3. CrashLoopBackOff caused by PersistentVolume Claim


FailedAttachVolume and FailedMount are two errors that indicate a pod had a problem mounting a PV. There is a difference between these two errors:

1. FailedAttachVolume — occurs when a volume cannot be detached from a previous node to be mounted on the current one.
2. FailedMount — occurs when a volume cannot be mounted on the required path. If the FailedAttachVolume error occurred, FailedMount will also occur as a result. But it is also possible that the volume is available, but there was a specific issue mounting on the path required.

> If the problem is Failure to Detach:
Use the storage provider’s interface to detach the volume manually. For example, in AWS you can use the following CLI command to detach a volume from a node:

`aws ec2 detach-volume --volume-id [persistent-volume-id] --force`

> If the problem is Failure to Attach or Mount:

The easiest fix is a problem in the mount configuration. Check for a wrong network path or network partitioning issue that is preventing the PV from mounting.
Next, try to force Kubernetes to run the pod on another node. The PV may be able to mount there. Here are a few options for moving the pod:

- Mark a node as unschedulable via the kubectl cordon command.
- Run kubectl delete pod. This will usually cause Kubernetes to run the pod on another node.
- Use node selectors, affinity, or taints, to specify that the pod should schedule on another node.
If you do not have other available nodes, or you tried the above and the problem recurs, try to resolve the problem on the node:

- Reduce the number of disk partitions or add mount points
- Check access mode on the new node


## MinimumReplicasUnavailable
### Deployment does not have minimum availability

This error indicates that the Kubernetes API server is unable to reach the node. This has nothing to do with your containers, pods or even your CNI network. no route to host indicates that either:

The host is unavailable
A network segmentation has occurred
The Kubelet is unable to answer the API server

- Try getting more explanation with:

`kubectl get deployment dashboard -o yaml -n consul`

Imagine that we have a namespace named restricted that only allows for 200 MiB, and our pod requires 50 MiB. The first 4 pods will be successfully created, but the fifth one will fail.
After a few seconds, the Available condition stabilizes to False:

We are asking for at most 0 unavailable replicas and there is 1 unavailable replica (due to the resource quota). Thus, the “minimum availability” inequality does not hold which means the deployment has the condition Available = False:




## 💡What do you do if your Kubernetes cluster runs out of IP addresses?

**networkPlugin cni failed to set up pod "nginx" network: add cmd: failed to assign an IP address to container**

- 📌This is a P1 issue, so it needs a super quick fix first and then a long term solution. The fastest way to resolve this issue is to add more IP’s to the cluster but lets first understand the problem in detail. Our schedular is saying that it has no more IP’s in its pool to provide to the pods. But how does the schedular gets this information ?

- 📌Kubernetes assigns an IP address (the Pod IP) to the virtual network interface in the Pod's network namespace from a range of addresses reserved for Pods on the node. In other words it takes the IP for the pod from the node on which the pod is to be scheduled. So can we say that in our case all the worker nodes have exhausted there IPs ? This is where things get interesting. There might be nodes which have IPs available but they don’t have the resources like CPU and Memory to run the pod. This is easy to conform if you are using a managed cluster like EKS. In the AWS console for EKS you can see the available IPs.

- 📌The quickest solution in this case is to find two nodes, one that has the IP but not the resource (Node A) and other that has resource but no IP (Node B). What you have to do now is to find out the pods which are consuming highest resources in Node A and the ones which are consuming least resource in Node B. Once you identify them, edit their deployments to define node affinity / label selector so that pods of Node A go to Node B and pod from Node B go to Node A. Congratulation you can call yourself a k8s manual schedular, because that is what we are doing and this is the fastest possible solution.

- 📌Longterm solution would be to understand the NI / NC (network interface / network card) of the worker nodes. The IP capacity of your Kubernetes cluster depends upon the VPC CIDR of course and then on the number of network card your instance can hold. Either replace the worker nodes with bigger capacity or add more worker nodes to the cluster. You can also create and use secondary CIDR for your Kubernetes cluster or create a new subnet and then add worker nodes, but these are all time taking solutions.


Kubernetes POD Troubleshooting Tactics Sequence 👇

𝟭. 𝗖𝗵𝗲𝗰𝗸 𝗹𝗼𝗴𝘀:
Use 𝘬𝘶𝘣𝘦𝘤𝘵𝘭 𝘭𝘰𝘨𝘴 <𝘱𝘰𝘥_𝘯𝘢𝘮𝘦>

𝟮. 𝗔𝗻𝗮𝗹𝘆𝘇𝗲 𝗣𝗼𝗱 𝗦𝘁𝗮𝘁𝘂𝘀:
Use 𝘬𝘶𝘣𝘦𝘤𝘵𝘭 𝘨𝘦𝘵 𝘱𝘰𝘥 <𝘱𝘰𝘥_𝘯𝘢𝘮𝘦> and examine status fields

𝟯. 𝗗𝗲𝘀𝗰𝗿𝗶𝗯𝗲 𝗣𝗼𝗱:
Execute 𝘬𝘶𝘣𝘦𝘤𝘵𝘭 𝘥𝘦𝘴𝘤𝘳𝘪𝘣𝘦 𝘱𝘰𝘥 <𝘱𝘰𝘥_𝘯𝘢𝘮𝘦>

𝟰. 𝗩𝗲𝗿𝗶𝗳𝘆 𝗣𝗼𝗱 𝗖𝗼𝗻𝗳𝗶𝗴𝘂𝗿𝗮𝘁𝗶𝗼𝗻:
Review the pod's YAML configuration

𝟱. 𝗖𝗵𝗲𝗰𝗸 𝗘𝘃𝗲𝗻𝘁𝘀:
Run 𝘬𝘶𝘣𝘦𝘤𝘵𝘭 𝘨𝘦𝘵 𝘦𝘷𝘦𝘯𝘵𝘴

𝟲. 𝗩𝗮𝗹𝗶𝗱𝗮𝘁𝗲 𝗖𝗼𝗻𝘁𝗮𝗶𝗻𝗲𝗿 𝗜𝗺𝗮𝗴𝗲𝘀:
Check image availability and version in pod YAML

𝟳. 𝗥𝗲𝘀𝘁𝗮𝗿𝘁 𝗣𝗼𝗱:
𝘬𝘶𝘣𝘦𝘤𝘵𝘭 𝘳𝘰𝘭𝘭𝘰𝘶𝘵 𝘳𝘦𝘴𝘵𝘢𝘳𝘵 𝘥𝘦𝘱𝘭𝘰𝘺𝘮𝘦𝘯𝘵/<𝘥𝘦𝘱𝘭𝘰𝘺𝘮𝘦𝘯𝘵_𝘯𝘢𝘮𝘦>

𝟴. 𝗥𝗲𝘃𝗶𝗲𝘄 𝗦𝗲𝗿𝘃𝗶𝗰𝗲 𝗗𝗲𝗽𝗲𝗻𝗱𝗲𝗻𝗰𝗶𝗲𝘀:
Analyze dependencies in YAML or documentation

𝟵. 𝗖𝗵𝗲𝗰𝗸 𝗡𝗲𝘁𝘄𝗼𝗿𝗸 𝗖𝗼𝗻𝗻𝗲𝗰𝘁𝗶𝘃𝗶𝘁𝘆:
Get a shell to the running container:
𝘬𝘶𝘣𝘦𝘤𝘵𝘭 𝘦𝘹𝘦𝘤 -𝘪𝘵 <𝘱𝘰𝘥_𝘯𝘢𝘮𝘦> -- 𝘴𝘩

ping or curl to test network connectivity:
𝘱𝘪𝘯𝘨 <𝘵𝘢𝘳𝘨𝘦𝘵_𝘩𝘰𝘴𝘵>
𝘤𝘶𝘳𝘭 <𝘵𝘢𝘳𝘨𝘦𝘵_𝘶𝘳𝘭>

𝟭𝟬. 𝗜𝗻𝘀𝗽𝗲𝗰𝘁 𝗥𝗲𝘀𝗼𝘂𝗿𝗰𝗲 𝗨𝘀𝗮𝗴𝗲:
Utilize 𝘬𝘶𝘣𝘦𝘤𝘵𝘭 𝘵𝘰𝘱 𝘱𝘰𝘥 <𝘱𝘰𝘥_𝘯𝘢𝘮𝘦>