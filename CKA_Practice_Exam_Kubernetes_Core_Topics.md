# CKA Practice Test – Kubernetes Core Topics

**Difficulty:** Beginner → Intermediate  
**Time:** 90 minutes  
**Total:** 13 tasks + 1 bonus challenge  
**Suggested passing target:** 70%

## Rules

- Use only `kubectl` and Kubernetes manifests.
- Do not look at solutions while attempting.
- Always specify the namespace when practicing.
- After each task, verify your work with `kubectl get`, `describe`, `exec`, etc.
- Clean up resources after the exam.

---

# Task 1 — Resource Requests & Limits

Create namespace:

```bash
kubectl create namespace cka-exam
```

Create a Pod called `resource-pod` in namespace `cka-exam`.

Requirements:

- Image: `nginx:1.25`
- CPU request: `100m`
- Memory request: `128Mi`
- CPU limit: `500m`
- Memory limit: `256Mi`

### Verify

```bash
kubectl get pod resource-pod -n cka-exam
kubectl describe pod resource-pod -n cka-exam
```

**CKA skills:** `resources.requests`, `resources.limits`

---

# Task 2 — Pod with Sidecar Container

Create Pod `web-sidecar` in namespace `cka-exam`.

### Container 1

- Name: `web`
- Image: `nginx:1.25`
- Port: `80`

### Container 2 — Sidecar

- Name: `sidecar`
- Image: `busybox:1.36`
- Command should continuously print:

```text
Sidecar running
```

Every 5 seconds.

### Verify

```bash
kubectl logs web-sidecar -c sidecar -n cka-exam
```

Expected:

```text
Sidecar running
Sidecar running
...
```

---

# Task 3 — Multiple Containers

Create Pod:

```text
multi-container-pod
```

Namespace:

```text
cka-exam
```

It must contain **3 containers**:

| Container | Image |
|---|---|
| app | nginx |
| logger | busybox |
| helper | busybox |

The `logger` and `helper` containers should remain running.

### Verify

```bash
kubectl get pod multi-container-pod -n cka-exam
```

Expected:

```text
3/3 Running
```

Then check individual containers:

```bash
kubectl logs multi-container-pod -c logger -n cka-exam
kubectl logs multi-container-pod -c helper -n cka-exam
```

---

# Task 4 — Shared Volume Between Multiple Containers

Create Pod:

```text
shared-volume-pod
```

with two containers.

### Container 1

Name:

```text
writer
```

Image:

```text
busybox
```

It should continuously write:

```text
Hello from writer
```

to:

```text
/shared/data.txt
```

### Container 2

Name:

```text
reader
```

Image:

```text
busybox
```

It should continuously read:

```text
/shared/data.txt
```

### Requirement

Both containers must use the **same `emptyDir` volume**.

Volume name:

```text
shared-data
```

Mount path:

```text
/shared
```

### Verify

```bash
kubectl exec -it shared-volume-pod -c reader -n cka-exam -- cat /shared/data.txt
```

---

# Task 5 — ConfigMap and Secret

Create:

### ConfigMap

Name:

```text
app-config
```

Data:

```text
APP_ENV=production
APP_NAME=payment-app
```

### Secret

Name:

```text
app-secret
```

Data:

```text
DB_USER=admin
DB_PASSWORD=Pass123
```

Create Pod:

```text
config-secret-pod
```

using image:

```text
nginx
```

## Requirement 1 — Environment Variables

Load ConfigMap using:

```yaml
envFrom:
```

Load Secret using:

```yaml
envFrom:
```

Verify:

```bash
kubectl exec -it config-secret-pod -n cka-exam -- env
```

## Requirement 2 — Volume Mount

Mount the ConfigMap into:

```text
/config
```

Mount the Secret into:

```text
/secrets
```

Verify:

```bash
kubectl exec -it config-secret-pod -n cka-exam -- ls /config
kubectl exec -it config-secret-pod -n cka-exam -- ls /secrets
```

