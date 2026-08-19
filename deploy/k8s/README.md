# kkFileView Kubernetes Deployment

This directory provides a one-command Kubernetes deployment for kkFileView. It is **stateless** (no persistent volume; the cache is written to the container's writable layer).

## Files

| File | Purpose |
|------|---------|
| `namespace.yaml` | Creates the dedicated namespace `kkfileview`. |
| `configmap.yaml` | Embeds `application.properties` (default config) and mounts it as a file for flexible configuration. |
| `deployment.yaml` | Stateless Deployment. Mounts the ConfigMap to override config at the fixed path `/opt/kkfileview/config/application.properties` (version-independent). Also defines startup/readiness/liveness probes. |
| `service-clusterip.yaml` | In-cluster access (ClusterIP). |
| `service-nodeport.yaml` | Node-port exposure (NodePort, fixed `30812`). |
| `kustomization.yaml` | Aggregates the above; use `kubectl apply -k` for one-shot deployment. |
| `application.properties` | A copy of the config (kept in sync with `configmap.yaml`; the ConfigMap is the source of truth at runtime). |
| `README.md` / `README.cn.md` | This documentation (English / Chinese). |

## Adjust configuration (flexible)

All configuration lives in `configmap.yaml`'s `data.application.properties`. After editing, re-apply and restart:

```bash
# edit configmap.yaml's application.properties content, then:
kubectl apply -f configmap.yaml
kubectl rollout restart deployment/kkfileview -n kkfileview
```

> Many options also support `${KK_*}` environment-variable overrides — inject them via the `env` block in `deployment.yaml` without touching the ConfigMap.

## Deploy

Option 1 (kustomize, recommended):

```bash
kubectl apply -k deploy/k8s/
```

Option 2 (file by file):

```bash
kubectl apply -f namespace.yaml
kubectl apply -f configmap.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service-clusterip.yaml
kubectl apply -f service-nodeport.yaml
```

## Image version

`deployment.yaml` defaults to `ghcr.io/zhuyifeiruichuang/kkfileview:latest`. To pin a version, override `newTag` in `kustomization.yaml`'s `images` section, or edit `deployment.yaml`'s `image` directly. The image tag matches the repository version (e.g. `5.0.2`).

## Access

- In-cluster: `http://kkfileview-clusterip.kkfileview.svc:8012`
- Node port: `http://<any-node-IP>:30812`
- Add your own Ingress/domain as needed.

## Health checks

The Deployment defines three probes (`httpGet` is issued by the kubelet on the node side, so it does **not** depend on curl being installed in the container):

- `startupProbe`: up to 180s grace (periodSeconds 10 × failureThreshold 18) to avoid liveness killing the pod during slow JVM + LibreOffice startup.
- `readinessProbe`: once healthy (HTTP 200), the pod receives Service traffic.
- `livenessProbe`: restarts the container on sustained failure (HTTP error).

```bash
kubectl get pods -n kkfileview
kubectl describe pod -n kkfileview <pod>   # inspect probe events / failure reasons
```
