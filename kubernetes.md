1. Bootstrap a yaml file
   
	`kubectl run redis --image=nginx --dry-run=client -o yaml > pod.yaml`

	`kubectl apply -f pod.yaml`

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
	
4. Format your output(json) and pipe it into a readable
	```kubectl get po -n kube-system
	kubectl get pods -n kube-system | grep etcd
	kubectl get pods -n kube-system kube-flannel-ds-amd64-xmcnm -ojson | jq .metadata.labels
	kubectl get pods -n kube-system kube-apiserver-node1 -o custom-columns=NAME:.metadata.name,NS:metadata.namespace
	kubectl get pods -n kube-system -o custom-columns=NAME:.metadata.name,NS:metadata.namespace
	```
	
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
	
### To access a cluster, you need to know the location of the cluster and have credentials to access it.
	kubectl config view
	kubectl cluster-info

## INGRESS
Kubernetes Ingress resources and controllers provide higher-level routing capabilities, such as HTTP, for services running on your cluster. Ingress helps with:
- Traffic consolidation (one entry point for traffic into a cluster)
- TLS Management
- Path based routing (L7)