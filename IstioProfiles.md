# Istio Profiles

###### tags: `istio` `kubernetes`

Istio comes with 5 predefined profiles. We can install these profiles and later customize it.

1. **default**: enables components according to the default settings of the IstioControlPlane API. You can display the default setting by running the following command 
```
# istioctl profile dump
```
2. **demo**: This is set for example. not suitable for performance tests
3. **minimal**: the basic set of components necessary to use Istio’s traffic management features.
4. **sds**: *default profile* + Istio’s SDS (secret discovery service).
5. **remote**: used for configuring remote clusters of a multicluster mesh with a shared control plane configuration.



|                        | default | demo | minimal | sds | remote |
| ---------------------- |:-------:|:----:|:-------:|:---:|:------:|
| **Core components**    |         |      |         |     |        |
| istio-citadel          |    ✔    |  ✔   |    ✖    |  ✔  |   ✔    |
| istio-egressgateway    |    ✖    |  ✔   |    ✖    |  ✖  |   ✖    |
| istio-galley           |    ✔    |  ✔   |    ✖    |  ✔  |   ✖    |
| istio-ingressgateway   |    ✔    |  ✔   |    ✖    |  ✔  |   ✖    |
| istio-nodeagent        |    ✖    |  ✖   |    ✖    |  ✔  |   ✖    |
| istio-pilot            |    ✔    |  ✔   |    ✔    |  ✔  |   ✖    |
| istio-policy           |    ✔    |  ✔   |    ✖    |  ✔  |   ✖    |
| istio-sidecar-injector |    ✔    |  ✔   |    ✖    |  ✔  |   ✔    |
| istio-telemetry        |    ✔    |  ✔   |    ✖    |  ✔  |   ✖    |
| **Addons**             |         |      |         |     |        |
| grafana                |    ✖    |  ✔   |    ✖    |  ✖  |   ✖    |
| istio-tracing          |    ✖    |  ✔   |    ✖    |  ✖  |   ✖    |
| kiali                  |    ✖    |  ✔   |    ✖    |  ✖  |   ✖    |
| prometheus             |    ✔    |  ✔   |    ✖    |  ✔  |   ✖    |

To further customize Istio and install addons, you can add one or more --set <key>=<value> options in the istioctl manifest command that you use when installing Istio. Refer to customizing the configuration for details.