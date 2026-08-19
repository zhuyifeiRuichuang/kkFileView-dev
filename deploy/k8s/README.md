# kkFileView Kubernetes 部署

本目录提供 kkFileView 在 K8s 环境的一键部署清单，**无状态**（不挂载持久卷，缓存写入容器可写层）。

## 资源说明

| 文件 | 作用 |
|------|------|
| `namespace.yaml` | 创建独立命名空间 `kkfileview` |
| `configmap.yaml` | 配置 `application.properties`（已内嵌默认配置），以文件形式挂载实现灵活配置 |
| `deployment.yaml` | 无状态 Deployment，挂载 ConfigMap 覆盖配置；挂载点固定为 `/opt/kkfileview/config/application.properties`（与镜像版本无关） |
| `service-clusterip.yaml` | 集群内部访问（ClusterIP） |
| `service-nodeport.yaml` | 通过节点端口暴露（NodePort，固定 30812） |
| `kustomization.yaml` | 聚合上述资源，可用 `kubectl apply -k` 一键部署 |

## 调整配置（灵活配置）

配置全部在 `configmap.yaml` 的 `data.application.properties` 中。修改后重新应用即可生效：

```bash
# 编辑 configmap.yaml 中的 application.properties 内容，然后：
kubectl apply -f configmap.yaml
kubectl rollout restart deployment/kkfileview -n kkfileview
```

> 提示：配置文件中大量项支持 `${KK_*}` 环境变量覆盖，也可在 `deployment.yaml` 的 `env` 中注入，无需改动 ConfigMap。

## 部署

方式一（kustomize，推荐）：

```bash
kubectl apply -k deploy/k8s/
```

方式二（逐个文件）：

```bash
kubectl apply -f namespace.yaml
kubectl apply -f configmap.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service-clusterip.yaml
kubectl apply -f service-nodeport.yaml
```

## 镜像版本

`deployment.yaml` 默认使用 `ghcr.io/zhuyifeiruichuang/kkfileview:latest`。锁定版本请在 `kustomization.yaml` 的 `images` 段覆写 `newTag`，或直接改 `deployment.yaml` 的 `image`。镜像 tag 与代码仓库版本号一致（如 `5.0.2`）。

## 访问

- 集群内部：`http://kkfileview-clusterip.kkfileview.svc:8012`
- 节点端口：`http://<任意节点IP>:30812`
- 如需 Ingress / 域名，请按需自行补充 Ingress 资源。

## 健康检查

Deployment 配置了三类探针（`httpGet` 由 kubelet 在节点侧发起，**不依赖容器内是否安装 curl 等工具**）：

- `startupProbe`：最长 180s 宽限期（periodSeconds 10 × failureThreshold 18），避免 JVM + LibreOffice 冷启动期间被 liveness 误杀。
- `readinessProbe`：就绪后（HTTP 200）才纳入 Service 流量。
- `livenessProbe`：持续异常（HTTP 失败）时重启容器。

```bash
kubectl get pods -n kkfileview
kubectl describe pod -n kkfileview <pod>   # 查看探针事件与失败原因
```
