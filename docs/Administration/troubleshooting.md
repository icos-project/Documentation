---
weight: 100
---

# Troubleshooting

This section collects some of the most frequent errors and issues that might appear when managing an ICOS Continuum.

## On-Boarding
### New ICOS Workers do not appear as resources in the ICOS Continuum

After the installation of a new ICOS Worker, it should be possible to see it in the [Telemetry Dashboards](./ICOS%20Controller/telemetry-dashboards.md#icos-agent-view-dashboard) or using the `icos-shell get resource` command.

If it is not happening, the first possible cause could be related to the telemetry from the new ICOS Worker that is not "flowing" correctly from the Worker to the ICOS Agent and from the ICOS Agent to the ICOS Controller.

As first step to debug this issue, connect to the ICOS Worker and get the logs of the pods that have in the name `...-telemetruum-leaf-otel-node-agent-...` (for the Kubernetes version) or the container named `otelcol` (for the Docker verion) and see if there are exceptions or error messages. The following is a **normal log** of the telemetry containers:

```
2025-02-21T07:06:17.490Z        info    Metrics {"kind": "exporter", "data_type": "metrics", "name": "debug", "resource metrics": 1, "metrics": 110, "data points": 477}
2025-02-21T07:06:23.910Z        info    Metrics {"kind": "exporter", "data_type": "metrics", "name": "debug", "resource metrics": 65, "metrics": 725, "data points": 781}
2025-02-21T07:06:27.719Z        info    Metrics {"kind": "exporter", "data_type": "metrics", "name": "debug", "resource metrics": 1, "metrics": 12, "data points": 50}
2025-02-21T07:06:33.953Z        info    Metrics {"kind": "exporter", "data_type": "metrics", "name": "debug", "resource metrics": 1, "metrics": 44, "data points": 1879}
2025-02-21T07:07:10.216Z        info    Metrics {"kind": "exporter", "data_type": "metrics", "name": "debug", "resource metrics": 1, "metrics": 6, "data points": 7}
2025-02-21T07:07:17.435Z        info    Metrics {"kind": "exporter", "data_type": "metrics", "name": "debug", "resource metrics": 1, "metrics": 110, "data points": 477}
2
```

Typical errors are related to the network like `Connection refused`, `No such host`, `Timeout`, `Invalid Certificate` or `Expired Certificate`. This indicates that the Worker is not able to connect and send data to the Telemetry Gateway in the ICOS Agent.

This is a typical **error log** with network errors:
```
2025-02-21T06:18:27.835Z        info    internal/retry_sender.go:126    Exporting failed. Will retry the request after interval.        {"kind": "exporter", "data_type": "metrics", "name": "otlphttp/gateway", "error": "failed to make an HTTP request: Post \"https://telemetry.ocm-agent-1.staging.10-160-3-14.sslip.io/v1/metrics\": dial tcp: lookup telemetry.ocm-agent-1.staging.10-160-3-14.sslip.io on 10.43.0.10:53: no such host", "interval": "4.277566092s"}
2025-02-21T06:18:29.669Z        info    internal/retry_sender.go:126    Exporting failed. Will retry the request after interval.        {"kind": "exporter", "data_type": "metrics", "name": "otlphttp/gateway", "error": "failed to make an HTTP request: Post \"https://telemetry.ocm-agent-1.staging.10-160-3-14.sslip.io/v1/metrics\": dial tcp: lookup telemetry.ocm-agent-1.staging.10-160-3-14.sslip.io on 10.43.0.10:53: no such host", "interval": "11.172065255s"}
2025-02-21T06:18:31.310Z        info    internal/retry_sender.go:126    Exporting failed. Will retry the request after interval.        {"kind": "exporter", "data_type": "metrics", "name": "otlphttp/gateway", "error": "failed to make an HTTP request: Post \"https://telemetry.ocm-agent-1.staging.10-160-3-14.sslip.io/v1/metrics\": dial tcp: lookup telemetry.ocm-agent-1.staging.10-160-3-14.sslip.io on 10.43.0.10:53: no such host", "interval": "9.043482127s"}
2025-02-21T07:03:23.988Z        info    internal/retry_sender.go:126    Exporting failed. Will retry the request after interval.        {"kind": "exporter", "data_type": "metrics", "name": "otlphttp/gateway", "error": "failed to make an HTTP request: Post \"https://telemetry.ocm-agent-1.staging.10-160-3-14.sslip.io/v1/metrics\": read tcp 10.0.1.49:42008->10.160.3.14:443: read: connection reset by peer", "interval": "4.818103227s"}
...
2025-02-20T15:31:52.171Z        info    internal/retry_sender.go:126    Exporting failed. Will retry the request after interval.        {"kind": "exporter", "data_type": "metrics", "name": "otlphttp/tlum-hub", "error": "failed to make an HTTP request: Post \"https://telemetry.controller.icos-stable.10-127-50-2.sslip.io:31000/v1/metrics\": tls: failed to verify certificate: x509: certificate has expired or is not yet valid: current time 2025-02-20T15:31:52Z is after 2025-02-20T02:23:02Z", "interval": "25.655902211s"}
2025-02-20T15:31:53.259Z        error   internal/queue_sender.go:100    Exporting failed. Dropping data.        {"kind": "exporter", "data_type": "metrics", "name": "otlphttp/tlum-hub", "error": "no more retries left: failed to make an HTTP request: Post \"https://telemetry.controller.icos-stable.10-127-50-2.sslip.io:31000/v1/metrics\": tls: failed to verify certificate: x509: certificate has expired or is not yet valid: current time 2025-02-20T15:31:53Z is after 2025-02-20T02:23:02Z", "dropped_items": 33}
```

Each error will have its own solution, but a general list of things to double check is:

- verify that the URL to which the Telemetry module in the Worker is trying to connect is correct. If not, the `url` value of the Helm value file is probably wrong. Also make sure that the URL is reachable from the Worker

- verify that the Telemetry Gateway in the ICOS Agent is listening correctly.
  
    For instance, the command `curl -k https://telemetry.my-agent-url.com/v1/metrics` should return `405 method not allowed, supported: [POST]`

- verify that the TLS certificate (if HTTPS is used) for the Telemetry service in the ICOS Agent is not expired. If it is expired, some components of the ICOS CA could be not working (see the specific section in this page)

If in the ICOS Worker there are no errors, the same should be checked in the ICOS Agent looking at the pod that has in the name `...-telemetruum-gateway-otel-collector-...`. In this case, if there are errors, it indicates that the Telemtry module in the ICOS Agent is not able to send data to the ICOS Controller.


### TLS errors

ICOS services uses TLS certificates to establish HTTPS connections. The certificates are issued by the ICOS CA running in the ICOS Core node.


If in the logs of an ICOS service there are errors related to TLS certificates, check the status of certificates in the ICOS Core, Controllers and Agents clusters.

```
❯ kubectl get certificate -n icos-system
NAME                                  READY   SECRET                                AGE
agent-nuvla-1-telemetry-ingress-tls   True    agent-nuvla-1-telemetry-ingress-tls   27d
agent1-telemetry-ingress-tls          True    agent1-telemetry-ingress-tls          70d
contrl1-grafana-ingress-tls           True    contrl1-grafana-ingress-tls           7d16h
contrl1-jobmanager-ingress-tls        True    contrl1-jobmanager-ingress-tls        27d
contrl1-shell-ingress-tls             True    contrl1-shell-ingress-tls             27d
contrl1-telemetruum-ingress-tls       True    contrl1-telemetruum-ingress-tls       27d
contrl1-test-polmangui-ingress-tls    True    contrl1-test-polmangui-ingress-tls    24d
```

If there are missing certificates, it might indicate that the `Cert Manager` component in the cluster is not working properly.

If there are certificates that are not `Ready` or are expired, it might indicate that the ICOS CA is not working properly or the Step Issuer component in the cluster is not configured correctly.

Typical things to check are:

- check that in the **ICOS Core** node the CA pod is running fine without errors in the logs
    ```bash
    ❯ kubectl get pods -n icos-system -l app.kubernetes.io/name=step-certificates
    core1-step-certificates-0                                        1/1     Running     0          70d
    ```
    and that it is exposed at the expected URL/IP (it depends on the Helm values used in the ICOS Core installation)

- in the clusters with missing or invalid certificates:

    - check that the `cert-manager` pods are running fine without errors in the logs
      ```bash
      ❯ kubectl get pods -n icos-system -l app.kubernetes.io/name=cert-manager
      NAME                                               READY   STATUS    RESTARTS        AGE
      contrl1-cert-manager-cainjector-5554548786-ds44g   1/1     Running   168 (21h ago)   73d
      contrl1-cert-manager-controller-57c565d4d5-fw66q   1/1     Running   142 (21h ago)   73d
      contrl1-cert-manager-webhook-7df445c8f7-vpddd      1/1     Running   181 (21h ago)   73d
      ```
    - check that the `step-issuer` pod is running fine without errors in the logs. This component is the one responsible for connecting to the ICOS CA and request the issuing of certificates
      ```bash
      ❯ kubectl get pods -n icos-system -l app.kubernetes.io/name=step-issuer
      NAME                                   READY   STATUS    RESTARTS        AGE
      contrl1-step-issuer-758755cbdf-44ssr   2/2     Running   195 (21h ago)   27d
      ```
    - check that the issuer configuration is ok. If not double check that the `global.core.url`, `global.core.routing`, `global.core.ca.bundle`, `global.core.ca.issuerKid` and `global.core.ca.issuerPassword` values in the Helm value file are correct and corresponds to the ones used in the [ICOS Core Suite installation](./ICOS%20Core/icoscore.md#step-4-upgrade-the-suite-configuration).
      ```bash
      ❯ kubectl get stepclusterissuers.certmanager.step.sm -o yaml
      [ verify in the output that the is in a Ready status]
      ...
      status:
          conditions:
          - lastTransitionTime: "2025-02-20T10:42:23Z"
          message: StepClusterIssuer verified and ready to sign certificates
          reason: Verified
          status: "True"
          type: Ready
      ...
      ```

### Keycloak JWT Token or Credentials errors

==TBD==



## ICOS Shell

### `Connection Refused` exception while using any command

If the Lighthouse is exposed as HTTPS service and you are getting the following errors:
```
Controller not defined, asking lighthouse for a controller...
Error fetching controllers: Get "http://localhost:8080/api/v3/controller/": dial tcp [::1]:8080: connect: connection refused
```

it is related to a known isuse in the ICOS Shell CLI that is not able to handle https endpoints. We are working to fix this issue.

In the meantime, the solution for this is to expose the Shell Backend service as nodeport and update the ICOS Shell CLI configuration to point to the new endpoint (that will be in the format ip:port and so supported).


1. Add the following section to the ICOS Controller Helm values file:
   ```yaml
   shell-backend:
     service:
       type: NodePort
       nodePort: 32500
   ```

2. Upgrade the ICOS Controller Helm release: `helm upgrade...`

3. Modify the ICOS Shell CLI configuration file, removing the `lighthouse` section and adding the `controller: <controller-ip>:32500` value. For instance:
   ```yaml
   controller: 10.10.10.1:32500
   keycloak:
     user: test
     pass: test
   ```
