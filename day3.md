# Day 3 — CKAD Practice: Multi-container Pods

**Goal:** Create Pods with initContainers that wait for services; implement logging sidecar pattern.  
**Target time:** 20–30 minutes total (10 minutes per YAML)  
**Namespace:** `day3`  
**Focus:** Init containers, sidecars, multi-container communication

---

## 0) Prep

```bash
alias k=kubectl
k create ns day3
# (optional) use day3 by default
k config set-context --current --namespace=day3
```

---

## 1) Pod with initContainer that waits for a service

**Scenario:** Main app container should only start after a database service is available.

First, create a service to wait for:

```bash
# Create a simple service (we'll create the pod later)
k create deployment db --image=postgres:13 --port=5432 -n day3
k expose deployment db --port=5432 --target-port=5432 -n day3
k get svc db -n day3
```

Create **`init-container-pod.yaml`**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
  namespace: day3
  labels:
    app: web-app
spec:
  initContainers:
  - name: wait-for-db
    image: busybox:1.35
    command: 
    - 'sh'
    - '-c'
    - |
      echo "Waiting for database service to be ready..."
      until nslookup db.day3.svc.cluster.local; do
        echo "Database not ready yet, sleeping 5 seconds..."
        sleep 5
      done
      echo "Database service is ready!"
  containers:
  - name: web
    image: nginx:1.25
    ports:
    - containerPort: 80
    env:
    - name: DB_HOST
      value: "db.day3.svc.cluster.local"
    - name: DB_PORT
      value: "5432"
```

Apply and observe:

```bash
k apply -f init-container-pod.yaml
k get pods -n day3 -w
# You should see: Init:0/1 -> PodInitializing -> Running
```

Verify the init process worked:

```bash
k describe pod web-app -n day3 | grep -A 10 "Init Containers"
k logs web-app -c wait-for-db -n day3
```

---

## 2) Logging Sidecar Pattern

**Scenario:** Main app writes logs to a file, sidecar container ships logs to stdout for kubectl logs.

Create **`sidecar-logging-pod.yaml`**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
  namespace: day3
  labels:
    app: logging-demo
spec:
  containers:
  # Main application container
  - name: app
    image: busybox:1.35
    command:
    - 'sh'
    - '-c'
    - |
      while true; do
        echo "$(date): App processing request #$RANDOM" >> /shared/app.log
        echo "$(date): Database query executed" >> /shared/app.log
        sleep 10
      done
    volumeMounts:
    - name: shared-logs
      mountPath: /shared
  
  # Sidecar container for log shipping
  - name: log-shipper
    image: busybox:1.35
    command:
    - 'sh'
    - '-c'
    - |
      echo "Log shipper started, monitoring /shared/app.log"
      tail -f /shared/app.log
    volumeMounts:
    - name: shared-logs
      mountPath: /shared
  
  volumes:
  - name: shared-logs
    emptyDir: {}
```

Apply and test:

```bash
k apply -f sidecar-logging-pod.yaml
k get pods -n day3
k wait --for=condition=Ready pod/app-with-sidecar -n day3 --timeout=60s
```

Verify sidecar logging:

```bash
# Check logs from main app (should be empty in stdout)
k logs app-with-sidecar -c app -n day3

# Check logs from sidecar (should show app logs)
k logs app-with-sidecar -c log-shipper -n day3 -f

# Verify shared volume
k exec app-with-sidecar -c app -n day3 -- ls -la /shared/
k exec app-with-sidecar -c app -n day3 -- tail -5 /shared/app.log
```

---

## 3) Advanced: Init + Sidecar Combined

**Scenario:** Pod with init container + main app + logging sidecar.

Create **`init-sidecar-combo.yaml`**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: full-example
  namespace: day3
  labels:
    app: full-stack
