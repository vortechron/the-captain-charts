# Larakube Examples

This directory contains example configurations for the Larakube Helm chart.

## Available Examples

### websocket-values.yaml

A complete example showing how to configure a Laravel application with WebSocket server support (like Laravel Reverb). This example includes:

- **WebSocket Server Configuration**: Dedicated WebSocket server worker with proper port exposure
- **Ingress Setup**: WebSocket-specific ingress with upgrade headers and timeout configurations
- **Environment Variables**: All necessary environment variables for Laravel Reverb
- **Health Checks**: TCP-based liveness and readiness probes for WebSocket connections
- **Autoscaling**: Horizontal Pod Autoscaler configuration for both web and WebSocket workers
- **Security**: TLS/SSL configuration with cert-manager integration
- **Multiple Workers**: Both WebSocket server and traditional queue workers

#### Usage

```bash
# Install with WebSocket support
helm install my-laravel-app renoki-co/larakube \
  --values examples/websocket-values.yaml \
  --set image.repository=your-laravel-app \
  --set image.tag=latest

# Upgrade existing installation
helm upgrade my-laravel-app renoki-co/larakube \
  --values examples/websocket-values.yaml \
  --set image.repository=your-laravel-app \
  --set image.tag=latest
```

#### Key Features Demonstrated

1. **WebSocket Server**: Runs `php artisan reverb:start` with proper configuration
2. **Dual Ingress**: Separate ingresses for HTTP traffic and WebSocket connections
3. **Environment Configuration**: Shows how to configure Laravel for WebSocket broadcasting
4. **Scaling**: Demonstrates autoscaling for high-traffic WebSocket applications
5. **Health Monitoring**: TCP probes to ensure WebSocket server availability

#### Prerequisites

- Laravel application with Reverb or similar WebSocket server package
- Nginx Ingress Controller with WebSocket support
- cert-manager for TLS certificate management (optional)
- Redis for session/cache/queue storage and WebSocket scaling

#### Customization

Before using this example, customize the following values:

- `image.repository` and `image.tag`: Your Laravel application image
- `websocket.app.*`: WebSocket application credentials
- `websocket.ingress.hosts`: Your WebSocket domain
- `ingress.hosts`: Your main application domain
- `secretEnvs`: Configure your secret environment variables

#### WebSocket Client Configuration

When using this configuration, your frontend WebSocket client should connect to:

```javascript
// For development
const websocket = new WebSocket('ws://ws.yourdomain.com/app/your-app-key');

// For production with TLS
const websocket = new WebSocket('wss://ws.yourdomain.com/app/your-app-key');
```

Make sure your Laravel broadcasting configuration matches the WebSocket server settings. 