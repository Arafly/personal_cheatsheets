## Service Mesh

### East-West API Gateway Use Cases: Use a Service Mesh

As your architecture increases in complexity, you’re more likely to get value from using a service mesh. The use cases we find most beneficial are related to E2EE (End-to-end Encryption) and traffic splitting – such as A/B testing, canary deployments, and blue‑green deployments.

### Sample Scenario: Canary Deployment

You want to set up a canary deployment between services with conditional routing based on HTTP/S criteria.

The advantage is that you can gradually roll out API changes – such as new functions or versions – without impacting most of your production traffic.

Service Mesh routes traffic between two services: Coffee.frontdoor.svc and Tea.frontdoor.svc.
These services receive traffic from NGINX Ingress Controller and route it to the appropriate app functions, including **Tea.cream1.svc**. You decide to refactor Tea.cream1.svc, calling the new version **Tea.cream2.svc**. You want your beta testers to provide feedback on the new functionality so you configure a canary traffic split based on the beta testers’ *unique session cookie*, ensuring your regular users only experience Tea.cream1.svc.

* insert api-gateway-canary 
