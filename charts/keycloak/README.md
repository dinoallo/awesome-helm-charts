# Keycloak Helm Chart

A Helm chart for deploying [Keycloak](https://www.keycloak.org/) on Kubernetes.

[Keycloak](https://www.keycloak.org/) is an open-source identity and access management (IAM) solution, providing single sign-on (SSO), authentication, and authorization for modern applications and services.

## Features

- ✅ Customizable image registry — use any Keycloak-compatible image (quay.io, Docker Hub, private/aliyun mirrors, etc.)
- ✅ Ingress for external cluster access (default enabled)
- ✅ TLS and cert-manager support
- ✅ Admin user bootstrapping
- ✅ Production-grade database configuration (PostgreSQL, MySQL, MariaDB, MSSQL, Oracle)
- ✅ Built-in H2 database for development/testing
- ✅ Health checks (startup, liveness, readiness via Keycloak's `/health` endpoints)
- ✅ Resource limits and security hardening
- ✅ Horizontal Pod Autoscaler support
- ✅ Hostname configuration for proper redirects and token issuers
- ✅ Extra environment variables for full Keycloak configuration

## Quick Start

```bash
# Install Keycloak (development mode, built-in H2 database)
helm install my-keycloak ./charts/keycloak \
  --set ingress.hosts[0].host=keycloak.example.com \
  --set auth.adminPassword=myadminpassword

# Production: use an external database
helm install my-keycloak ./charts/keycloak -f values-example.yaml
```

## Configuration

### Image

The chart supports any Keycloak image registry:

```yaml
image:
  # Default: quay.io/keycloak/keycloak
  # Docker Hub: docker.io/keycloak/keycloak
  # Aliyun mirror: registry.cn-hangzhou.aliyuncs.com/keycloak/keycloak
  # Private registry: your-registry.example.com/keycloak/keycloak
  repository: quay.io/keycloak/keycloak
  tag: "26.1"
  pullPolicy: IfNotPresent
```

For pulling from private registries, configure `imagePullSecrets`:

```yaml
imagePullSecrets:
  - name: my-registry-cred
```

### Database

Keycloak supports multiple database backends. The `database.vendor` field accepts:

| Vendor     | Value       |
|------------|-------------|
| H2 (built-in, dev only) | `dev-file` |
| PostgreSQL | `postgres`  |
| MySQL      | `mysql`     |
| MariaDB    | `mariadb`   |
| MSSQL      | `mssql`     |
| Oracle     | `oracle`    |

For production, **always use an external database**:

```yaml
database:
  vendor: postgres
  hostname: my-postgresql.default.svc.cluster.local
  port: 5432
  database: keycloak
  username: keycloak
  password: "your-password"
  # schema: public
```

You can also reference an existing Kubernetes Secret for the database password:

```yaml
database:
  vendor: postgres
  hostname: my-postgresql.default.svc.cluster.local
  port: 5432
  database: keycloak
  username: keycloak
  existingPasswordSecret: keycloak-db-password  # key: password
```

For vendor-specific database TLS settings, use `extraEnv`:

```yaml
extraEnv:
  - name: KC_DB_URL_PROPERTIES
    value: "?sslmode=require"
```

### Admin Credentials

```yaml
auth:
  adminUser: admin
  adminPassword: "your-strong-password"
  # Or use existing Secrets:
  # existingAdminPasswordSecret: keycloak-admin-password  # key: admin-password
  # existingAdminUserSecret: keycloak-admin-user          # key: admin-user
```

### Hostname Configuration

```yaml
hostname:
  strict: false
  url: https://keycloak.mycompany.com
  adminUrl: https://keycloak-admin.mycompany.com
```

This sets the `KC_HOSTNAME` environment variable, which Keycloak uses to generate redirect URIs and token issuers.

### Full Configuration Reference

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | Image repository | `quay.io/keycloak/keycloak` |
| `image.tag` | Image tag | `26.1` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `imagePullSecrets` | Image pull secrets | `[]` |
| `nameOverride` | Override chart name | `""` |
| `fullnameOverride` | Override full name | `""` |
| `serviceAccount.create` | Create service account | `true` |
| `serviceAccount.annotations` | Service account annotations | `{}` |
| `serviceAccount.name` | Service account name | `""` |
| `rbac.create` | Create RBAC for cache stack | `false` |
| `podAnnotations` | Pod annotations | `{}` |
| `podLabels` | Additional pod labels | `{}` |
| `podSecurityContext` | Pod security context | `{fsGroup: 1000}` |
| `securityContext` | Container security context | (run as non-root, drop all caps) |
| `service.type` | Service type | `ClusterIP` |
| `service.port` | HTTP port | `8080` |
| `service.managementPort` | Management/health port | `9000` |
| `service.managementPortEnabled` | Expose management port on Service | `true` |
| `service.annotations` | Service annotations | `{}` |
| `ingress.enabled` | Enable ingress | `true` |
| `ingress.className` | Ingress class name | `nginx` |
| `ingress.annotations` | Ingress annotations | (see values.yaml) |
| `ingress.hosts` | Ingress hosts | `[keycloak.example.com]` |
| `ingress.tls` | TLS configuration | `[]` |
| `updateStrategy` | Deployment update strategy | `RollingUpdate` |
| `resources` | CPU/memory resources | (see values.yaml) |
| `livenessProbe` | Liveness probe config | (HTTP GET /health) |
| `readinessProbe` | Readiness probe config | (HTTP GET /health/ready) |
| `startupProbe` | Startup probe config | (HTTP GET /health/ready) |
| `persistence.enabled` | Enable PVC (H2 only) | `false` |
| `persistence.size` | PVC size | `10Gi` |
| `persistence.storageClass` | Storage class | `""` |
| `database.vendor` | Database vendor | `dev-file` |
| `database.hostname` | Database hostname | `""` |
| `database.port` | Database port | `5432` |
| `database.database` | Database name | `keycloak` |
| `database.username` | Database username | `keycloak` |
| `database.password` | Database password | `""` |
| `database.existingPasswordSecret` | Existing secret for DB password | `""` |
| `database.schema` | Database schema | `""` |
| `auth.adminUser` | Admin username | `admin` |
| `auth.adminPassword` | Admin password | `""` |
| `auth.existingAdminPasswordSecret` | Existing secret for admin password | `""` |
| `auth.existingAdminUserSecret` | Existing secret for admin user | `""` |
| `hostname.strict` | Strict hostname checking | `false` |
| `hostname.url` | Frontend URL (sets KC_HOSTNAME) | `""` |
| `hostname.adminUrl` | Admin URL | `""` |
| `extraEnv` | Extra environment variables | `[]` |
| `extraEnvFrom` | Extra envFrom sources | `[]` |
| `extraVolumes` | Extra volumes | `[]` |
| `extraVolumeMounts` | Extra volume mounts | `[]` |
| `nodeSelector` | Node selector | `{}` |
| `tolerations` | Tolerations | `[]` |
| `affinity` | Affinity | `{}` |
| `autoscaling.enabled` | Enable HPA | `false` |
| `autoscaling.minReplicas` | HPA min replicas | `1` |
| `autoscaling.maxReplicas` | HPA max replicas | `5` |
| `autoscaling.targetCPUUtilizationPercentage` | CPU target | `80` |
| `autoscaling.targetMemoryUtilizationPercentage` | Memory target | `80` |

## Production Considerations

1. **Database**: Always use a production-grade external database. The built-in H2 (`dev-file`) is for development only.
2. **Replicas**: For high availability, set `replicaCount: 2` or more, enable `KC_CACHE_STACK: kubernetes`, and set `rbac.create: true`.
3. **TLS**: Always enable TLS for production. Use cert-manager with `ingress.tls` or terminate TLS at the ingress.
4. **Resources**: Adjust CPU/memory limits based on your expected workload. Keycloak benefits from sufficient heap memory.
5. **Backup**: Regularly back up your database. Keycloak stores all configuration and user data in the database.
6. **Proxy headers**: The chart sets `KC_PROXY_HEADERS=xforwarded` by default for ingress-based deployments. Adjust as needed for your proxy setup.

## Uninstall

```bash
helm uninstall my-keycloak
```