---

# Task 6 — Labels and Selectors

Create a Pod:

```text
frontend-pod
```

with labels:

```yaml
app: web
tier: frontend
environment: production
```

Create another Pod:

```text
backend-pod
```

with labels:

```yaml
app: web
tier: backend
environment: production
```

Retrieve only frontend Pods:

```bash
kubectl get pods -n cka-exam -l tier=frontend
```

Retrieve all production Pods:

```bash
kubectl get pods -n cka-exam -l environment=production
```

Retrieve all Pods where:

```text
app=web
```

---

# Task 7 — Services: ClusterIP and NodePort

Create Deployment:

```text
web-deployment
```

Namespace:

```text
cka-exam
```

Requirements:

- Image: `nginx`
- Replicas: `3`
- Label:

```text
app=web
```

## Part A — ClusterIP

Create Service:

```text
web-clusterip
```

Requirements:

```text
type: ClusterIP
port: 80
targetPort: 80
selector:
  app: web
```

Verify:

```bash
kubectl get svc web-clusterip -n cka-exam
kubectl get endpoints web-clusterip -n cka-exam
```

## Part B — NodePort

Create another Service:

```text
web-nodeport
```

Requirements:

```text
type: NodePort
port: 80
targetPort: 80
```

Verify:

```bash
kubectl get svc web-nodeport -n cka-exam
```

Find the NodePort.

---

# Task 8 — ReplicationController

Create a **ReplicationController**:

```text
frontend-rc
```

Requirements:

```text
replicas: 3
image: nginx
label:
  app=frontend
```

Verify:

```bash
kubectl get rc -n cka-exam
kubectl get pods -n cka-exam -l app=frontend
```

### Failure test

Delete one Pod:

```bash
kubectl delete pod <pod-name> -n cka-exam
```

Watch:

```bash
kubectl get pods -n cka-exam -w
```

Confirm that the ReplicationController creates a replacement.

---

# Task 9 — ReplicaSet

Create ReplicaSet:

```text
backend-rs
```

Requirements:

- Replicas: `3`
- Image: `nginx`
- Label:

```text
app=backend
```

### Test scaling

Scale to 5:

```bash
kubectl scale rs backend-rs --replicas=5 -n cka-exam
```

Verify:

```bash
kubectl get rs -n cka-exam
kubectl get pods -n cka-exam -l app=backend
```

Then scale back to 2.

---

# Task 10 — Deployment Strategies

Create Deployment:

```text
payment-app
```

with:

```text
replicas: 4
image: nginx
```

## Part A — RollingUpdate

Use the default:

```text
RollingUpdate
```

Update the image:

```text
nginx:1.25
```

to:

```text
nginx:1.26
```

Watch the rollout:

```bash
kubectl rollout status deployment payment-app -n cka-exam
```

Check:

```bash
kubectl rollout history deployment payment-app -n cka-exam
```

## Part B — Rollback

Rollback to the previous version:

```bash
kubectl rollout undo deployment payment-app -n cka-exam
```

Verify:

```bash
kubectl rollout status deployment payment-app -n cka-exam
```

## Part C — Recreate

Create another Deployment:

```text
payment-recreate
```

with:

```yaml
strategy:
  type: Recreate
```

Use:

```text
replicas: 3
image: nginx
```

Update its image and observe the difference.

---

# Task 11 — Requests and Limits + Deployment

Create Deployment:

```text
api-deployment
```

with:

```text
replicas: 3
image: nginx
```

Each container must have:

```text
CPU request: 200m
Memory request: 256Mi

CPU limit: 500m
Memory limit: 512Mi
```

Verify the generated Pods:

```bash
kubectl describe pod <pod-name> -n cka-exam
```

Find:

```text
Requests:
Limits:
```

---

# Task 12 — DaemonSet

Create a DaemonSet:

```text
node-monitor
```

using:

```text
busybox
```

The container should remain running.

Example behavior:

```text
Node monitoring...
```

