# KubeCent-Monitoring-dashboard
Monitoring Dashboard in KubeCent delivers real-time visibility into Kubernetes cluster performance and cost analytics. It integrates with Prometheus and Grafana to visualize CPU, memory, and storage metrics, detect idle resources, track spending trends, and enable data-driven cost optimization.


## Table of contents <!-- omit in toc -->

- [Description](#description)
- [Features](#features)
- [Installation](#installation)
  - [Install manually](#install-manually)
  - [Install via grafana.com](#install-via-grafanacom)
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








