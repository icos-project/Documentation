
# Services Exposure

Each ICOS node (excepting for the Workers) runs multiple services that needs to be exposed to be called by other nodes. For instance, each ICOS Controller exposes the `Shell Backend` service that is called by the `ICOS Shell CLI` and the `Job Manager` that is called by the ICOS Agents. Similarly, the ICOS Agents expose the `Telemetry Gateway` service that is called by the ICOS Workers.

<figure markdown="span">
![An ICOS Continuum](../../assets/images/suites-services-light.png#only-light){ align=center }
![An ICOS Continuum](../../assets/images/suites-services-dark.png#only-dark){ align=center }
  <figcaption>Services exposed by ICOS nodes</figcaption>
</figure>

Exposing services publicly in Kubernetes clusters is a vast and, in some cases, complex aspect and **a basic understanding of it is assumed in the rest of this section**. Several [resources](https://www.scaleway.com/en/docs/kubernetes/reference-content/exposing-services/) on this [topic](https://medium.com/@seanlinsanity/how-to-expose-applications-running-in-kubernetes-cluster-to-public-access-65c2fa959a3b) are [available](https://medium.com/@seanlinsanity/how-to-expose-applications-running-in-kubernetes-cluster-to-public-access-65c2fa959a3b) online.

The decisions on how to expose services stronly depends on the configuration and the capabilities of the cluster where the node is being installed. The ICOS Suites cover the configuration for exposing the services using either and **Ingress Controller** or **NodePort** services. Other options can work, but the administrator of the cluster will have to manually configure them.

The configuration of the exposure of services in the ICOS Suites is based on two Helm values:

- `url` that defines a base url or IP used to derive the url/IP where each single service of the sute will be exposed
- `routing` that defines the method used to multiplex the different services of the suite .

The `url` parameter can be an ip address or a valid url with **protocol** (defaults to `http`), **host** (mandatory), **port** (defaults to `80` or `443`) and **path** (defaults to `/`). Valid examples of values are:

- http://192.168.100.30/

- https://10.160.3.100:31000/

- https://my-icos-controller.com/

- http://my-icos-controller.com:31200/acme

- my-icos-controller

The `routing` parameter can be one of:

- **host**: expose each service to a different subdomain of `url`

- **path**: expose each service to a different path of `url`

- **port**: expose each service to a different port of `url`

Based on those two parameters the Helm templates will derivce the exact way each service of the suite will be exposed to the extern.


!!! warning
    `host` routing can only be used if `url` is an URL and not an IP address

!!! warning
    both `host` and `path` routing will make use of Kubernetes Ingresses. It will need the `icos-ingress-controller.enabled` value to be `true`.

## Routing options

### Host Routing

Given the ICOS Core Suite as example, if the Helm values are:
```yaml
global:
  core:
    routing: host
    # when using routing: host the url cannot be an IP address
    url: https://my-icos-core.example.org/

# enables the Nginx Ingress Controller. See the section below
icos-ingress-controller:
  enabled: true
```

the suite's services will be exposed at:

| Service    | Url                                          |
| ---------- | -------------------------------------------- |
| CA         | https://ca.my-icos-core.example.org/         |
| IAM        | https://iam.my-icos-core.example.org/        |
| Lighthouse | https://lighthouse.my-icos-core.example.org/ |

The name of the subdomains for each service can be customized with the `global.core.ca.config.subdomain`, `global.core.iam.config.subdomain` and `global.core.lighthouse.config.subdomain` values respectively.

Note that no service will be exposed at the url provided in the `global.core.url` value. It is used just as base url to build the urls of the other services.

### Path Routing

Given the ICOS Core Suite as example, if the Helm values are:
```yaml
global:
  core:
    routing: path
    url: https://my-icos-core.example.org/

# enables the Nginx Ingress Controller. See the section below
icos-ingress-controller:
  enabled: true
```

the suite's services will be exposed at:

| Service    | Url                                         |
| ---------- | ------------------------------------------- |
| CA         | https://my-icos-core.example.org/ca         |
| IAM        | https://my-icos-core.example.org/iam        |
| Lighthouse | https://my-icos-core.example.org/lighthouse |

The name of the subdomains for each service can be customized with the `global.core.ca.config.path`, `global.core.iam.config.path` and `global.core.lighthouse.config.path` values respectively.

Note that no service will be exposed at the url provided in the `global.core.url` value. It is used just as base url to build the urls of the other services.

### Port Routing

Given the ICOS Core Suite as example, if the Helm values are:
```yaml
global:
  core:
    url: 10.160.3.100
    routing: port

# Nginx Ingress Controller not needed since NodePort services will
# be used
#icos-ingress-controller:
#  enabled: true
```

the suite's services will be exposed at:

| Service    | Url                |
| ---------- | ------------------ |
| CA         | 10.160.3.100:32010 |
| IAM        | 10.160.3.100:32020 |
| Lighthouse | 10.160.3.100:32030 |
|            |                    |

The name of the subdomains for each service can be customized with the `global.core.ca.config.port`, `global.core.iam.config.port` and `global.core.lighthouse.config.port` values respectively.

## ICOS Ingress Controller

The **host** and **path** routing mechanisms make use of [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) and [Certificate](https://cert-manager.io/docs/usage/certificate/) resources. The ICOS Suites will automatically generate the appropriate resources based on the `url` and `routing` values, but an additional software to handle those resources is needed at cluster level.

In order to simplify the deployment and reduce the requirements to install the ICOS Suites, it is possible to install all the required software directly using the `ICOS Ingress Controller` Helm Chart (also available as dependency of each ICOS Suite Chart).

The `ICOS Ingress Controller` chart contains the following components:

- [NGINX Ingress Controller](https://github.com/kubernetes/ingress-nginx) to deploy a reverse proxy that manage the exposure of the internal services using the configuration provided through the Ingresses

- [Cert Manager](https://cert-manager.io/docs/) to autoamte the generation and renewal of TLS certificates for each service of the ICOS Suites exposed to the extern

- [Step Issuer](https://github.com/smallstep/step-issuer) a component that creates required TLS certificates contacting the ICOS CA.

In each ICOS Suite Helm chart, it is possible to deploy the `ICOS Ingress Controller` with the following value:

```yaml
icos-ingress-controller:
  enabled: true
```

The default values will use a `LoadBalancer` service to expose the Nginx Ingerss Controller. When using `LoadBalancer` service, the Nginx Ingress Controller is used on the default ports 80 and 443 (but the specific cluster configuration might be different, please check carefully).

If `LoadBalancer` services are not available in the cluster, it is possible to deploy the Nginx Ingress controller using a `NodePort` service:

```yaml
# user NodePort
icos-ingress-controller:
  enabled: true
  nginx-ingress-controller:
    service:
      type: NodePort
      nodePorts:
        https: 30000

global:
  core|agent|controller:
    url: https://....:30000/  # port used in the url MUST match the port at which the ingress controller is exposed!
```

It is also possible to install the Nginx Ingress Controller as a Daemonset and be able to use host's port `80` and `443`:

```yaml
# use Daemonset
icos-ingress-controller:
  enabled: true
  nginx-ingress-controller:
    kind: DaemonSet
    daemonset:
      useHostPort: true
    service:
      type: ClusterIP
```

Documentation on more configuration options is available in [GitHub](https://github.com/icos-project/ICOS-Ingress-Controller) and in the [Bitnami Nginx Ingress Controller Helm chart](https://github.com/bitnami/charts/tree/main/bitnami/nginx-ingress-controller).

!!! important
    Regardless of the way used to expose the Ingress Controller (LoadBalancer, Daemonset or NodePort) it is important that the port number in the `global.[agent|core|controller].url` Helm value is the same of the one used by the Ingress Controller. This because it indicates to which port the ICOS services will be reachable from the extern.

    For instance, in the Helm values for an ICOS Controller Suite:

    ```yaml
    global:
      controller:
        # WRONG!!! Here it is defined port 443 as the one where services are
        # exposed, but in reality they are exposed on port 32000
        url: https://my-controller.example.org/
        # CORRECT!
        url: https://my-controller.example.org:32000/
    
    # Ingress Controller is exposed on port 32000
    icos-ingress-controller:
      enabled: true
      nginx-ingress-controller:
        service:
          type: NodePort
          nodePorts:
            https: 32000
    ```

!!! important
    Only **one instance** of the `ICOS Ingress Controller` must exist in the cluster. In deployment schema where multiple ICOS Suites are deployed in the same cluster, make sure to activate the `ICOS Ingress Controller` in only one of them.

### Insecure exposure with http

It is possible to expose the ICOS services also using the **http** protocol. However this is not recommended for obvious security risks.

In order to deploy an insecure ICOS Core Suite with a path routing using http, the follwoing values can be used an example:

```yaml
global:
  core:
    url: my-insecure-core.com
    routing: path

# we disable the CA because it will not be needed
icos-ca:
  enabled: false

icos-ingress-controller:
  step-issuer:
    enabled: false
  cert-manager:
    enabled: false
  nginx-ingress-controller:
    config:
      ssl-redirect: "false"

icos-iam:
  keycloak:
    ingress:
      tls: false

icos-lighthouse:
  ingress:
    tls: []
```

In order to deploy an ICOS Controller using **port** routing the following values can be used as an example:
```yaml
global:
  controller:
    url: 192.168.100.100
    routing: port

icos-ingress-controller:
  enabled: false
```

## Exposure of Zenoh

The ICOS Controller, Agent and Worker Suites include a deployment of **Zenoh**. Since, unlike all the other services exposed in the Suites, Zenoh does not use the HTTP protocol, but TCP connections. For this reason, it cannot be (easily) exposed using the routing options described in the previous sections. For this reason, at the moment, when Zenoh is enabled in the ICOS Controller and ICOS Agent Suites (`zenoh.enabled: true`), it will create a NodePort service on port `32700` (customizable).