spec:
  initContainers:
  - name: setup
    image: busybox:1.35
    command:
    - 'sh'
    - '-c'
    - |
      echo "Setting up application..."
      echo "config_loaded=true" > /shared/config.txt
      echo "app_version=1.0" >> /shared/config.txt
      echo "Setup complete!"
    volumeMounts:
    - name: shared-data
      mountPath: /shared
  
  containers:
  # Main application
  - name: app
    image: nginx:1.25
    ports:
    - containerPort: 80
    command:
    - 'sh'
    - '-c'
    - |
      echo "App started with config:" && cat /shared/config.txt
      # Start nginx and log to file
      nginx -g 'daemon off;' &
      while true; do
        echo "$(date): Nginx serving requests" >> /shared/access.log
        sleep 15
      done
    volumeMounts:
    - name: shared-data
      mountPath: /shared
  
  # Logging sidecar
  - name: logger
    image: busybox:1.35
    command: ['sh', '-c', 'tail -f /shared/access.log']
    volumeMounts:
    - name: shared-data
      mountPath: /shared
  
  volumes:
  - name: shared-data
    emptyDir: {}
```

Apply and verify:

```bash
k apply -f init-sidecar-combo.yaml
k get pods -n day3 -o wide

# Check init container logs
k logs full-example -c setup -n day3

# Check main app
k logs full-example -c app -n day3

# Check sidecar logs
k logs full-example -c logger -n day3 -f
```

---

## 4) Debugging Multi-container Pods

Essential commands for troubleshooting:

```bash
# Check all containers in a pod
k get pod <pod-name> -n day3 -o jsonpath='{.spec.containers[*].name}'

# Get status of all containers
k get pod <pod-name> -n day3 -o jsonpath='{.status.containerStatuses[*].name}'

# Describe for detailed events
k describe pod <pod-name> -n day3

# Logs from specific container
k logs <pod-name> -c <container-name> -n day3

# Exec into specific container
k exec -it <pod-name> -c <container-name> -n day3 -- /bin/sh

# Check shared volumes
k exec <pod-name> -c <container-name> -n day3 -- ls -la /shared/
```

---

## 5) Quick Verification Checklist

**Init Container Pod:**
```bash
k get pod web-app -n day3
k logs web-app -c wait-for-db -n day3 | grep "Database service is ready"
```

**Sidecar Logging:**
```bash
k logs app-with-sidecar -c log-shipper -n day3 --tail=5
k exec app-with-sidecar -c app -n day3 -- wc -l /shared/app.log
```

**Combined Example:**
```bash
k get pod full-example -n day3 -o jsonpath='{.status.phase}'
k logs full-example -c logger -n day3 --tail=3
```

---

## 6) Clean up

```bash
k delete ns day3
```

---

## ⏱ Timed Mini-Checklist (Target: 20 minutes total)

**Part 1 (10 min):** Init Container
1. Create service for db ✔️  
2. Write Pod YAML with initContainer using nslookup ✔️  
3. Apply and verify init process ✔️  
4. Check logs to confirm init completed ✔️

**Part 2 (10 min):** Sidecar Pattern  
1. Write Pod YAML with main app + sidecar ✔️  
2. Use shared emptyDir volume ✔️  
3. Main writes to file, sidecar tails to stdout ✔️  
4. Verify logs appear in sidecar container ✔️

---

## 💡 CKAD Exam Tips

- **Init containers** run to completion before main containers start
- **Sidecar containers** run alongside main containers for the pod's lifetime
- Use **emptyDir** volumes for sharing data between containers in the same pod
- **Always specify container name** with `-c` flag when using `kubectl logs` or `kubectl exec`
- **nslookup** is common for service discovery in init containers
- **tail -f** is the standard pattern for log shipping sidecars

---

## 🔍 Common Patterns to Remember

1. **Service Readiness Check**: `nslookup service-name.namespace.svc.cluster.local`
2. **Log Shipping**: Main app writes to file, sidecar does `tail -f filename`  
3. **Shared Storage**: `emptyDir: {}` volume mounted in multiple containers
4. **Container Communication**: Through shared filesystem or localhost networking
