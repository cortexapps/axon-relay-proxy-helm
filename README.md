# Axon Relay Proxy Helm Chart

This Helm chart is used to deploy Axon Relay in proxy mode to enable Cortex plugins and workflows to access internal services securely.

## Overview

Axon Relay Proxy acts as a secure tunnel between Cortex and your internal services. When configured, you can make requests from Cortex plugins and workflows to `https://<alias>@relay-proxy.cortex.io/path` and Axon Relay will forward them to your internal services based on the configured rules.

## Configuration

The critical configuration options are:

- `proxy.alias`: The alias name that becomes part of the relay-proxy.cortex.io URL
- `proxy.configs`: The forwarding rules that define how requests are routed and authenticated
- `proxy.verbose`: Enable verbose logging for debugging

## Installation

0. (Recommended) Create a namespace in Kubernetes where you will run the proxy:
   ```bash
   kubectl create namespace my-axon-relay-proxy-namespace
   ```

1. Edit `values.yaml` to configure the proxy settings for your internal services:
    ```yaml
    proxy:
      alias: my-api-proxy
      verbose: false
      configs:
        - method: any
          path: /*
          origin: https://internal-api.company.com
          auth:
            scheme: bearer
            token: your_internal_api_token
    ```

2. Configure your Cortex API token. Axon relay needs this token to connect to Cortex.
    ```bash
    kubectl create secret generic axon-secrets --namespace <namespace> --from-literal=CORTEX_API_TOKEN=<your_cortex_token>
    ```

3. Install the chart:
    ```bash
    helm install axon-relay-proxy . --namespace <namespace>
    ```

    Replace `<namespace>` with your Kubernetes namespace.

## Usage in Plugins and Workflows

Once deployed, you can access your internal services from Cortex plugins and workflows using URLs like:

```
https://my-api-proxy@relay-proxy.cortex.io/api/users
https://my-api-proxy@relay-proxy.cortex.io/health/status
```

These requests will be forwarded to your configured origin server with any authentication you've specified.

## Configuration Reference

### Proxy Settings

- `method`: HTTP method to match (`GET`, `POST`, `PUT`, `DELETE`, or `any` for all methods)
- `path`: URL path pattern to match (supports wildcards like `/*`)
- `origin`: Target server base URL where requests are forwarded
- `auth`: Optional authentication configuration
  - `scheme`: Authentication type (`bearer`, `token`, or `basic`)
  - `token`: API token for bearer/token authentication
  - `username`/`password`: Credentials for basic authentication

### Example Configurations

**Simple proxy without authentication:**
```yaml
proxy:
  alias: internal-api
  configs:
    - method: any
      path: /*
      origin: https://api.internal.company.com
```

**API with bearer token authentication:**
```yaml
proxy:
  alias: secure-api
  configs:
    - method: any
      path: /*
      origin: https://secure.internal.company.com
      auth:
        scheme: bearer
        token: your_api_token_here
```

## Uninstallation

To uninstall the chart:
```bash
helm uninstall axon-relay-proxy --namespace <namespace>
```