every 10 seconds.

### Requirements

- Namespace: `cka-exam`
- One Pod should run on every eligible node.

Verify:

```bash
kubectl get daemonset -n cka-exam
kubectl get pods -n cka-exam -o wide
```

Check:

```bash
kubectl describe daemonset node-monitor -n cka-exam
```

### CKA Bonus

Find how many nodes are available:

```bash
kubectl get nodes
```

Compare:

```text
DESIRED
CURRENT
READY
```

for the DaemonSet.

---

# Task 13 — HPA

Create namespace:

```bash
kubectl create namespace hpa-exam
```

Create Deployment:

```text
hpa-demo
```

Requirements:

```text
image: nginx
replicas: 2
```

Container:

```text
name: nginx
```

CPU request:

```text
100m
```

Create HPA:

```text
hpa-demo
```

Requirements:

| Setting | Value |
|---|---:|
| Min replicas | 2 |
| Max replicas | 5 |
| CPU target | 50% |

### Verify

```bash
kubectl get hpa -n hpa-exam
```

Then:

```bash
kubectl describe hpa hpa-demo -n hpa-exam
```

Check:

```text
TARGETS
MINPODS
MAXPODS
REPLICAS
```

---

# 🔥 Bonus CKA Challenge

Do this without looking at documentation.

Create:

```text
production-app
```

in namespace:

```text
cka-exam
```

## Deployment

- 3 replicas
- nginx
- RollingUpdate
- label:

```text
app=production
tier=frontend
```

## Resources

```text
CPU request: 100m
Memory request: 128Mi
CPU limit: 500m
Memory limit: 256Mi
```

## ConfigMap

Name:

```text
production-config
```

with:

```text
ENV=production
APP_NAME=frontend
```

Load it using:

```text
envFrom
```

## Secret

Name:

```text
production-secret
```

with:

```text
DB_USER=admin
DB_PASSWORD=secret123
```

Load it using:

```text
envFrom
```

## Service

Create:

```text
production-service
```

as:

```text
NodePort
```

## HPA

Configure:

```text
minReplicas: 3
maxReplicas: 6
CPU: 60%
```

### Final verification

You should be able to run:

```bash
kubectl get all -n cka-exam
kubectl get configmap -n cka-exam
kubectl get secret -n cka-exam
kubectl get hpa -n cka-exam
```

and see all required resources.

---

# CKA Practice Exam — Quick Checklist

| # | Topic | Task |
|---|---|---|
| 1 | Requests/Limits | Create Pod with resources |
| 2 | Sidecar | 2-container Pod |
| 3 | Multiple containers | 3-container Pod |
| 4 | Volume | Shared `emptyDir` |
| 5 | ConfigMap/Secret | `envFrom` + volumes |
| 6 | Labels/Selectors | Filter Pods |
| 7 | Services | ClusterIP + NodePort |
| 8 | ReplicationController | Self-healing |
| 9 | ReplicaSet | Scale 3 → 5 → 2 |
| 10 | Deployment | RollingUpdate + Recreate + rollback |
| 11 | Resources | Deployment resources |
| 12 | DaemonSet | One Pod per node |
| 13 | HPA | 2 → 5 CPU scaling |
| Bonus | Combined | Full application |

---

# Recommended Practice Order

For CKA preparation, practice in this sequence:

**Pod → Multi-container → Volume → ConfigMap/Secret → Labels/Selectors → Service → RC → RS → Deployment → DaemonSet → Requests/Limits → HPA**

For every task, follow:

**Question → Create YAML → `kubectl apply` → Verify → Troubleshoot → Delete**

## Exam Strategy

1. Read the complete task first.
2. Identify the resource type.
3. Write the smallest YAML possible.
4. Apply it.
5. Verify immediately.
6. Fix errors before moving to the next question.
7. Use `kubectl explain` when you forget a field.
8. Avoid spending too much time on one task.

> **CKA mindset:** Slow is smooth, smooth is fast.
