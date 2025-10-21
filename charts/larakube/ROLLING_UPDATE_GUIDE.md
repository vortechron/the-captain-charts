# Rolling Update Configuration Guide

## Overview

Larakube Helm chart v1.2.0 provides comprehensive rolling update configuration for all deployments including web, worker, and nginx components. Rolling updates allow you to deploy new versions of your application with zero downtime by gradually replacing old pods with new ones.

## Configuration Options

### Web Deployment

The web deployment comes with pre-configured rolling update settings that can be customized:

```yaml
web:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

**Parameters:**
- `type`: Update strategy type (`RollingUpdate` or `Recreate`)
- `maxSurge`: Maximum number of pods that can be created above the desired replica count
- `maxUnavailable`: Maximum number of pods that can be unavailable during the update

### Worker Deployments

Each worker type (websocket and default) has individual rolling update configuration:

```yaml
worker:
  websocket:
    updateStrategy:
      type: RollingUpdate
      rollingUpdate:
        maxSurge: 1
        maxUnavailable: 0
  
  default:
    updateStrategy:
      type: Recreate  # Default workers use Recreate strategy
```

### Nginx Deployment

Nginx proxy deployments support rolling updates:

```yaml
nginx:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

## Usage Examples

### Basic Rolling Update (Zero Downtime)

For web applications that require zero downtime:

```yaml
web:
  replicaCount: 3
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

This configuration ensures:
- One additional pod can be created during updates
- No pods are taken down until the new pod is ready
- Guarantees zero downtime for your application

### Fast Rolling Update

For faster deployments when brief downtime is acceptable:

```yaml
web:
  replicaCount: 3
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
```

This allows:
- Up to 2 additional pods during updates
- 1 pod can be unavailable during the process
- Faster deployment at the cost of potential brief downtime

### WebSocket Worker Configuration

For WebSocket workers that maintain persistent connections:

```yaml
worker:
  websocket:
    enabled: true
    replicaCount: 2
    updateStrategy:
      type: RollingUpdate
      rollingUpdate:
        maxSurge: 1
        maxUnavailable: 0
```

This ensures WebSocket connections are not dropped during deployments.

### Queue Worker Configuration

For background job workers where brief interruption is acceptable:

```yaml
worker:
  default:
    enabled: true
    replicaCount: 3
    updateStrategy:
      type: RollingUpdate
      rollingUpdate:
        maxSurge: 1
        maxUnavailable: 1
```

## Best Practices

### 1. Health Checks

Always configure proper health checks to ensure rolling updates work correctly:

```yaml
web:
  readinessProbe:
    httpGet:
      path: /health
      port: 8000
    initialDelaySeconds: 30
    periodSeconds: 10
  
  livenessProbe:
    httpGet:
      path: /health
      port: 8000
    initialDelaySeconds: 60
    periodSeconds: 10
```

### 2. Resource Allocation

Ensure your cluster has enough resources for the maxSurge pods:

```yaml
web:
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 500m
      memory: 512Mi
```

### 3. Pod Disruption Budgets

Use PodDisruptionBudgets alongside rolling updates:

```yaml
web:
  pdb:
    enabled: true
    maxUnavailable: 1
```

### 4. Gradual Rollout

For critical applications, consider a gradual rollout:

```yaml
web:
  replicaCount: 10
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
```

This updates only 3 pods at a time (2 new + 1 unavailable) out of 10 total.

## Monitoring Rolling Updates

### Check Deployment Status

```bash
kubectl rollout status deployment/myapp-web
```

### View Rollout History

```bash
kubectl rollout history deployment/myapp-web
```

### Rollback if Needed

```bash
kubectl rollout undo deployment/myapp-web
```

## Troubleshooting

### Common Issues

1. **Pods not starting**: Check resource limits and readiness probes
2. **Slow rollouts**: Increase maxSurge or decrease readiness probe delays
3. **Failed deployments**: Verify health check endpoints and resource availability

### Debugging Commands

```bash
# Check pod status
kubectl get pods -l app.kubernetes.io/component=web

# View pod logs
kubectl logs -l app.kubernetes.io/component=web

# Describe deployment
kubectl describe deployment myapp-web
```

## Migration from Previous Versions

If upgrading from larakube v1.1.0 or earlier, the rolling update configuration is backward compatible. Your existing deployments will continue to work with the default settings.

To apply new rolling update settings:

```bash
helm upgrade myapp ./charts/larakube -f values.yaml
```

## Version History

- **v1.2.0**: Added comprehensive rolling update configuration for all deployments
- **v1.1.0**: Basic rolling update support for web deployments