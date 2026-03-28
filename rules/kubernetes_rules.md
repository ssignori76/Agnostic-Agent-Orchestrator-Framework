# Kubernetes Rules

> **Purpose:** Kubernetes deployment patterns, manifest conventions, and Minikube-first
> testing strategy for all AAOF projects targeting Kubernetes.
>
> **Scope:** All projects with `K8S` in `VAR_DEPLOY_TARGET`.

---

## 1. Minikube-First Approach

**All Kubernetes testing must be done on Minikube** before targeting any cloud provider.
This ensures:

- No cloud costs during development and validation
- Consistent environment for all contributors
- Easy reset (`minikube delete && minikube start`)

### Minikube Setup Assumptions

```bash
minikube start --driver=docker --memory=4096 --cpus=2
minikube addons enable ingress
eval $(minikube docker-env)   # Use Minikube's Docker daemon for local images
```

### Minikube Prerequisite Check

Minikube verification is performed at STEP 0 only when `deploy_targets` includes Kubernetes.

- If Minikube is **not installed** and the user **declines** installation:
  - `VAR_MINIKUBE_APPROVED` is set to `false` and persists across sessions.
  - ALL Kubernetes operations are disabled until the user authorizes installation.
  - The agent proceeds with Docker-only deployment.
  - At every new session, the agent reminds the user: *"Minikube is not authorized. Kubernetes deployment is disabled."*
- If the user later **approves** installation:
  - The agent guides installation and verification.
  - `VAR_MINIKUBE_INSTALLED` and `VAR_MINIKUBE_APPROVED` are set to `true`.
  - Kubernetes operations are enabled from that point forward.

---

## 2. One Resource Per YAML File

**Never put multiple Kubernetes resources in the same YAML file.**

✅ Correct:
```
k8s/
├── namespace.yaml
├── deployment.yaml
├── service.yaml
├── configmap.yaml
└── secret.yaml
```

❌ Wrong:
```yaml
# A single file with multiple resources separated by ---
apiVersion: v1
kind: Namespace
---
apiVersion: apps/v1
kind: Deployment
```

This makes diffs, review, and targeted `kubectl apply` much cleaner.

---

## 3. Kustomize for Templating

Use **Kustomize** (built into `kubectl`) for environment-specific overlays:

```
k8s/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   └── service.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

Apply with: `kubectl apply -k k8s/overlays/dev`

---

## 4. Namespace Isolation

- Every project must have its own namespace.
- Never deploy to the `default` namespace in a shared cluster.
- Namespace name convention: `<project-name>-<env>` e.g. `my-app-dev`, `my-app-prod`.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-app-dev
  labels:
    app.kubernetes.io/managed-by: aaof
```

---

## 5. Standard Resource Patterns

### Deployment

- Always set `replicas` explicitly (even if 1).
- Always define `resources.requests` and `resources.limits`.
- Always define `livenessProbe` and `readinessProbe`.
- Use `RollingUpdate` strategy with `maxUnavailable: 0` for zero-downtime.
- Use `app.kubernetes.io/*` labels.

### Service

- Use `ClusterIP` for internal services.
- Use `NodePort` only for Minikube development access.
- Use `LoadBalancer` or `Ingress` for external production access.

### ConfigMap

- Store non-sensitive configuration as ConfigMaps.
- Mount as files (not env vars) for complex configs.
- Name convention: `<app-name>-config`.

### Secret

- Never store secrets in git. See `security_rules.md`.
- Use Kubernetes Secrets for credentials passed into pods.
- For production, use an external secret manager (Vault, AWS Secrets Manager).
- Name convention: `<app-name>-secret`.

---

## 6. Health Probes

Every Deployment must define both probes:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

---

## 7. Resource Requests and Limits

Always set both:

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "50m"
  limits:
    memory: "256Mi"
    cpu: "500m"
