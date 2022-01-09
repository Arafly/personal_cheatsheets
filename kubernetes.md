1. Bootstrap a yaml file
	`kubectl run redis --image=nginx --dry-run=client -o yaml > pod.yaml`
	`kubectl apply -f pod.yaml`

### Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

`kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml`
(This will automatically use the pod's labels as selectors)

Or

`kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml` (This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)


### Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

`kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml`
(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml`
(This will not use the pods labels as selectors)

> Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the kubectl expose command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

 NodePort uses Layer 4 routing rules and the Linux iptables utility, which limits Layer 7 routing.

### Create an NGINX Pod
	kubectl run nginx --image=nginx

### Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
	kubectl run nginx --image=nginx --dry-run=client -o yaml

## Deployment

### Create a deployment
	kubectl create deployment --image=nginx nginx

### Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
	kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

### Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)
	kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml

### Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment.
	$ kubectl create deployment httpd-frontend --image=httpd:2.4-alpine 
	$ kubectl set image deployment/httpd-frontend httpd=httpd:2.4-alpine
	$ kubectl scale deployment httpd-frontend --replicas=3
	$ kubectl create -f /root/deployment-definition-1.yaml
	
### To apply an update to a deployment
	$ kubectl apply -f /root/deployment-definition-1.yaml 
	
### To rollback a deployment
	kubectl rollout undo deployment/myapp-deployment

## ReplicaSet  

### Create|get|explain|delete ReplicaSet
	kubectl create -f replica.yml
	kubectl get rs
	kubectl explain replicaset | grep VERSION
	kubectl delete replicaset myapp-replicaset

### Scaling ReplicaSet
	kubectl edit replicaset replica.yml
	kubectl scale rs --replica=6 replicaset-name

	
1. Forgot what something does. Use:

	`kubectl explain pod`
	`kubectl explain pod.spec.containers.ports`
	
2. Format your output(json) and pipe it into a readable
   
	```kubectl get po -n kube-system
	kubectl get pods -n kube-system | grep etcd
	kubectl get pods -n kube-system kube-flannel-ds-amd64-xmcnm -ojson | jq .metadata.labels
	kubectl get po -n kube-system kube-flannel-ds-amd64-xmcnm -o yaml | less
	kubectl get pods -n kube-system kube-apiserver-node1 -o custom-columns=NAME:.metadata.name,NS:metadata.namespace
	kubectl get pods -n kube-system -o custom-columns=NAME:.metadata.name,NS:metadata.namespace
	```

3. To see only the configuration information associated with the current context, use the --minify flag.

	`kubectl config --kubeconfig=config-demo view --minify`

### Get all pods with their labels
    kubectl get pods
    kubectl get pods -o wide
	kubectl get pods -o wide -w (-w is for watch)
	kubectl describe pod podname
	kubectl describe pods
	kubectl get po -n kube-system -L k8s-app
	kubectl get po -n kube-system -l k8s-app=coredns
	
	
> TYPICALLY WE DON'T ROUTE TRAFFIC DIRECTLY TO PODS (EVEN THO THEY'VE GOT THEIR OWN IP ADDRESSES WHICH CHANGES OFTEN DURING SCALING). WE DO SO THE THE SERVICES WHICH HAS A VIRTUAL IP AND DNS MAPPING, THESE THEN TAKES THE TRAFFIC AND FORWARD TO THE PODS, REGARDLESS OF THEIR NUMBER OR CHANGED IP ADDRESS.
SERVICE IS A K8S RESOURCE THAT PROVIDES LAYER-4 LOAD BALANCING FOR A GROUP OF PODS, ALSO SERVICE DISCOVERY USING CLSUTER'S INTERNAL DNS
	kubectl get po,svc -o wide
	kubectl create -f service-def.yml
	kubectl get services

    kubectl expose deployment nginx --port=8080 --target-port=80
	kubectl port-forward svc/nginx 8080:80

    kubectl edit service-name
### To see the endpoint associated with a running pod
	kubectl get endpoints service-name || podname

> List of healthy endpoints are maintained by the endpoint-controller (one of control-manager goons)

- Cluster IP:
    - An internal LB for exposing services WITHIN a cluster (default provided by k8s)
- Node Port:
  - An internal LB for exposing services EXTERNAL to the cluster
- Load Balancer:
  - Usually provisioned by the Cloud Provider 

> Controllers are control loops that watch the state of your cluster, then make or request changes where needed. Each controller tries to move the current cluster state closer to the desired state.” Controllers are used to manage state in Kubernetes for many tasks: properly assigning resources, designating persistent storage, and managing cron jobs.


>  Ingress controllers are designed to treat dynamic Layer 7 routing as a first‑class citizen. This means that Ingress controllers provide far more granular control and management with less toil.
> You can easily use an Ingress controller not only to control ingress traffic but also to deliver service‑level performance metrics and as part of a security policy. Ingress controllers have many features of traditional external load balancers, like TLS termination, handling multiple domains and namespaces, and of course, load balancing traffic. Ingress controllers can load balance traffic at the per‑request rather than per‑service level, a more useful view of Layer 7 traffic and a far better way to enforce SLAs. Ingress controllers can also enforce egress rules which permit outgoing traffic from certain pods only to specific external services, or ensure that traffic is mutually encrypted using mTLS.

### Check the built-in service for DNS
    kubectl get svc -n kube-system kube-dns

```
1. Initializes cluster master node:
	kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16

2. Initialize cluster networking:
	kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
	
3. (Optional) Create an nginx deployment:
	kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/application/nginx-app.yaml
```

### To get the running nodes
	kubectl get nodes
	kubectl get nodes --request-timeout 5s

> JUST LIKE DOCKER, KUBELET DOESN'T RUN AS A POD, RUNS AS SYSTEMD. kUBELET invokES CRI (container runtime interface), which is an interface layer between Kubernetes and container runtimes, such as docker, containerd
	`systemctl status kubelet`
	
	
> KUBE-PROXY LIKE KUBELET RUNS IN EVERY WORKKER NODE AND IT WATCHES THE ENDPOINT RESOURCE THAT CONNECTS THE SERVICES WITH THE PODS AND ALSO UPDATES THE IPTABLES ON ITS NODE IN ORDER TO ENSURE THAT TRAFFIC SENT TO THE SERVICES GET TO THE INTENDED IP IN THE POOL. KUBE-PROXY USES IPTABLES BEHIND THE SCENE TO FUNCTION EFFICIENTLY TO MAKE THE CLUSTER IP FUNCTION PROPERLY
	
### Check deployments
    kubectl describe deployment
	kubectl get deploy -n kube-system

### Check the rollout on a deployment
	kubectl rollout status deployment/myapp-deployment
	kubectl rollout history deployment/myapp-deployment
	
5. To get components running as daemonsets
	`kubectl get ds -n kube-system`
	
### To retrieve configmaps
    kubectl get configmap -n kube-system
	kubectl get cm -n kube-system
	
### Get the details of a particular configmap
	kubectl get cm coredns -n kube-system -o yaml
	
## Namespace

> Using namespace allows to segregate rules about traffic flow for each tier in a cluster, thereby eliminating the need for physical seperation and external firewall between the tiers.
> It's common for different tiers of applications to be owned by different teams in an organization, and having them segregated by the namespaces is a clear delineation of responsibilities between those teams and apps.

### Check all the pods in the entire namespace in a system
	`kubectl get pods -n kube-system`

### Get namespaces
	kubectl get ns || kubectl get ns --no-headers
	
### Create a new one
	kubectl create ns namespace-name
	
### Add a pod to the namespace
	kubectl run redis --image=redis -n finance || kubectl pod -ns --image=nginx -n namespace-name
	kubectl get po -n namespace-name || kubectl get po --namespace=research
	
	kubectl get pods --all-namespaces
### Add a deployment to a namespace
	kubectl apply -f whatevs.yml -n namesapce-name
	
### Switch current namspace you're in
	kubectl config set-context $(kubectl config current-context) --namespace=dev

### Get current namspace you're in.
	kubectl config current-context
	kubectl config get-contexts

## CRD (Custom Resource Definition), allows you to extend the K8s objects, go beyond pods, services, deployments etc
	kubectl get crds
	
> Controller manager manages the controller, which in turn are responsible for implementing the reconciliation loop for a particular crd


### Extract the value of a secret and pipe it into a file
	kubectl get secrets
	kubectl get secret demo-config -o json | jq -r .data.value | base63 --decode > ./demo.config

	
6. To show at which level resources are scoped
	`kubectl api-resources`
	
7. Useful etcd commands
	```etcdctl version
	etcdctl member list```
	
### Get the list of resource you're authorized to interact with in a cluster
	kubectl auth can-i --list
	
### To access a cluster, you need to know the location of the cluster and have credentials to access it. (All clusters you have access to)
	kubectl config view
	kubectl cluster-info

## INGRESS
Kubernetes Ingress resources and controllers provide higher-level routing capabilities, such as HTTP, for services running on your cluster. Ingress helps with:
- Traffic consolidation (one entry point for traffic into a cluster)
- TLS Management
- Path based routing (L7)

> Network policies are namespace-scoped, and that's why there's the need for a namespaceSelector

### Persistent Volume and Claims

PV is the way to define the storage data, such as storage classes. Unlike ordinary volumes, PV is a resource object in a Kubernetes cluster; creating a PV is equivalent to creating a storage resource object. To use this resource, it must be requested through persistent volume claims (PVC). 
A PVC volume is a request for storage, which is used to mount a PV into a Pod.

<!-- To check volumes and get the persistent volume list: -->
`kubectl get pv`

<!-- To get the persistent volume claim list: -->
`kubectl get pvc`

<!-- To get the storage class list: -->
`kubectl get sc`


### ServiceAccount
Every Pod that's deployed to a Kubernetes cluster (with more or less default configuration) will have a service account associated with it. *“What’s a service account?”* It’s a way of giving and “identity” to a process that runs on a specific Pod. In simple words it means that each pod will have its own “username” and “password” (actually it’s just a token, but the analogy holds). By default, every Pod in your cluster will be associated with a single service account called… well, “default”

Where could it prove useful? As a result, Pod can use these credentials to call cluster’s apiserver, and the apiserver will know exactly which Pod (or actually - which service account) is calling it. That’s quite useful if you want to restrict what Pods can and cannot do when calling the Kubernetes control plane API.



# TROUBLESHOOTING

- To see the endpoint associated with a running pod
	`kubectl get endpoints pod-name`
	

- To check nodes with taints and might be missing a matching toleration
	`kubectl get nodes -o json | jq '.items[].spec.taints'`
	
> ...Here we see that node01 is in a Ready state, but that Scheduling is disabled. It appears as though someone has cordon'd that node so that the scheduler wouldn't deploy resources to it. After talking with teammates you find out that someone was testing and forgot to enable scheduling again.

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
If the container can't start, then Kubernetes shows the CrashLoopBackOff message as a status.
Usually, a container can't start when:
- There's an error in the application that prevents it from starting.
- You misconfigured the container.
- The Liveness probe failed too many times.

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

