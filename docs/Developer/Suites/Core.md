---
status: draft
review:
  editor: ENG
  version: 0.1
---

# ICOS Core Suite

There are multiple configuration parameters that can be specified as Helm values for the installation of the ICOS Core Suite that will drive how it is installed. This page collects some common configuration values. Make sure to read the [Exposure](Exposure.md) section for all possible values.

## Use HTTPS with Ingress Controller exposed on port 443

```yaml linenums="1"
global:
  core:
    # IMPORTANT the port in the url must match the port on 
    # which the ingress controller is exposed
    url: https://my.icos-core.continuum.org/
    routing: host

icos-ingress-controller:
  enabled: true
  nginx-ingress-controller:
    kind: DaemonSet
    daemonset:
      useHostPort: true
    service:
      type: ClusterIP
```


## Use HTTPS with Ingress Controller exposed as NodePort

```yaml linenums="1"
global:
  core:
    # IMPORTANT the port in the url must match the port on 
    # which the ingress controller is exposed
    url: https://my.icos-core.continuum.org:31000/
    routing: host

icos-ingress-controller:
  enabled: true
  nginx-ingress-controller:
    service:
      type: NodePort
      nodePorts:
        https: 31000
```

## Use HTTP without Ingress Controller

With this configuration the `CA`, `Lighthouse` and `IAM` services will be exposed on three different ports using HTTP protocol.

!!! danger
    This is an insecure setup since it is not using HTTPS protocol and should be avoided!

```yaml linenums="1"
global:
  core:
    url: 10.10.10.10
    routing: port

# we disable the CA because it will not be needed
icos-ca:
  enabled: false

```

## Use pre-generated CA secrets and certificates.

Using this method, the ICOS CA secrets and certificates are generated before the installation and specified as Helm values (while in the standard method suggested in the Administration guide, they are generated automatically after the installation). This method allow to fully automate the installation in a single step without manual setup. It is useful to manage the installation using automation tools (e.g. Argo CD, GitLab).

1. First install the `step` cli (https://smallstep.com/docs/step-cli/)

2. Use the `step` command to generate the configuration.
    ```
    export RELEASE_NAME=core1
    export RELEASE_NAMESPACE=icos-system
    export CA_PUBLIC_URL=ca.core.icos-staging.10-160-3-151.sslip.io
    ```

    ```bash
    step ca init --deployment-type=standalone --name=my-icos-continuum-ca --dns "$CA_PUBLIC_URL" --dns "$RELEASE_NAME-step-certificates.$RELEASE_NAMESPACE.svc.cluster.local" --dns 127.0.0.1 --address=0.0.0.0:9000 --provisioner admin --helm
    ```

    The command will prompt for a key that will be used to encrypt the CA secrets. Enter a password or type "Enter" to let the script generate one. Take note of the generated password (CA_PASSWORD).


    Encode the CA_PASSWORD in base64

    ENCODED_PWD = echo "CA_PASSWORD" | base64 


    Take note of the generated output (CA_HELM_VALUES).

3. Create a yaml file for the ICOS Core Suite using the values generated in the previous step:
    ```yaml
    global:
      external:
        # the dns name of this node. This will be used to generate the DNS Names for the IAM and Lighthouse services
        host: core.icos-staging.10-160-3-151.sslip.io

    icos-ca:
      step-certificates:

        #
        # Copy here the entire output of the `step init ca` command. Pay attention to keep it indented under the `icos-ca.step-certificates` section.
        #
        #

        inject:
          enabled: true
          # Config contains the configuration files ca.json and defaults.json
          config:
            files:
              ca.json:
                root: /home/step/certs/root_ca.crt
                federateRoots: []

          [...]

          secrets:
            ca_password: <BASE64_ENCODED_PWD>
            provisioner_password: <BASE64_ENCODED_PWD>

          [...]      

    ```

4. Run the `helm install` command:

    ```
    helm install --namespace icos-system core1 /data/ICOS/code/suites/icos-core/ --values my-values.yaml --set icos-ingress-controller.step-issuer.icos-issuer.enabled=false
    ```

## Backup / Restore

```
RELEASE_NAME=testcore kubectl get secret/$RELEASE_NAME-step-certificates-ca-password secret/$RELEASE_NAME-step-certificates-provisioner-password configmap/$RELEASE_NAME-step-certificates-secrets configmap/$RELEASE_NAME-step-certificates-config configmap/$RELEASE_NAME-step-certificates-certs -o yaml > ca_backup.yaml
```

```
kubectl apply -f ca_backup.yaml
```

```
helm upgrade --install --namespace gabriele-core testcore /data/ICOS/code/suites/icos-core/ --values values-2.yaml --values persistence.yaml --set icos-ca.step-certificates.bootstrap.enabled=false --set icos-ca.step-certificates.bootstrap.configmaps=false --set icos-ca.step-certificates.bootstrap.secrets=false
```

## Troubleshooting

### The requested resource could not be found. Please see the certificate authority logs for more info.

If the following error appears in the StepIssuer log:
```
"The requested resource could not be found. Please see the certificate authority logs for more info.
```

and the following error appears in the cert-manager-controller log:
```
failed calling webhook \"webhook.cert-manager.io\": failed to call webhook: Post \"https://cert-manager-webhook.cert-manager.svc:443/validate?timeout=30s\": service \"cert-manager-webhook\" not found" logger="cert-manager.controller" key="gabriele-core/testcore-keycloak"
```
then:
1. Delete the Helm release
2. Run `kubectl get MutatingWebhookConfiguration` and delete the old ones
3. RUun `kubectl get validatingwebhookconfigurations` and delete the old ones
4. Reinstall the Helm chart

### Error "spec.caBundle: Required value" is shown when performing an helm install/updgrade

If you see this error during the first step of the ICOS Core Suite installation:
```
* StepClusterIssuer.certmanager.step.sm "icos-step-ca-issuer" is invalid: [spec.provisioner.kid: Required value, spec.caBundle: Required value]
```

Then you forgot to disable the issuer component. Append the following to the command line:
```
--set icos-ingress-controller.step-issuer.icos-issuer.enabled=false
```