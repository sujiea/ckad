# 🧩 CKAD 五天突击打卡表
_目标：5 天快速掌握核心命令 + 模拟实战，免费 voucher 用好、稳过一张证。_

---

## 🌱 Day 1 – Pod & Namespace 基础
**🎯 目标：** 熟悉 kubectl 基本命令，理解 Pod 与命名空间。

| 内容 | 命令 / 练习 | 状态 |
|------|--------------|------|
| 创建 Pod | `kubectl run nginx --image=nginx` | ☐ |
| 查看日志与进入容器 | `kubectl logs nginx` / `kubectl exec -it nginx -- bash` | ☐ |
| 查看对象详情 | `kubectl get pod`, `kubectl describe pod nginx` | ☐ |
| 删除对象 | `kubectl delete pod nginx` | ☐ |
| 创建与切换命名空间 | `kubectl create ns dev` / `kubectl config set-context --current --namespace=dev` | ☐ |
| 💪 练习：创建 nginx Pod 并切换命名空间 | [Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/) | ☐ |
| 💻 实战环境 | [Killercoda Pod lab](https://killercoda.com/killer-shell-cka) | ☐ |

---

## ⚙️ Day 2 – Deployment 与 Service
**🎯 目标：** 掌握部署扩容、滚动更新与服务暴露。

| 内容 | 命令 / 练习 | 状态 |
|------|--------------|------|
| 创建 Deployment | `kubectl create deploy web --image=nginx` | ☐ |
| 扩容缩容 | `kubectl scale deploy web --replicas=3` | ☐ |
| 查看 rollout 状态 | `kubectl rollout status deploy web` | ☐ |
| 暴露服务 | `kubectl expose deploy web --port=80 --type=ClusterIP` | ☐ |
| 回滚 Deployment | `kubectl rollout undo deploy web` | ☐ |
| 💪 练习：创建、更新、回滚 Deployment | [killer.sh CKAD sample](https://killer.sh/ckad) | ☐ |
| 📘 文档练习 | `kubectl explain deploy.spec` | ☐ |

---

## 💾 Day 3 – ConfigMap / Secret / 环境变量
**🎯 目标：** 学会注入配置与机密到容器环境。

| 内容 | 命令 / 练习 | 状态 |
|------|--------------|------|
| 创建 ConfigMap | `kubectl create configmap app-config --from-literal=MODE=prod` | ☐ |
| 创建 Secret | `kubectl create secret generic db-secret --from-literal=PASSWORD=1234` | ☐ |
| 验证环境变量 | `kubectl exec -it pod-name -- env | grep MODE` | ☐ |
| 💪 练习：在 Pod YAML 注入 ConfigMap + Secret | [ConfigMap Docs](https://kubernetes.io/docs/concepts/configuration/configmap/) | ☐ |
| 📘 Secret 官方文档 | [Secret Docs](https://kubernetes.io/docs/concepts/configuration/secret/) | ☐ |

---

## 🩺 Day 4 – Probe / Resource / 多容器 Pod
**🎯 目标：** 写健康检查、资源限制、多容器 YAML。

| 内容 | 命令 / 练习 | 状态 |
|------|--------------|------|
| Liveness probe | `livenessProbe: httpGet: path: / port: 8080` | ☐ |
| Readiness probe | 同上，定义 readinessProbe | ☐ |
| 资源限制 | `resources: requests: cpu: "100m", limits: cpu: "200m"` | ☐ |
| 多容器 Pod | `containers: - name: app - name: sidecar` | ☐ |
| 💪 练习：编写带探针与 limits 的 Pod YAML | [Probe Task](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) | ☐ |

---

## 🚀 Day 5 – 模拟考试 + Docs 查找练习
**🎯 目标：** 熟悉考试界面、文档导航、节奏控制。

| 内容 | 命令 / 练习 | 状态 |
|------|--------------|------|
| 熟悉 `kubectl explain` | `kubectl explain pod.spec.containers` | ☐ |
| 使用 Docs 查 YAML 示例 | [Kubernetes Docs](https://kubernetes.io/docs/) | ☐ |
| 快速 YAML 创建模板 | `kubectl create deploy myapp --image=nginx -o yaml --dry-run=client > myapp.yaml` | ☐ |
| 开启命令自动补全 | `source <(kubectl completion bash)` | ☐ |
| 💪 模拟考试 1 | [KillerCoda CKAD](https://killercoda.com/killer-shell-cka) | ☐ |
| 💪 模拟考试 2 | [Killer.sh CKAD Free Sample](https://killer.sh/ckad) | ☐ |

---

## 💡 考试技巧备忘
- ✅ 考试允许打开 [Kubernetes Docs](https://kubernetes.io/docs/)
- ✅ 多用 `kubectl explain`（比查 docs 快）
- ✅ 多用 `--dry-run=client -o yaml` 生成模板
- ✅ 先写简单题，难题标记再回头
- ✅ 提前 alias：  
  ```bash
  alias k=kubectl
  complete -F __start_kubectl k
