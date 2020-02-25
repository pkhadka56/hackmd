# Istio Architecture
https://www.slideshare.net/ssuserc5886a/istio-service-mesh-introduction

###### tags: `istio` `kubernetes`

![](https://i.imgur.com/z9AFrV7.png)

## Data plane:
- Envoy

## Control plane
- Mixer
- Pilot
- Citadel
- Galley

## Data Plane
- **Envoy**
    - circuit breakers
    - health checks
    - staged rollouts with %-based traffic split
    - fault injection
    - rich metrics

## Control Plane
- **Mixer**
![](https://i.imgur.com/50ecVFl.png)

    Responsible for providing policy controls and telemetry collection
    - Enforces access control and usage policies across the service mesh
    - collects telemetry data from the envoy proxy and other services
    - includes a flexible plugin model

- **Pilot**
![](https://i.imgur.com/TFChnja.png)

    Provides service  discovery for:
    - Envoy sidecars
    - Traffic management capabilities for intelligent routing (A/B tests, canary rollouts)
    - Resiliency (Timeouts, retries, circuit breakers)

- **Citadel**
    Strong service-to-service/end-user authentication with build-in identity and credential management

- **Galley**
    Istio's configuration validation, ingestion, processing and distribution component


