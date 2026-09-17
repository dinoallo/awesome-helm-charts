# distribution

[![Artifact Hub](https://img.shields.io/badge/Artifact%20Hub-distribution-4DB3B3)](https://artifacthub.io/packages/search?repo=awesome-helm-charts)

A Helm chart for [distribution/distribution](https://github.com/distribution/distribution) (Docker Registry v2/v3) on Kubernetes.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2+
- Ingress controller (e.g., [ingress-nginx](https://kubernetes.github.io/ingress-nginx/)) — required when Ingress is enabled

## Installing

### Basic installation

Update the host and install:

```bash
helm install my-registry ./charts/distribution \
  --set ingress.hosts[0].host=registry.mycompany.com
```

### With TLS and basic auth (recommended)

Create a TLS secret first, then install with authentication:

```bash
# Create TLS secret (or use cert-manager)
kubectl create secret tls registry-tls \
  --cert=tls.crt \
  --key=tls.key

# Generate htpasswd entry
HTPASSWD=$(htpasswd -Bbn admin mypassword)

helm install my-registry ./charts/distribution \
  --set ingress.hosts[0].host=registry.mycompany.com \
  --set ingress.tls[0].secretName=registry-tls \
  --set ingress.tls[0].hosts[0]=registry.mycompany.com \
  --set "secrets.htpasswd=${HTPASSWD}"
```

> ⚠️ **Important:** When using basic auth, TLS must be configured to avoid sending credentials in plaintext.

### With cert-manager (automatic TLS)

```bash
helm install my-registry ./charts/distribution \
  --set ingress.hosts[0].host=registry.mycompany.com \
  --set ingress.annotations."cert-manager\.io/cluster-issuer"=letsencrypt-prod \
  --set ingress.tls[0].secretName=registry-tls \
  --set ingress.tls[0].hosts[0]=registry.mycompany.com \
  --set "secrets.htpasswd=$(htpasswd -Bbn admin mypassword)"
```

### With custom storage class

```bash
helm install my-registry ./charts/distribution \
  --set ingress.hosts[0].host=registry.mycompany.com \
  --set persistence.storageClass=fast-ssd \
  --set persistence.size=50Gi
```

### With S3/GCS/Azure backend storage

Override the registry config to use cloud storage:

```bash
helm install my-registry ./charts/distribution \
  --set ingress.hosts[0].host=registry.mycompany.com \
  --set persistence.enabled=false \
  --set-json 'config.storage={"s3":{"bucket":"my-registry","region":"us-east-1","encrypt":true}}'
```

### Enable image deletion and debug metrics

```bash
helm install my-registry ./charts/distribution \
  --set ingress.hosts[0].host=registry.mycompany.com \
  --set extraEnv[0].name=REGISTRY_STORAGE_DELETE_ENABLED \
  --set extraEnv[0].value=true \
  --set-json 'config.http.debug.prometheus.enabled=true'
```

### Minimal (no persistence, no ingress)

For local testing:

```bash
helm install my-registry ./charts/distribution \
  --set ingress.enabled=false \
  --set persistence.enabled=false
```

## Configuration

See [values.yaml](values.yaml) for the full reference, or use [values-example.yaml](values-example.yaml) as a starting point.

### Ingress parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `ingress.enabled` | `true` | Enable Ingress |
| `ingress.className` | `nginx` | Ingress class name |
| `ingress.hosts[0].host` | `registry.example.com` | Ingress hostname |
| `ingress.hosts[0].paths[0].path` | `/` | Path prefix |
| `ingress.hosts[0].paths[0].pathType` | `Prefix` | Path type |
| `ingress.tls` | `[]` | TLS configuration |
| `ingress.annotations` | `{proxy-body-size: "0", proxy-request-buffering: "off"}` | Additional annotations |

### Storage parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `persistence.enabled` | `true` | Enable persistent storage |
| `persistence.size` | `10Gi` | PVC size |
| `persistence.storageClass` | `""` | Storage class (empty = default; set to `"-"` to disable) |
| `persistence.accessModes` | `[ReadWriteOnce]` | Access modes |
| `persistence.existingClaim` | `""` | Use existing PVC name |

### Registry configuration

The full `config.yml` is managed via the `config` value. See the [official docs](https://github.com/distribution/distribution/blob/main/docs/configuration.md) for all options.

Default config:

```yaml
config:
  version: 0.1
  log:
    level: info
    formatter: text
    fields:
      service: registry
  storage:
    cache:
      blobdescriptor: inmemory
    filesystem:
      rootdirectory: /var/lib/registry
    delete:
      enabled: false
  http:
    addr: :5000
    headers:
      X-Content-Type-Options: [nosniff]
    debug:
      addr: 127.0.0.1:5001
      prometheus:
        enabled: false
        path: /metrics
  health:
    storagedriver:
      enabled: true
      interval: 10s
      threshold: 3
```

### Basic auth

Set `secrets.htpasswd` to an htpasswd string (e.g., output of `htpasswd -Bbn admin mypassword`). The chart automatically configures `REGISTRY_AUTH` environment variables to enable authentication. TLS is required when using basic auth.

## Verifying the installation

```bash
# Check pods
kubectl get pods -l app.kubernetes.io/name=distribution

# Test the registry API (without auth)
curl -i http://<ingress-host>/v2/

# Test with auth
curl -i -u admin:mypassword https://<ingress-host>/v2/

# Should return: 200 {"repositories":[]}
```

## Uninstalling

```bash
helm uninstall my-registry
```

Note: PVCs are not automatically deleted. Run `kubectl delete pvc <pvc-name>` to release storage.
