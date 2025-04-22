# Datadog Installation & Configuration Guide for Kubernetes

## 📚 Documentation
- [Datadog Agent in Kubernetes](https://docs.datadoghq.com/containers/kubernetes/)
- [Kubernetes Installation Guide](https://docs.datadoghq.com/containers/kubernetes/installation/?tab=datadogoperator)
- [Datadog Helm Chart (ArtifactHub)](https://artifacthub.io/packages/helm/datadog/datadog)
- [Official Helm Chart Values](https://github.com/DataDog/helm-charts/blob/main/charts/datadog/values.yaml)

---

## 🚀 Installation

### 1. Add Helm Repository
```bash
helm repo add datadog https://helm.datadoghq.com
```

### 2. Update Helm Repositories
```bash
helm repo update
```

### 3. Install Datadog
```bash
helm install datadog -n datadog \
  --set datadog.clusterName='aksenvironment01' \
  --set datadog.site='datadoghq.com' \
  --set datadog.clusterAgent.replicas='2' \
  --set datadog.clusterAgent.createPodDisruptionBudget='true' \
  --set datadog.kubeStateMetricsEnabled=true \
  --set datadog.kubeStateMetricsCore.enabled=true \
  --set datadog.logs.enabled=true \
  --set datadog.logs.containerCollectAll=true \
  --set datadog.apiKey='<REPLACE_WITH_API_KEY>' \
  --set datadog.processAgent.enabled=true \
  datadog/datadog --create-namespace
```

#### Minimal Installation
```bash
helm install datadog/datadog -n datadog \
  --set datadog.apiKey='<REPLACE_WITH_API_KEY>' \
  --set datadog.processAgent.enabled=true \
  datadog/datadog --create-namespace
```

---

## 🔧 Configuration

## 🔄 Update Configuration
To modify existing deployment:
```bash
helm upgrade datadog -n datadog -f values.yaml datadog/datadog
```

### 🛠 custom integration

to add [custom integration](https://github.com/DataDog/integrations-core)
```yaml

datadog:
  confd:
    nginx.yaml: |
      init_config:
      instances:
        - nginx_status_url: http://localhost/nginx_status

```

### 🛠 Pod Tolerations
- This is fairly easy to do using the Datadog Helm chart, as there is a specific section in the values.yaml file to add tolerations:

- Add tolerations for master nodes in `values.yaml`:
```yaml
tolerations:
  - key: node-role.kubernetes.io/master
    effect: NoSchedule
```

```yaml

  # agents.tolerations -- Allow the DaemonSet to schedule on tainted nodes (requires Kubernetes >= 1.6)
  tolerations: []
```

```yaml
# change it to
tolerations:
  -key: node-role.kubernetes.io/master
   effect: NoSchedule
```
### 🌐 Environment Variables

- **Case Scenario**
in some cases you can find the kubelet monitor in the agent status give you an error 
That error happens because we cannot verify the Kubelet certificates correctly. As this is not a production environment, let's tell the Datadog agent to skip the TLS verification by setting the environment variable called DD_KUBELET_TLS_VERIFY to false.

Setting environment variables in the values.yaml file is easy: there is a section to do just that:



**Use case:** If the kubelet monitor shows a TLS error in the agent status, you can skip TLS verification.

#### Default:
```yaml
env: []
#   - name: <ENV_VAR_NAME>
#     value: <ENV_VAR_VALUE>
```

#### Updated:
```yaml
env:
  - name: DD_KUBELET_TLS_VERIFY
    value: false
``
> 🔎 Reference: [Datadog Agent Docker Environment Variables](https://docs.datadoghq.com/agent/docker/?tab=standard#environment-variables)

```yaml
  ## The Datadog Agent supports many environment variables.
  ## ref: https://docs.datadoghq.com/agent/docker/?tab=standard#environment-variables
  env: []
  #   - name: <ENV_VAR_NAME>
  #     value: <ENV_VAR_VALUE>
```

```yaml
  ## The Datadog Agent supports many environment variables.
  ## ref: https://docs.datadoghq.com/agent/docker/?tab=standard#environment-variables
  env: 
     - name: DD_KUBELET_TLS_VERIFY
       value: false
```


Disable Kubelet TLS verification in `values.yaml`:
```yaml
env:
  - name: DD_KUBELET_TLS_VERIFY
    value: "false"
```

### 📝 Log Collection
Enable comprehensive logging in `values.yaml`:
```yaml
logs:
  enabled: true
  containerCollectAll: true
```


```yaml
  ## Enable logs agent and provide custom configs
  logs:
    # datadog.logs.enabled -- Enables this to activate Datadog Agent log collection

    ## ref: https://docs.datadoghq.com/agent/basic_agent_usage/kubernetes/#log-collection-setup
    enabled: false

    # datadog.logs.containerCollectAll -- Enable this to allow log collection for all containers

    ## ref: https://docs.datadoghq.com/agent/basic_agent_usage/kubernetes/#log-collection-setup
    containerCollectAll: false
```
- to

```yaml
  ## Enable logs agent and provide custom configs
  logs:
    # datadog.logs.enabled -- Enables this to activate Datadog Agent log collection

    ## ref: https://docs.datadoghq.com/agent/basic_agent_usage/kubernetes/#log-collection-setup
    enabled: true

    # datadog.logs.containerCollectAll -- Enable this to allow log collection for all containers

    ## ref: https://docs.datadoghq.com/agent/basic_agent_usage/kubernetes/#log-collection-setup
    containerCollectAll: true
```

### 📡 APM Configuration
Enable APM with socket volume in `values.yaml`:
```yaml
apm:
  enabled: true
  useSocketVolume: true
```


```yaml
   # datadog.apm.enabled -- Enable this to enable APM and tracing, on port 8126
    # DEPRECATED. Use datadog.apm.portEnabled instead

    ## ref: https://github.com/DataDog/docker-dd-agent#tracing-from-the-host
    enabled: false

    # datadog.apm.port -- Override the trace Agent port

    ## Note: Make sure your client is sending to the same UDP port.
    port: 8126

    # datadog.apm.useSocketVolume -- Enable APM over Unix Domain Socket
    # DEPRECATED. Use datadog.apm.socketEnabled instead

    ## ref: https://docs.datadoghq.com/agent/kubernetes/apm/
    useSocketVolume: false
```
- to

```yaml
   # datadog.apm.enabled -- Enable this to enable APM and tracing, on port 8126
    # DEPRECATED. Use datadog.apm.portEnabled instead

    ## ref: https://github.com/DataDog/docker-dd-agent#tracing-from-the-host
    enabled: true

    # datadog.apm.port -- Override the trace Agent port

    ## Note: Make sure your client is sending to the same UDP port.
    port: 8126

    # datadog.apm.useSocketVolume -- Enable APM over Unix Domain Socket
    # DEPRECATED. Use datadog.apm.socketEnabled instead

    ## ref: https://docs.datadoghq.com/agent/kubernetes/apm/
    useSocketVolume: true
```

### 🎛 Cluster Agent Setup

Another type of Datadog's Kubernetes agent is the `Cluster Agent`, which acts as a proxy between the API server and the agents and provides cluster-level monitoring data.

The `Cluster Agent` is disabled by default in the Datadog Helm chart default values. There is a section in the values.yaml file to enable the `Cluster Agent` and set the number of replicas:

- Enable and configure Cluster Agent in `values.yaml`:
```yaml
clusterAgent:
  enabled: true
  replicas: 2
  createPodDisruptionBudget: true
```

```yaml
clusterAgent:
  # clusterAgent.enabled -- Set this to false to disable Datadog Cluster Agent
  enabled: true

  # clusterAgent.shareProcessNamespace -- Set the process namespace sharing on the Datadog Cluster Agent
  shareProcessNamespace: false

  ## Define the Datadog Cluster-Agent image to work with
  image:
    # clusterAgent.image.name -- Cluster Agent image name to use (relative to `registry`)
    name: cluster-agent

    # clusterAgent.image.tag -- Cluster Agent image tag to use
    tag: 7.64.1

    # clusterAgent.image.digest -- Cluster Agent image digest to use, takes precedence over tag if specified
    digest: ""

    # clusterAgent.image.repository -- Override default registry + image.name for Cluster Agent
    repository:

    # clusterAgent.image.pullPolicy -- Cluster Agent image pullPolicy
    pullPolicy: IfNotPresent

    # clusterAgent.image.pullSecrets -- Cluster Agent repository pullSecret (ex: specify docker registry credentials)

    ## See https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod
    pullSecrets: []
    #   - name: "<REG_SECRET>"

    # clusterAgent.image.doNotCheckTag -- Skip the version and chart compatibility check

    ## By default, the version passed in clusterAgent.image.tag is checked
    ## for compatibility with the version of the chart.
    ## This boolean permits completely skipping this check.
    ## This is useful, for example, for custom tags that are not
    ## respecting semantic versioning.
    doNotCheckTag:  # false

  # clusterAgent.securityContext -- Allows you to overwrite the default PodSecurityContext on the cluster-agent pods.
  securityContext: {}

  containers:
    clusterAgent:
      # clusterAgent.containers.clusterAgent.securityContext -- Specify securityContext on the cluster-agent container.
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
    initContainers:
      # clusterAgent.containers.initContainers.securityContext -- Specify securityContext on the initContainers.
      securityContext: {}
      # clusterAgent.containers.initContainers.resources -- Resource requests and limits for the Cluster Agent init containers
      resources: {}
      #  requests:
      #    cpu: 100m
      #    memory: 200Mi
      #  limits:
      #    cpu: 100m
      #    memory: 200Mi

  # clusterAgent.command -- Command to run in the Cluster Agent container as entrypoint
  command: []

  # clusterAgent.token -- Cluster Agent token is a preshared key between node agents and cluster agent (autogenerated if empty, needs to be at least 32 characters a-zA-z)
  token: ""

  # clusterAgent.tokenExistingSecret -- Existing secret name to use for Cluster Agent token. Put the Cluster Agent token in a key named `token` inside the Secret
  tokenExistingSecret: ""

  # clusterAgent.replicas -- Specify the of cluster agent replicas, if > 1 it allow the cluster agent to work in HA mode.
  replicas: 1


```
---

## 🕵️ Agent Status Verification
Check agent status on a specific node:
```bash
kubectl exec -ti $(kubectl get pods -l app=datadog -o custom-columns=:.metadata.name --field-selector spec.nodeName=<NODE_NAME>) -- agent status
```
> Replace `<NODE_NAME>` with your actual node name

---

## 🚨 Important Notes
1. Always replace placeholder values (e.g., `<REPLACE_WITH_API_KEY>`)
2. For production environments, maintain TLS verification
3. Monitor resource allocations when enabling comprehensive logging
4. Ensure proper node affinity rules for cluster agent deployment

---



[//]: # (Maintain consistent indentation and formatting in YAML snippets)
```yaml
# Example values.yaml snippet
datadog:
  clusterName: "aksenvironment01"
  site: "datadoghq.com"
  logs:
    enabled: true
  apm:
    enabled: true
```

