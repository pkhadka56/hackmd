# Istio

https://istio.io/docs/concepts/what-is-istio/ 

###### tags: `istio` `kubernetes`

![](https://i.imgur.com/yXuXweJ.png)

## What is Istio?
1. Open source
2. Control plane for a service mesh
3. Istio lets you connect, secure, control, and observe services.
4. Istio makes it easy to create a network of deployed services

## Why use Istio?
1. Automatic load balancing for HTTP, gRPC, WebSocket, and TCP traffic.
2. Fine-grained control of traffic behavior with rich routing rules, retries, failovers, and fault injection.
3. A pluggable policy layer and configuration API supporting access controls, rate limits and quotas.
4. Automatic metrics, logs, and traces for all traffic within a cluster, including cluster ingress and egress.
5. Secure service-to-service communication in a cluster with strong identity-based authentication and authorization.

## Features
1. Traffic management
2. Security
3. Policis
4. Observability
5. Platform support
6. Integration and customization

### 1. Traffic management
- Istio’s easy rules configuration and traffic routing lets you control the flow of traffic and API calls between services
- If you’ve installed Istio on a Kubernetes cluster, then Istio automatically detects the services and endpoints in that cluster.
- It relies in Envoy proxies
- All traffic that your mesh services send and receive (data plane traffic) is proxied through Envoy
- By default, the Envoy proxies distribute traffic across each service’s load balancing pool using a round-robin model, where requests are sent to each pool member in turn, returning to the top of the pool once each service instance has received a request.

**Envoy proxy**
1. service  proxy
2. written in c++
3. advanced load balancing

#### Traffic management API resources
a. Virtual services
b. Destination rules
c. Gateways
d. Service entries
e. Sidecars

## a. Virtual Services
- **Virtual services**, along with **destination rules**, are the key building blocks of Istio’s traffic routing functionality
- It lets you configure how requests are routed to a service within an Istio service mesh
- Each virtual service consists of a set of routing rules that are evaluated in order, letting Istio match each given request to the virtual service to a specific real destination within the mesh

**Why use vitual services?**
- flexible and powerful
- without it, Envoy distributes traffic using round-robin load balancing between all service instances, it can be improved with workloads
- We can use routing rules in the virtual service that tell Envoy how to send the virtual service’s traffic to appropriate destinations
- We can configure a virtual service to handle all services in a specific namespace. Our routing rules can specify “calls to these URIs of monolith.com go to microservice A”, and so on.
- Configure traffic rules in combination with gateways to control ingress and egress traffic.


**Example**

The following virtual service routes requests to different versions of a service depending on whether the request comes from a particular user.
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v3
```
**The host field**
```
hosts:
- reviews
```
- The virtual service hostname can be an IP address, a DNS name, or a short name (such as a Kubernetes service short name) that resolves, implicitly or explicitly, to a fully qualified domain name (FQDN).
- You can also use wildcard (”*”) prefixes
- Virtual service hosts don’t actually have to be part of the Istio service registry, they are simply virtual destinations.

**The http field**
The *http* section contains the virtual service’s routing rules, describing match conditions and actions for routing traffic sent to the destination(s) specified in the hosts field

**The match field**
```
- match:
   - headers:
       end-user:
         exact: jason
```
The first routing rule in the example has a condition and so begins with the match field. In this case you want this routing to apply to all requests from the user “jason”, so you use the headers, end-user, and exact fields to select the appropriate requests.

**Destination**
```
route:
- destination:
    host: reviews
    subset: v2
```
- The route section’s destination field specifies the actual destination for traffic that matches this condition
- the destination’s host must be a real destination that exists

**Routing rule precedence**
- Routing rules are evaluated in sequential order from top to bottom, first rule will have first pririoty
- Anything that doesn’t match the first routing rule will go to a default destination, specified in the second rule.
- The traffic will route to second destination if the user is not jason

```
- route:
  - destination:
      host: reviews
      subset: v3
```
**More routing rules**
We can also set match conditions based on traffic ports, header fields, URIs, and more. For example, this virtual service lets users send traffic to two separate services, ratings and reviews, as if they were part of a bigger virtual service at http://bookinfo.com/. The virtual service rules match traffic based on request URIs and direct requests to the appropriate service.

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
    - bookinfo.com
  http:
  - match:
    - uri:
        prefix: /reviews
    route:
    - destination:
        host: reviews
  - match:
    - uri:
        prefix: /ratings
    route:
    - destination:
        host: ratings
...
  http:
  - match:
      sourceLabels:
        app: reviews
    route:
...
```

For some match conditions, we can also choose to select them using the **exact value, a prefix, or a regex**.

In addition to using match conditions, we can distribute traffic by percentage “weight”. This is useful for A/B testing and canary rollouts:

```
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 75
    - destination:
        host: reviews
        subset: v2
      weight: 25
```
## b. Destination rules
- Think *virtual service* as how you route your traffic to a given destination
- Think *destination rules* to configure what happens to traffic for that destination
- Destination rules are applied after virtual services are evaluated
- Destination rules is use to specify named service subsets, such as grouping all a given service’s instances by version. Then use these service subsets in the routing rules of virtual services to control the traffic to different instances of your services.
- It is also use to customize Envoy’s traffic policies

**Load balancing options**
- By default, Istio uses a round-robin load balancing policy
- Istio also supports the following models
    - **Random**: Requests are forwarded at random to instances in the pool.
    - **Weighted**:  Requests are forwarded to instances in the pool according to a specific percentage.
    - **Least requests**: Requests are forwarded to instances with the least number of requests.

**Example**

The following example destination rule configures three different subsets for the my-svc destination service, with different load balancing policies:
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-destination-rule
spec:
  host: my-svc
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
  - name: v3
    labels:
      version: v3
```

The default policy, defined above the subsets field, sets a simple random load balancer for the v1 and v3 subsets. In the v2 policy, a round-robin load balancer is specified in the corresponding subset’s field.

## c. Gateways
- A gateway is to manage inbound and outbound traffic for the mesh
- Gateways are primarily used to manage ingress traffic, but you can also configure egress gateways. 
- Istio provides some preconfigured gateway proxy deployments (**istio-ingressgateway and istio-egressgateway**) that you can use - both are deployed if you use our demo installation, while just the ingress gateway is deployed with our default or sds profiles. You can apply your own gateway configurations to these deployments or deploy and configure your own gateway proxies.

**Example**
```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: ext-host-gwy
spec:
  selector:
    app: my-gateway-controller
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - ext-host.example.com
    tls:
      mode: SIMPLE
      serverCertificate: /tmp/tls.crt
      privateKey: /tmp/tls.key
```

This gateway configuration lets HTTPS traffic from ext-host.example.com into the mesh on port 443, but doesn’t specify any routing for the traffic.

To specify routing and for the gateway to work as intended, you must also bind the gateway to a virtual service. You do this using the virtual service’s gateways field, as shown in the following example:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: virtual-svc
spec:
  hosts:
  - ext-host.example.com
  gateways:
    - ext-host-gwy
```

You can then configure the virtual service with routing rules for the external traffic.

## d. Service entries
- Service entry can be use to add an entry to the service registry that Istio maintains internally. After added, the service entry, the Envoy proxies can send traffic to the service as if it was a service in your mesh
-  Configuring service entries allows you to manage traffic for services running outside of the mesh
    - Redirect and forward traffic for external destinations, such as APIs consumed from the web, or traffic to services in legacy infrastructure.
    - Define retry, timeout, and fault injection policies for external destinations.
    - Run a mesh service in a Virtual Machine (VM) by adding VMs to your mesh.
    - Logically add services from a different cluster to the mesh to configure a multicluster Istio mesh on Kubernetes.
- By default, Istio configures the Envoy proxies to passthrough requests to unknown services.
- you can’t use Istio features to control the traffic to destinations that aren’t registered in the mesh.

**Example**

The following example mesh-external service entry adds the ext-resource external dependency to Istio’s service registry:
```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: svc-entry
spec:
  hosts:
  - ext-svc.example.com
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  location: MESH_EXTERNAL
  resolution: DNS
```
You specify the external resource using the hosts field. You can qualify it fully or use a wildcard prefixed domain name.

## e. Sidecars
- By default, Istio configures every Envoy proxy to accept traffic on all the ports of its associated workload, and to reach every workload in the mesh when forwarding traffic. You can use a sidecar configuration to do the following:

    - Fine-tune the set of ports and protocols that an Envoy proxy accepts.
    - Limit the set of services that the Envoy proxy can reach.

- You might want to limit sidecar reachability like this in larger applications, where having every proxy configured to reach every other service in the mesh can potentially affect mesh performance due to high memory usage.

**Example**
The following sidecar configuration configures all services in the bookinfo namespace to only reach services running in the same namespace and the Istio control plane (currently needed to use Istio’s policy and telemetry features):

```
apiVersion: networking.istio.io/v1alpha3
kind: Sidecar
metadata:
  name: default
  namespace: bookinfo
spec:
  egress:
  - hosts:
    - "./*"
    - "istio-system/*"
```
---
## Misc
### Timeouts
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
    timeout: 10s
```

### Retries
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
    retries:
      attempts: 3
      perTryTimeout: 2s
```

### Circuit breakers
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
    trafficPolicy:
      connectionPool:
        tcp:
          maxConnections: 100
```

### Fault injection
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
    route:
    - destination:
        host: ratings
        subset: v1
```
