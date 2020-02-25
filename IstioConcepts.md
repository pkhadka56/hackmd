# Istio Concepts
https://www.slideshare.net/luckyfengyong/istio-deep-dive

###### tags: `istio` `kubernetes`

## Data plane
- composed of a set of proxy (Envoy) deployed as sidecars
![](https://i.imgur.com/H6BHe3t.jpg)

- these proxies control all  network communication between microservices along with mixer, a general-purpose policy and telemetry hub

## Control plane
- manages and configures the proxies to route traffic.
- configures mixers to  enforce  policies and collect telemetry.

**Proxy**: Bashed on Envoy, mediated inbound and outbound traffic for all  istio-managed services
**Pilot**: Configures istio deployments and propagate configuration to the other componenets of the system
**Mixer**: Responsible for policy decisions and aggregating telemetry data from the other components in the system using a flexible plugin architecture
**Citadel**: Secures the service-to-service  communication and provides akey management system to manage keys and certificates.


# Routing control
### Ingress:
gateway: configures a load balancer for HTTP/TCP traffic at the edge  of the mesh
### Inside mesh:
**VirtualService**: defines the rules that control how requests for a service are routed
**DestinationRule**: configures the set of policies to be applied to a request after VirtualService routing has  occured
### Egress:
**ServiceEntry**: commonly used to enablerequests to services outside of an Istio service mesh

> More on another note