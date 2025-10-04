Week 1 – Build Core Knowledge + Daily Labs


Day 1 – Core Concepts

Course: Pods, Deployments, Namespaces

Practice:

Create a pod with kubectl run and YAML.

```bash
kind create cluster --config kind-ckad.yaml
kubectl get cluster-info
kubectl get nodes
```

Scale a Deployment (imperative + YAML).
```bash
kubectl create namespace demo
kubectl create deployment web --image=ngnix -n demo
kubectl get deploy -n demo
kubectl scale deploy/web --replicas=3 -n demo
kubectl rollout status deploy/web -n demo
kubectl get pods -n demo -o wide
```

debug the image pull stuck issue

```bash
kubectl describe deployment web -n demo
```
or else use deployment-web.yaml

```bash
kubectl apply -f deployment-web.yaml
kubectl rollout status deploy/web
kubectl get deployment,pods -n demo
kubectl delete namespace demo
```

Switch between namespaces.

```bash
kubectl config current-context

```

Time yourself: 5–7 mins each task.

Day 2 – Configurations

Course: ConfigMaps, Secrets, Env Vars

Practice:

Create ConfigMap from a file and inject into Pod.

Mount Secret as env var + volume.

Add liveness & readiness probes.

Mini-lab: 3 tasks in 20 minutes.

Day 3 – Multi-container Pods

Course: Init containers, sidecars

Practice:

Pod with initContainer that waits for a service.

Logging sidecar pattern.

Timebox: 10 minutes per YAML.

Day 4 – Observability

Course: Logging & Monitoring

Practice:

Use kubectl logs, kubectl exec.

Debug a CrashLoopBackOff pod.

Use kubectl describe to find wrong image tag.

Lab: Solve 3 debugging tasks in 25 minutes.

Day 5 – Pod Design I

Course: Jobs & CronJobs

Practice:

Create a Job that prints “Hello CKAD”.

Create a CronJob every minute.

Timebox: 10 minutes.

Day 6 (Sat) – Pod Design II

Course: DaemonSets, StatefulSets

Practice:

DaemonSet for logging agent.

StatefulSet with headless Service.

Mini exam: 5 tasks, 40 minutes.

Day 7 (Sun) – Services & Networking

Course: Services (ClusterIP, NodePort, LoadBalancer), Ingress

Practice:

Expose a Deployment via ClusterIP.

Ingress routing to 2 services.

Write a NetworkPolicy restricting access.

Lab: 4 tasks, 50 minutes.

Week 2 – Exam Simulation Mode
Day 8 – State Persistence

Course: Volumes, PVCs, PVs, StorageClass

Practice:

Pod with PVC mount.

Expand PVC storage.

Timebox: 15 minutes.

Day 9 – Review + Speed Practice

Revisit weak topics from Week 1.

Do short sprints:

5 YAML tasks in 30 minutes.

Only use kubectl explain + docs (no Google).

Day 10 – Mock Exam 1

Resource: killer.sh or KodeKloud CKAD labs.

19 questions, 2 hours.

Review mistakes immediately.

Day 11 – Focused Weakness Review

Retry failed mock exam questions.

Practice those domains until you can finish in <7 minutes each.

Day 12 – Mock Exam 2

Full timed lab, 19 questions, 2 hours.

Aim for 70–75% score (passing = 66%).

Day 13 – Final Review

Drill high-frequency tasks:

Probes, ConfigMaps/Secrets, Deployments, Services, Ingress.

Do 10 tasks in 1 hour.

Quick check: kubectl cheat sheet.

Day 14 – Light Prep + Exam Day

Skim official docs:

CKAD curriculum

Kubernetes docs bookmarks

Do 3–4 warm-up tasks.

Go into exam rested.