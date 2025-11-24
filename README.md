# code-server Helm Chart

A Helm chart for deploying [code-server](https://github.com/coder/code-server) (VS Code in the browser) on Kubernetes.

## Features

- 🔐 **Auto-generated secrets** - Passwords are automatically generated on first install
- 🔄 **Persistent secrets** - Passwords are preserved across upgrades
- 🚀 **LoadBalancer service** - Easy external access
- 📊 **Health checks** - Liveness and readiness probes configured
- 🎛️ **Highly configurable** - Supports ingress, autoscaling, and more

## Installation

### Quick Start

```bash
# Create namespace
kubectl create namespace code-server

# Install chart with auto-generated password
helm install code-server . --namespace code-server

# Get the auto-generated password
kubectl get secret code-server-coder-project-secret -n code-server \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

### Custom Password

```bash
# Install with custom password
helm install code-server . \
  --namespace code-server \
  --set auth.password='MySecurePassword123'
```

### Using Existing Secret

```bash
# Create secret first
kubectl create secret generic my-code-server-secret \
  --from-literal=password='MyPassword' \
  --from-literal=sudo-password='MySudoPassword' \
  --namespace code-server

# Install referencing existing secret
helm install code-server . \
  --namespace code-server \
  --set auth.existingSecret=my-code-server-secret
```

## Configuration

### Authentication

| Parameter | Description | Default |
|-----------|-------------|---------|
| `auth.password` | Custom password (leave empty for auto-generated) | `""` |
| `auth.sudoPassword` | Custom sudo password (leave empty for auto-generated) | `""` |
| `auth.existingSecret` | Use existing secret instead of creating one | `""` |
| `auth.existingSecretPasswordKey` | Key in existing secret for password | `"password"` |
| `auth.existingSecretSudoKey` | Key in existing secret for sudo password | `"sudo-password"` |

## How Secret Generation Works

### First Install
- If `auth.password` is empty, a random 32-character password is generated
- If `auth.sudoPassword` is empty, a random 32-character password is generated
- Secret is created with name: `<release-name>-coder-project-secret`

### Upgrades
- Existing passwords are preserved using Helm's `lookup` function
- Only generates new passwords if the secret doesn't exist

### Custom Passwords
- Set `auth.password` and/or `auth.sudoPassword` in values.yaml
- Or use `--set` during installation

### Existing Secrets
- Set `auth.existingSecret` to use a pre-created secret
- Chart will not create a new secret if this is set

## Accessing code-server

### Get External IP (LoadBalancer)

```bash
kubectl get svc -n code-server
export SERVICE_IP=$(kubectl get svc code-server-coder-project -n code-server \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo "Access at: http://${SERVICE_IP}:8080"
```

### Port-Forward (Testing)

```bash
kubectl port-forward -n code-server svc/code-server-coder-project 8080:8080
# Access at: http://localhost:8080
```

## Uninstall

```bash
helm uninstall code-server -n code-server
kubectl delete namespace code-server
```
