# Awesome Helm Charts

Collection of Helm charts for awesome projects.

## Charts

### [distribution](charts/distribution)

A Helm chart for [distribution/distribution](https://github.com/distribution/distribution) (Docker Registry v2/v3) on Kubernetes.

See the [chart README](charts/distribution/README.md) for installation examples and full configuration reference.

**Features:**
- ✅ Ingress for external cluster access (default enabled)
- ✅ TLS and cert-manager support
- ✅ Basic auth via htpasswd
- ✅ Configurable storage backends (filesystem, S3, GCS, Azure)
- ✅ Persistent storage via PVC
- ✅ Health checks, resource limits, and security hardening
- ✅ Horizontal Pod Autoscaler support
