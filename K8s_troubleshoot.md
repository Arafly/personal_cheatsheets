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

### 3. Troubleshooting Ingress
If you've reached this section, then:

- The Pods are Running and Ready.
- The Service distributes the traffic to the Pod.

But you still can't see a response from your app.

It means that most likely, the Ingress is misconfigured.
Since the Ingress controller is a third-party component in the cluster, there are different debugging techniques depending on the type of Ingress controller.

The Ingress uses the service.name and service.port to connect to the Service.
You should check that those are correctly configured.

You can inspect that the Ingress is correctly configured with:

`kubectl describe ingress my-ingress`

If the Backend column is empty, then there must be an error in the configuration.

If you can see the endpoints in the Backend column, but still can't access the application, the issue is likely to be:

- How you exposed your Ingress to the public internet.
- How you exposed your cluster to the public internet.

You can isolate infrastructure issues from Ingress by connecting to the Ingress Pod directly.

First, retrieve the Pod for your Ingress controller (which could be located in a different namespace):


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



