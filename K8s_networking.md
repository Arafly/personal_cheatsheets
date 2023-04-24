## Service Mesh

### East-West API Gateway Use Cases: Use a Service Mesh

As your architecture increases in complexity, you’re more likely to get value from using a service mesh. The use cases we find most beneficial are related to E2EE (End-to-end Encryption) and traffic splitting – such as A/B testing, canary deployments, and blue‑green deployments.

### Sample Scenario: Canary Deployment

You want to set up a canary deployment between services with conditional routing based on HTTP/S criteria.

The advantage is that you can gradually roll out API changes – such as new functions or versions – without impacting most of your production traffic.

Service Mesh routes traffic between two services: Coffee.frontdoor.svc and Tea.frontdoor.svc.
These services receive traffic from NGINX Ingress Controller and route it to the appropriate app functions, including **Tea.cream1.svc**. You decide to refactor Tea.cream1.svc, calling the new version **Tea.cream2.svc**. You want your beta testers to provide feedback on the new functionality so you configure a canary traffic split based on the beta testers’ *unique session cookie*, ensuring your regular users only experience Tea.cream1.svc.

* insert api-gateway-canary 

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

