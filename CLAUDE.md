# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Helm **umbrella chart** (`cloud-native-chart`) that bundles multiple cloud-native infrastructure subcharts for deployment to Kubernetes. It targets the `bitbuddy-dev` namespace.

## Common Commands

```bash
# Update/fetch chart dependencies
helm dependency update

# Validate the chart
helm lint .

# Render all templates locally (for inspection)
helm template cloud-native-chart .

# Render a specific template
helm template cloud-native-chart . --show-only templates/argocd-ingress.yaml

# Dry-run install to inspect what would be deployed
helm install cloud-native-chart . --dry-run --debug

# Install to cluster
helm install cloud-native-chart . -f values.yml

# Upgrade an existing release
helm upgrade cloud-native-chart . -f values.yml
```

## Architecture

### Umbrella Chart Pattern

`Chart.yaml` declares all subcharts as dependencies. The packaged `.tgz` files in `charts/` are the resolved versions (committed to the repo). Each subchart is **disabled by default** via `enabled: false` in `values.yml` — enable only what is needed per environment.

### Subcharts

| Alias | Chart | Purpose |
|-------|-------|---------|
| `kafka` | bitnami/kafka 26.2.0 | Message broker |
| `loki` | grafana/loki-stack 2.9.11 | Log aggregation (Grafana disabled — uses Prometheus Stack's Grafana) |
| `sonarqube` | sonarqube 10.5.0 | Code quality |
| `keycloak` | bitnami/keycloak 24.3.0 | Identity/auth |
| `jaeger` | jaegertracing/jaeger 3.0.9 | Distributed tracing (in-memory storage) |
| `argocd` | argoproj/argo-cd 7.3.4 | GitOps CD |

### Custom Templates

- `templates/kafka-config.yaml` — ConfigMap exposing `KAFKA_BOOTSTRAP_SERVERS` from `values.kafkaBootstrapServer`
- `templates/argocd-ingress.yaml` — Nginx Ingress for ArgoCD at `argo.local` (namespace: `bitbuddy-dev`, TLS secret: `argocd-tls`)

### Values Structure

Top-level keys in `values.yml` map directly to subchart aliases. To enable a component, set its `enabled: true`. The global `kafkaBootstrapServer` value is used by the `kafka-config` ConfigMap and can be consumed by other services.