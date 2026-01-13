# Alertmanager Deployment

Alertmanager handles alerts sent by Prometheus and routes them to the correct receiver.

## Structure

```
alertmanager/
├── base/                    # Base Alertmanager configuration
│   ├── namespace.yaml       # monitoring namespace
│   ├── configmap.yaml      # Alertmanager configuration
│   ├── pvc.yaml            # Persistent storage (5Gi)
│   ├── deployment.yaml     # Alertmanager deployment
│   ├── service.yaml        # ClusterIP service
│   ├── ingress.yaml        # Ingress configuration
│   └── kustomization.yaml  # Base kustomization
├── overlays/
│   ├── stage/              # Stage environment
│   │   └── kustomization.yaml
│   └── prod/               # Production environment
│       └── kustomization.yaml
└── README.md
```

## Environments

### Stage
- **URL:** https://alertmanager.apps-stage.in.hinisoft.com
- **Ingress:** nginx-private (internal only, via VPN)
- **Replicas:** 1
- **Resources:** 100m CPU / 128Mi Memory
- **Storage:** 5Gi

### Production
- **URL:** https://alertmanager.apps-prod.in.hinisoft.com
- **Ingress:** nginx-public (external access)
- **Replicas:** 2 (HA with clustering)
- **Resources:** 200m CPU / 256Mi Memory
- **Storage:** 10Gi

## Deployment

### Stage Environment
```bash
kubectl apply -k overlays/stage
```

### Production Environment
```bash
kubectl apply -k overlays/prod
```

## Features

- Alertmanager v0.30.1
- Persistent storage with Longhorn
- Automatic SSL certificates via cert-manager + Let's Encrypt
- Health checks (liveness and readiness probes)
- Resource limits and requests
- Clustering support for HA in production

## Configuration

The default configuration includes:
- Route grouping by alertname, cluster, and service
- Separate receivers for critical and warning alerts
- Inhibit rules to suppress warning alerts when critical alerts fire

### Customizing Receivers

Edit the ConfigMap to add notification receivers (Slack, PagerDuty, Email, etc.):

```yaml
receivers:
  - name: 'slack-notifications'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/...'
        channel: '#alerts'
```

## Integration with Prometheus

Add Alertmanager to Prometheus configuration:

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - stage-alertmanager.monitoring.svc.cluster.local:9093
```

## SSL Certificates

Certificates are automatically issued by cert-manager using Let's Encrypt with GoDaddy DNS-01 challenge.

## Accessing Alertmanager

### Stage (Internal)
1. Connect to VPN
2. Access: https://alertmanager.apps-stage.in.hinisoft.com

### Production (External)
Access directly: https://alertmanager.apps-prod.in.hinisoft.com
