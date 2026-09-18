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

### [keycloak](charts/keycloak)

A Helm chart for [Keycloak](https://www.keycloak.org/) on Kubernetes.

See the [chart README](charts/keycloak/README.md) for installation examples and full configuration reference.

**Features:**
- ✅ Customizable image registry — supports any Keycloak image source (quay.io, Docker Hub, private registries, Aliyun mirrors, etc.)
- ✅ Ingress for external cluster access (default enabled)
- ✅ TLS and cert-manager support
- ✅ Admin user bootstrapping with password or existing Secret
- ✅ Production-grade database support (PostgreSQL, MySQL, MariaDB, MSSQL, Oracle)
- ✅ Built-in H2 database for development
- ✅ Health checks (startup, liveness, readiness)
- ✅ Hostname configuration for proper redirects and token issuers
- ✅ Horizontal Pod Autoscaler support
- ✅ Resource limits and security hardening
