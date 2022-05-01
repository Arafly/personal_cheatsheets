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
	kubectl get pods -n kube-system kube-flannel-ds-amd64-xmcnm -o json | jq .metadata.labels
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
> You can easily use an Ingress controller not only to control ingress traffic but also to deliver service‑level performance metrics and as part of a security policy. Ingress controllers have many features of traditional external load balancers, like TLS termination, handling multiple domains and namespaces, and of course, load balancing traffic. Ingress controllers can load-balance traffic at the per‑request rather than per‑service level, a more useful view of Layer 7 traffic and a far better way to enforce SLAs. Ingress controllers can also enforce egress rules which permit outgoing traffic from certain pods only to specific external services, or ensure that traffic is mutually encrypted using mTLS.

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

When you need to provide external access to your Kubernetes services, you create an Ingress resource to define the connectivity rules, including the URI path, backing service name, and other information. On its own, however, the Ingress resource doesn’t do anything. You must deploy and configure an Ingress controller application (using the Kubernetes API) to implement the rules defined in Ingress resources.

> Network policies are namespace-scoped, and that's why there's the need for a namespaceSelector

### Persistent Volume and Claims

- PV is the way to define the storage data, such as storage classes. Unlike ordinary volumes, PV is a resource object in a Kubernetes cluster; creating a PV is equivalent to creating a storage resource object. To use this resource, it must be requested through persistent volume claims (PVC). 
- A PVC volume is a request for storage, which is used to mount a PV into a Pod.
- PVs and PVCs are analogous to nodes and pods. Just like a node is a computing resource, and a pod seeks a node to run on, a PersistentVolume is a storage resource, and a PersistentVolumeClaim seeks a PV to bind to.

<!-- To check volumes and get the persistent volume list: -->
`kubectl get pv`

<!-- To get the persistent volume claim list: -->
`kubectl get pvc`

<!-- To get the storage class list: -->
`kubectl get sc`


### ServiceAccount
Every Pod that's deployed to a Kubernetes cluster (with more or less default configuration) will have a service account associated with it. *“What’s a service account?”* It’s a way of giving and “identity” to a process that runs on a specific Pod. In simple words it means that each pod will have its own “username” and “password” (actually it’s just a token, but the analogy holds). By default, every Pod in your cluster will be associated with a single service account called… well, “default”

Where could it prove useful? As a result, Pod can use these credentials to call cluster’s apiserver, and the apiserver will know exactly which Pod (or actually - which service account) is calling it. That’s quite useful if you want to restrict what Pods can and cannot do when calling the Kubernetes control plane API.


