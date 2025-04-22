# Installation Guide for Datadog in Kubernetes Cluster

## Step 1: Add the Datadog Helm Repository

To begin, add the Datadog Helm repository to your Helm configuration:

```bash
helm repo add datadog https://helm.datadoghq.com
```

## Step 2: Update Helm Repositories

Next, update your Helm repositories to ensure you have the latest charts:

```bash
helm repo update
```

## Step 3: Install Datadog

Now, you can install Datadog using the following command. Make sure to replace the placeholders with your actual values:

```bash
helm install datadog -n datadog \  # Install the Datadog chart in the 'datadog' namespace
  --set datadog.clusterName='aksenvironment01' \  # Set the name of the Kubernetes cluster
  --set datadog.site='datadoghq.com' \  # Specify the Datadog site to send data to
  --set datadog.clusterAgent.replicas='2' \  # Set the number of replicas for the Datadog Cluster Agent
  --set datadog.clusterAgent.createPodDisruptionBudget='true' \  # Enable Pod Disruption Budget for the Cluster Agent
  --set datadog.kubeStateMetricsEnabled=true \  # Enable Kubernetes state metrics collection
  --set datadog.kubeStateMetricsCore.enabled=true \  # Enable core Kubernetes state metrics
  --set datadog.logs.enabled=true \  # Enable log collection
  --set datadog.logs.containerCollectAll=true \  # Collect logs from all containers
  --set datadog.apiKey='<replace it with your API>' \  # Set your Datadog API key for authentication
  --set datadog.processAgent.enabled=true \  # Enable the Datadog Process Agent for process monitoring
  datadog/datadog --create-namespace  # Specify the Datadog chart and create the namespace if it doesn't exist
```
