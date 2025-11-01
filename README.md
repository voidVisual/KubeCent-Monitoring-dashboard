# KubeCent-Monitoring-dashboard
Monitoring Dashboard in KubeCent delivers real-time visibility into Kubernetes cluster performance and cost analytics. It integrates with Prometheus and Grafana to visualize CPU, memory, and storage metrics, detect idle resources, track spending trends, and enable data-driven cost optimization.


## Table of contents <!-- omit in toc -->

- [Description](#description)
- [Features](#features)
- [Installation](#installation)
  - [Install manually](#install-manually)
  - [Install with ArgoCD](#install-with-argocd)
  - [Install as ConfigMaps](#install-as-configmaps)
- [Known issue(s)](#known-issues)
  - [Broken panels due to a too-high resolution](#broken-panels-due-to-a-too-high-resolution)
  - [Broken panels on k8s-views-nodes when a node changes its IP address](#broken-panels-on-k8s-views-nodes-when-a-node-changes-its-ip-address)
  - [Broken panels on k8s-views-nodes due to the nodename label](#broken-panels-on-k8s-views-nodes-due-to-the-nodename-label)
- [Contributing](#contributing)


## Description
Monitoring Dashboard in KubeCent delivers real-time visibility into Kubernetes cluster performance and cost analytics. It integrates with Prometheus and Grafana to visualize CPU, memory, and storage metrics, detect idle resources, track spending trends, and enable data-driven cost optimization.

This repository contains a modern set of [Grafana](https://github.com/grafana/grafana) dashboards for [Kubernetes](https://github.com/kubernetes/kubernetes).\
They are inspired by many other dashboards from `kubernetes-mixin` and `grafana.com`.

## Features

These dashboards are made and tested for the [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) chart, but they should work well with others as soon as you have [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics) and [prometheus-node-exporter](https://github.com/prometheus/node_exporter) installed on your Kubernetes cluster.

They are not backward compatible with older Grafana versions because they try to take advantage of Grafana's newest features like:

- `gradient mode` introduced in Grafana 8.1 ([Grafana Blog post](https://grafana.com/blog/2021/09/10/new-in-grafana-8.1-gradient-mode-for-time-series-visualizations-and-dynamic-panel-configuration/))
- `time series` visualization panel introduced in Grafana 7.4 ([Grafana Blog post](https://grafana.com/blog/2021/02/10/how-the-new-time-series-panel-brings-major-performance-improvements-and-new-visualization-features-to-grafana-7.4/))
- `$__rate_interval` variable introduced in Grafana 7.2 ([Grafana Blog post](https://grafana.com/blog/2020/09/28/new-in-grafana-7.2-__rate_interval-for-prometheus-rate-queries-that-just-work/))

They also have a `Prometheus Datasource` variable so they will work on a federated Grafana instance.


## Installation

In most cases, you will need to clone this repository (or your fork):

```terminal
git clone https://github.com/voidVisual/KubeCent-Monitoring-dashboard.git
cd KubeCent-Monitoring-dashboard
```

If you plan to deploy these dashboards using [ArgoCD](#install-with-argocd), [ConfigMaps](#install-as-configmaps) or [Terraform](#install-as-configmaps-with-terraform), you will also need to enable and configure the `dashboards sidecar` on the Grafana Helm chart to get the dashboards loaded in your Grafana instance:

```yaml
# kube-prometheus-stack values
grafana:
  sidecar:
    dashboards:
      enabled: true
      defaultFolderName: "General"
      label: grafana_dashboard
      labelValue: "1"
      folderAnnotation: grafana_folder
      searchNamespace: ALL
      provider:
        foldersFromFilesStructure: true
```
### Install manually

On the WebUI of your Grafana instance, put your mouse over the `+` sign on the left menu, then click on `Import`.\
Once you are on the Import page, you can upload the JSON files one by one from your local copy using the `Upload JSON file` button.

### Install with ArgoCD

If you are using ArgoCD, this will deploy the dashboards in the default project of ArgoCD:

```terminal
kubectl apply -f argocd-app.yml
```

You will also need to enable and configure the Grafana `dashboards sidecar` as described in [Installation](#installation).

### Install as ConfigMaps

Grafana dashboards can be provisioned as Kubernetes ConfigMaps if you configure the [dashboard sidecar](https://github.com/grafana/helm-charts/blob/main/charts/grafana/values.yaml#L667) available on the official [Grafana Helm Chart](https://github.com/grafana/helm-charts/tree/main/charts/grafana).

To build the ConfigMaps and output them on STDOUT :

```terminal
kubectl kustomize .
```

*Note: no namespace is set by default, you can change that in the `kustomization.yaml` file.*

To build and deploy them directly on your Kubernetes cluster :

```terminal
kubectl apply -k . -n monitoring
```

You will also need to enable and configure the Grafana `dashboards sidecar` as described in [Installation](#installation).

*Note: you can change the namespace if needed.*


### Broken panels due to a too-high resolution


some panels were broken because the default value of the `$resolution` variable was too low. The root cause hasn't been identified precisely, but he was using Grafana Agent & Grafana Mimir. Changing the `$resolution` variable to a higher value (a lower resolution) will likely solve the issue.
To make the fix permanent, you can configure the `Scrape interval` in your Grafana Datasource to a working value for your setup.


### Broken panels on k8s-views-nodes when a node changes its IP address

To make this dashboard more convenient, there's a small variable hack to display `node` instead of `instance`.
Because of that, some panels could lack data when a node changes its IP address.

No easy fix for this scenario yet, but it should be a corner case for most people.
Feel free to reopen the issue if you have ideas to fix this.

### Broken panels on k8s-views-nodes due to the nodename label

The `k8s-views-nodes` dashboard will have many broken panels if the `node` label from `kube_node_info` doesn't match the `nodename` label from `node_uname_info`.

This situation can happen on certain deployments of the node exporter running inside Kubernetes(e.g. via a `DaemonSet`), where `nodename` takes a different value than the node name as understood by the Kubernetes API.

Below are some ways to relabel the metric to force the `nodename` label to the appropriate value, depending on the way the collection agent is deployed:

#### Directly through the Prometheus configuration file <!-- omit in toc -->

Assuming the node exporter job is defined through `kubernetes_sd_config`, you can take advantage of the internal discovery labels and fix this by adding the following relabeling rule to the job:

```yaml
# File: prometheus.yaml
scrape_configs:
- job_name: node-exporter
  relabel_configs:
  # Add this
  - action: replace
    source_labels: [ __meta_kubernetes_pod_node_name]
    target_label: nodename
```

#### Through a `ServiceMonitor` <!-- omit in toc -->

If using the Prometheus operator or the Grafana agent in operator mode, the scrape job should instead be configured via a `ServiceMonitor` that will dynamically edit the Prometheus configuration file. In that case, the relabeling has a slightly different syntax:

```yaml
# File: service-monitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
  endpoints:
  - port: http-metrics
    relabelings:
    # Add this
    - action: replace
      sourceLabels: [ __meta_kubernetes_node_name]
      targetLabel: nodename
```

As a convenience, if using the [kube-prometheus-stack helm chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack), this added rule can be directly specified in your values.yaml:

```yaml
# File: kube-prometheus-stack-values.yaml
prometheus-node-exporter:
  prometheus:
    monitor:
      relabelings:
      - action: replace
        sourceLabels: [__meta_kubernetes_pod_node_name]
        targetLabel: nodename
```

#### With Grafana Agent Flow mode <!-- omit in toc -->

The Grafana Agent can [bundle its own node_exporter](https://grafana.com/docs/agent/v0.33/flow/reference/components/prometheus.exporter.unix/). In that case, relabeling can be done this way:

```river
prometheus.exporter.unix {
}

prometheus.scrape "node_exporter" {
  targets = prometheus.exporter.unix.targets
  forward_to = [prometheus.relabel.node_exporter.receiver]

  job_name = "node-exporter"
}

prometheus.relabel "node_exporter" {
  forward_to = [prometheus.remote_write.sink.receiver]

  rule {
    replacement = env("HOSTNAME")
    target_label = "nodename"
  }

  rule {
    # The default job name is "integrations/node_exporter" and needs to be replaced
    replacement = "node-exporter"
    target_label = "job"
  }
}
```
## Contributing

Feel free to contribute to this project:

- Give a GitHub ⭐ if you like it
- Create an [Issue](https://github.com/voidVisual/KubeCent-Monitoring-dashboard/issues) to make a feature request, report a bug or share an idea.
- Create a [Pull Request](https://github.com/voidVisual/KubeCent-Monitoring-dashboard/pulls) if you want to share code or anything useful to this project.