```

Omitting limits can cause a single pod to consume all cluster resources.

---

## 8. PersistentVolumeClaim for Stateful Workloads

- Databases and stateful services must use PVCs — never `hostPath` volumes.
- Use `StorageClass: standard` for Minikube.
- Name convention: `<service-name>-pvc`.

---

## 9. Docker Compose to K8s Translation

When the user requests both Docker and K8s targets, the agent should:

1. Implement with Docker Compose first (STEP 4).
2. Validate with Docker (STEP 5).
3. Translate to K8s manifests following these rules:
   - Each `docker-compose` service → Deployment + Service YAML
   - Compose `volumes` → PVC
   - Compose `environment` → ConfigMap or Secret
   - Compose `ports` → Service `targetPort`/`port`
4. Apply and validate on Minikube (second STEP 5 pass).

---

## 10. Nothing Hardcoded Inside — Configuration Externalization Principle

> **Core Principle:** Every configuration file, parameter, or value that may vary between
> environments (dev/staging/prod) or between deployments MUST be externalized from the
> container image. Nothing configurable stays inside the container.

### 10.1 Why This Matters

Containers must be **immutable artifacts**. The same image must be deployable to any
environment by changing only external configuration. Hardcoding configuration inside
images creates:
- Rebuild requirements for every environment change
- Security risks (secrets baked into layers)
- Inability to hot-reload configuration
- Violation of the 12-Factor App methodology (Factor III: Config)

### 10.2 Classification Rules

| Category | K8s Resource | Mount Method | Examples |
| :--- | :--- | :--- | :--- |
| **Application config files** | ConfigMap | Volume mount as file | `application.yml` (Spring Boot), `httpd.conf` (Apache), `nginx.conf`, `php.ini`, `settings.json`, `appsettings.json` (.NET) |
| **Static pages / templates** | ConfigMap | Volume mount as file | `index.html` (landing/default pages), email templates, custom error pages |
| **Credentials & sensitive data** | Secret | Volume mount or env var | DB passwords, API keys, TLS certificates, OAuth tokens, connection strings |
| **Simple runtime parameters** | ConfigMap | Environment variable (`envFrom` or `env.valueFrom`) | Port numbers, log levels, feature flags, timeouts |
| **Environment-specific URLs** | ConfigMap | Environment variable | Database URLs, API endpoints, domain names, external service URLs |

### 10.3 Default Rule

> **When in doubt → ConfigMap.** If unsure whether something should be a ConfigMap,
> Secret, or env var, default to ConfigMap mounted as a file. It's the safest and most
> flexible option.

### 10.4 Framework-Specific Examples

#### Spring Boot (Java)
```yaml
# ConfigMap for application.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  application.yml: |
    server:
      port: 8080
    spring:
      datasource:
        url: ${DB_URL}
---
# Mount in Deployment
volumes:
  - name: config-volume
    configMap:
      name: app-config
volumeMounts:
  - name: config-volume
    mountPath: /app/config/application.yml
    subPath: application.yml
```

#### Apache HTTP Server
```yaml
# ConfigMap for httpd.conf AND index.html
apiVersion: v1
kind: ConfigMap
metadata:
  name: apache-config
data:
  httpd.conf: |
    ServerRoot "/usr/local/apache2"
    Listen 80
    # ... full config
  index.html: |
    <h1>Welcome</h1>
```

#### Nginx
```yaml
# ConfigMap for nginx.conf
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    worker_processes auto;
    # ... full config
  default.conf: |
    server {
      listen 80;
      # ... server block
    }
```

#### Node.js / Express
```yaml
# ConfigMap for environment-based config
apiVersion: v1
kind: ConfigMap
metadata:
  name: node-config
data:
  APP_PORT: "3000"
  LOG_LEVEL: "info"
  NODE_ENV: "production"
```

### 10.5 Agent Workflow Integration

During **STEP 2 (Execution Plan)**, the agent MUST:

1. **Scan** the project source code for configuration files and patterns.
2. **Classify** each item using the table in §10.2.
3. **Present** the externalization map to the user for approval.
4. **Store** the approved map in `VAR_K8S_EXTERNALIZATION_MAP`.
5. **Generate** all K8s manifests during STEP 4 according to the approved map.

The `VAR_K8S_EXTERNALIZATION_MAP` structure:
```json
{
  "configmaps": [
    {
      "name": "app-config",
      "files": ["application.yml"],
      "mount_path": "/app/config/",
      "description": "Spring Boot application configuration"
    }
  ],
  "secrets": [
    {
      "name": "db-credentials",
      "keys": ["DB_PASSWORD", "DB_USERNAME"],
      "type": "env",
      "description": "Database access credentials"
    }
  ],
  "env_vars": [
    {
      "name": "APP_PORT",
      "source": "configmap/app-runtime",
      "description": "Application listen port"
    }
  ],
  "user_approved": true
}
```

### 10.6 Validation at STEP 5

During non-regression check (STEP 5.0), the agent must verify:
- Every file listed in `VAR_K8S_EXTERNALIZATION_MAP` has a corresponding ConfigMap or Secret manifest.
- No configuration file identified in the map is embedded inside a Docker image — specifically, verify that the final runtime stage of multi-stage Dockerfiles does not COPY any externalized config file.
- All Secrets use Kubernetes Secret resources — never plain ConfigMaps for sensitive data.
