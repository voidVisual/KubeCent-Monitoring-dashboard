# KubeCent-Monitoring-dashboard
Monitoring Dashboard in KubeCent delivers real-time visibility into Kubernetes cluster performance and cost analytics. It integrates with Prometheus and Grafana to visualize CPU, memory, and storage metrics, detect idle resources, track spending trends, and enable data-driven cost optimization.


## Table of contents <!-- omit in toc -->

- [Description](#description)
- [Features](#features)
- [Installation](#installation)
  - [Install manually](#install-manually)
  - [Install with ArgoCD](#install-with-argocd)
  - [Install with Helm values](#install-with-helm-values)
  - [Install as ConfigMaps](#install-as-configmaps)
  - [Install as ConfigMaps with Terraform](#install-as-configmaps-with-terraform)
  - [Install as GrafanaDashboard with Grafana Operator](#install-as-grafanadashboard-with-grafana-operator)
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










