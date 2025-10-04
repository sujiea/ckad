# Day 2 — CKAD Practice: ConfigMaps, Secrets, Probes

**Goal:** Create a ConfigMap from a file and inject into a Pod; mount a Secret as env vars and as a volume; add liveness & readiness probes.  
**Target time:** 25–35 minutes  
**Namespace:** `day2`  
**Pod name:** `web`

---

## 0) Prep

```bash
alias k=kubectl
k create ns day2
# (optional) use day2 by default
k config set-context --current --namespace=day2
```

---

## 1) Create a ConfigMap **from a file** and inject into a Pod

Use an **env file** so keys become env vars (works well with `envFrom`).

```bash
cat > app.env <<'EOF'
APP_COLOR=blue
APP_MODE=prod
FEATURE_X=true
EOF

k create configmap app-config --from-env-file=app.env -n day2
k get cm app-config -n day2 -o yaml
```

---

## 2) Create a Secret and use it as **env vars + mounted volume**

```bash
k create secret generic app-secret   --from-literal=DB_USER=dkuser   --from-literal=DB_PASS='S3cr3t!'   -n day2

k get secret app-secret -n day2
```

---

## 3) Create the Pod with envFrom + secret env + secret/config volumes + probes

Save as **`web.yaml`**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
  namespace: day2
  labels:
    app: web
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    ports:
    - containerPort: 80
    # Inject ALL keys from the ConfigMap as env vars
    envFrom:
    - configMapRef:
        name: app-config
    # Inject specific Secret keys as env vars
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: DB_USER
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: DB_PASS
    # Mount both Secret and ConfigMap as read-only files
    volumeMounts:
    - name: cfg
      mountPath: /etc/config
      readOnly: true
    - name: creds
      mountPath: /etc/secret
      readOnly: true
    # Probes
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 2
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 2
      periodSeconds: 3
      failureThreshold: 2
  volumes:
  - name: cfg
    configMap:
      name: app-config
  - name: creds
    secret:
      secretName: app-secret
      defaultMode: 0400
```

Apply and wait:

```bash
k apply -f web.yaml
k wait --for=condition=Ready pod/web -n day2 --timeout=60s
k get pod web -n day2 -o wide
```

Debugging if issues:
```bash
kubectl get pod web -n day2 && echo "---" && kubectl describe pod web -n day2
kubectl delete pod web -n day2 && kubectl apply -f web.yaml
```
---

## 4) Verify (quick checks)

**Env injected:**

```bash
k exec web -n day2 -- sh -c 'echo $APP_COLOR $APP_MODE $FEATURE_X'
k exec web -n day2 -- sh -c 'echo $DB_USER && echo $DB_PASS | sed "s/./*/g"'
```

**Files mounted:**

```bash
k exec web -n day2 -- sh -c 'ls -l /etc/config /etc/secret && cat /etc/config/APP_COLOR'
```

**Probes working:**

```bash
k describe pod web -n day2 | sed -n '/Liveness/,+4p;/Readiness/,+4p'
# You should see "success" counters increasing and the pod "Ready: True"
```

---

## 5) (Optional) See a failing probe in action (for learning)

Edit readiness to a bad path, watch it go NotReady, then revert:

```bash
k patch pod web -n day2 --type merge -p '{"spec":{"containers":[{"name":"nginx","readinessProbe":{"httpGet":{"path":"/does-not-exist","port":80}}}]}}'
k get pod web -n day2 -w

# Revert back:
k patch pod web -n day2 --type merge -p '{"spec":{"containers":[{"name":"nginx","readinessProbe":{"httpGet":{"path":"/","port":80}}}]}}'
```

---

## 6) Clean up (optional)

```bash
k delete ns day2
```

---

## ⏱ Timed mini-checklist (repeat until under 10 minutes)

1. Create `app.env`, build `ConfigMap` from file ✔️  
2. Create `Secret` with two literals ✔️  
3. Apply Pod YAML with: envFrom (ConfigMap), env (Secret), volumes, probes ✔️  
4. Verify env + mounts + probes ✔️
