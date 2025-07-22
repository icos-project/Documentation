# Policies

Policies can be defined in two locations:

- **per-component** policies section. Each policy defined here is applied only to the component where it is defined
- **top-level** policies section. Policies defined here are applied to all components (with the exception of custom policies with the `apply-to` key)

The following example shows where policies are allowed:
```yaml hl_lines="12-14 23-25"
name: test-policies
description: a sample manifest to describe the usage of policies
components:
  - name: comp1
    type: kubernetes
    manifests:
      - name: c1

    ## 
    ## PER-COMPONENT policies section
    ##
    policies:
      - name: ...
        type: ...

  - name: comp2
    type: kubernetes
    manifests:
      - name: c2

##
## TOP-LEVEL policies section
##
policies:
  - name: ...
    type: ...

manifests:
   ...
```

Unless specifie, each policy listeb below can be defined both at policy level (it will be applied to all the components of the app) or at component level (it will be applied only to the component where it is defined).

## Security

### Node Security Level

This policy define a security level (based on the [Wazuh SCA score](https://documentation.wazuh.com/current/user-manual/capabilities/sec-config-assessment/index.html)) for the nodes where the application should run. ICOS will ensure that the application will be deployed in a node that satisfy the security level defined. If during the runtime, the node level changes ICOS will move (redeploy) the application in a node taht satisfy the policy.

```yaml
policies:
  # short form
  - security: high

  # normal form
  - type: security
    level: high
```

Levels:

- **high**: SCA score >= 80
- **medium**: SCA score >= 50 and < 80
- **low**: SCA score >= 0 and < 50

### Security Events

This policy is based on the auditing events collected by ICOS on the workers. It is possible to trigger a violation when the number of generated events (optionally filtered by `severity` and/or `eventName`) is higher than a given threshold (`maxEvents`).

```yaml

policies:
  - type: custom
    fromTemplate: app-security-events
    remediation: redeploy
    variables:
      maxEvents: 12
      severity: "high"  # If omitted, all severity levels are considered
      eventName: "outside-connection"  # A regex can be used here. If omitted, it corrsponds to ".+"
```

In order for this policy to work, the auditing events must be enabled in the worker (the `icos-auditing: true` value should be provided when installing the ICOS Worker Suite).

## Resources Usage and Availability

### Node CPU and Memory

This policy allows to define a threshold of CPU and/or Memory usage **on the node where the app is running**. If the usage exceeds the threshold, it will trigger a violation. This is useful to make sure that the node where the app is running has always enough free resources (e.g. to meet app usage spikes).

```yaml
policies:
  - type: node-resource-usage
    cpu_threshold_perc: 0.8
    memory_threshold_perc: 0.6
    memory_threshold: 200Mi
    exclude_app_resources: false
```

Parameters:

- **cpu_threshold_perc** (0-1): the **maximum** CPU usage percentage allowed. If the current CPU usage is above the threshold it will trigger the violation
- **memory_threshold_perc** (0-1): the **maximum** Memory usage percentage allowed. If the current Memory usage is above the threshold it will trigger the violation
- **memory_threshould**: the **minimal** amount of Memory that is free. If less Memory than the threshold is available, it will trigger the violation
- **exclude_app_resources**: indicates if the resources taken from the app itself should be excluded from the count (default is `true`). This parameter makes sense only at runtime and, hence, it is ignored by the MatchMaker


The **memory_threshold** parameters of this policy, is equivalent to the memory requirement:

```yaml
requirements:
  memory: 200Mi
```
If both are specified, the largest one is used.

The **default remediation action** is: `redeploy`.

### Devices Availability

This policy makes sure that devices specified in the `requirements` section remain available at runtime. If, at any moment, the device becomes unavailable, it will trigger a violation.

```yaml
  requirements:
    devices: icos.eu/video1
```

In the above example, if at any moment the camera attached to the device becomes unavailable (e.g. it is detached), a violation will be triggered.

The **default remediation action** is: `redeploy`.

## Components

### Components Reachability

This policy monitors the ICOS telemetry data sent automatically for each app deployed. When this data is not received for a given amount of time the policy is violated. If the telemetry data is not received it is very likely that the node has been disconnected or crashed, or the app has been manually removed.

**This policy can only be specified at component level.**

```yaml

policies:
  # short form
  - redeployOnLostTelemetry: 15m

  # normal form
  - type: redeployOnLostTelemetry
    timeout: 15m
```

Default `timeout` time is `5m`.

The **default remediation action** is: `redeploy`.

## COMPSS

### COMPSS Under Allocation

This policy works with COMPSS applications and it is used to scale-up the application replicas when the estimated time to finish (ETA) is above a given threshold.

```yaml
policies:
  - type: custom
    fromTemplate: compss-under-allocation
    remediation: scale-up
    variables:
      thresholdTimeSeconds: 120
      compssTask: provesOtel.example_task
```


## Custom

Beside the predefined policies presented above, it is possible to define custom policies. These policies are interpreted only by the Policy Manager and not by the Matchmaker, so they are only enforced at runtime, not a deployment time.

### Templated Policies

This policy is defined using one of the templates defined in the Policy Manager service. To overcome the complexity of writing [PromQL Expressions](#promql-expression-policies) for custom policies, the service defines a set of templates. For instance, all the policies presented in the previous sections have their own template defined in the Policy Manager.

```yaml

policies:
  - type: custom
    fromTemplate: <template-name>
    remediation: redeploy
    variables:
      var1: val1
      var2: val2
```

Parameters:

- **fromTemplate** specify the name of the template to use. Currently the following templates are defined:

  | Name                               | Variables                                                                                                                                                                                                    |
  | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
  | `compss-under-allocation`          |                                                                                                                                                                                                              |
  | `cpu-usage-host`                   |                                                                                                                                                                                                              |
  | `app-host-cpu-usage-perc`          |                                                                                                                                                                                                              |
  | `app-host-cpu-usage-perc-excl`     |                                                                                                                                                                                                              |
  | `app-host-memory-usage-perc`       |                                                                                                                                                                                                              |
  | `app-host-memory-usage-perc-excl`  |                                                                                                                                                                                                              |
  | `app-host-memory-avail-bytes-excl` |                                                                                                                                                                                                              |
  | `app-host-memory-avail-bytes`      |                                                                                                                                                                                                              |
  | `app-sca-score`                    | `threshold`: the minimum SCA score to not trigger the violation                                                                                                                                              |
  | `device-available`                 | `device`: the name of the device that should be available                                                                                                                                                    |
  | `redeploy-on-lost-telemetry`       |                                                                                                                                                                                                              |
  | `app-security-events`              | `maxEvents`: maximum number of events before triggering a violation (default: `0`) <br/>`severity`: tetragon severity level  (default: `.+`) <br/>`eventName`: filter by Tetragon event name (default: `.+`) |

- **remediation**: the name of the remediation action. It is mandatory
- **variables**: variables for customizing the template

!!! warning
    This section is under construction. More details for each template and the expected variables are coming

### PromQL Expression Policies

This type of policy is defined by a PromQL expression. The Policy Manager will monitor that expression and will trigger a violation when the expression value is outside the specified range. This is the most generic way of defining a policy for the Policy Manager. All other policies enforced by the Policy Manager can also be expressed in this form, although they could have a very complex expression.

```yaml
policies:
  - type: custom
    spec:
      expr: "... custom PROMQL expression ..."
    remediation: redeploy
    variables:
      myVar: myVal
```

The `spec.expr` parameter contains the expression that the Policy Manager should evaluate. It can be any PromQL expression that returns a value when it is violated. Under the hood, the Policy Manager uses this expression to define a [Prometheus Alerting Rule](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/) so any valid alerting rule is also a valid expression for this policy.


#### Placeholders 

In the expression it is possible to use some **placeholders** that will be replaced at policy creation time. They are delimited by the `{{ }}` sequence. Any variable can be referenced with a placeholder.

For instance in this policy:

```yaml
policies:
  - type: custom
    spec:
      expr: "...  > {{myVar}}"
    remediation: redeploy
    variables:
      myVar: myVal
```

the `{{myVar}}` placeholder will be replaced with `myVal` when the policy is created. This promotes the readability, maintenability and reusability of policies.

There are some placeholders that are predefined and replaces the labels that identify the subject of the policy (e.g. the app). This is useful, **and often the only way**, to define a policy that monitor a single app.

For instance, a policy that uses the following expression:

```yaml
policies:
- spec:
    expr: "container_cpu_usage > 0.8" 
```

will monitor that metric for *ALL* the apps! This is, in most of the cases, not the wanted behaviour. A way to filter the expression in PromQL is to use labels, for instance `container_cpu_usage{icos_app_name: "myapp", icos_app_instance: "myapp-123"}`. However the `icos_app_instance` value is unknown until the app is deployed!

For this reason, there exists two predefined placeholders that will be automatically replace with the "coordinates" of the app for which the policy is created:

- `subject_label_selector` is replaces with `icos_app_name: "<app_name>", icos_app_component: "<app_component(s)>, icos_app_instance: "<app_instance>"`. It is used for writing metric selectors. If the policy applies to multiple components, the `icos_app_component` will be `"comp1|comp2|..."` (if it applies to all components, it will be `"(.+)"`)
- `subject_label_list` is replaced with `icos_app_name, icos_app_component, icos_app_instance`. It is useful to shorten the writing of grouping or aggregation expressions

For instance the expression of the `device-available` templated policy is:

```
(
  node_mounted{resource_path=~"{{device}}"} offset 10m 
  * on(icos_host_id) group_left( {{subject_label_list}} ) 
  tlum_workload_info{ {{subject_label_selector}} } offset 10m
) 
unless
(
  node_mounted{resource_path=~"{{device}}"}
  * on(icos_host_id) group_left( {{subject_label_list}} )
  tlum_workload_info{ {{subject_label_selector}} }
)
```

It makes uses of the predefined placeholders plus the `{{device}}` placeholder that corresponds to the value of the `device` variable defined in the app descriptor.

# Common Parameters

All policies presented in this page allows some common parameters that can customize their behaviour.

```yaml

policies:
  
    # each policy can be assigned a custom name useful to identify the policy in the system
    # if no name is assigned, one will be generated automatically
  - name: cusotm name

    # allowed only in top-level "policies" section
    # restrict the policy scope to only a subset of components
    apply-to:
      - c1
      - c2
    
    # remediation action. It is "redeploy" by default in most of the policies
    remediation: redeploy

    # variables. They are replaced in the templated and PromQL Expression policies if expected
    variables: 
      var1: val1
      var2: val2
    
    # properties that can be used to customize some internal aspects of the Policy Manager
    properties:
      # wait the timeout before triggering a violation. If in the meantime, the condition for
      # the violation disappear, then the violation is not triggered
      pendingInterval: 5m

```
