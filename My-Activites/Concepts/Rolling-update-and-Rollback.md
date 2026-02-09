Below is a **complete step-by-step solution with exact kubectl commands** to perform **deployment, rolling update, and rollback** as requested.

---

## 1) Create Namespace

```bash
kubectl create namespace datacenter
```

Verify:

```bash
kubectl get ns
```

---

## 2) Create Deployment (httpd-deploy)

Create a YAML file:

```bash
vi httpd-deploy.yaml
```

Paste below content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deploy
  namespace: datacenter
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:2.4.25
        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f httpd-deploy.yaml
```

Verify:

```bash
kubectl get deploy -n datacenter
kubectl get pods -n datacenter
```

---

## 3) Create NodePort Service

Create service YAML:

```bash
vi httpd-service.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-service
  namespace: datacenter
spec:
  type: NodePort
  selector:
    app: httpd
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30008
```

Apply:

```bash
kubectl apply -f httpd-service.yaml
```

Verify:

```bash
kubectl get svc -n datacenter
```

---

## 4) Perform Rolling Update (Upgrade Image)

Update deployment image:

```bash
kubectl set image deployment/httpd-deploy httpd=httpd:2.4.43 -n datacenter
```

Watch rollout:

```bash
kubectl rollout status deployment/httpd-deploy -n datacenter
```

Verify updated image:

```bash
kubectl describe deploy httpd-deploy -n datacenter | grep Image
```

---

## 5) Rollback to Previous Version

Undo last deployment:

```bash
kubectl rollout undo deployment/httpd-deploy -n datacenter
```

Verify rollback:

```bash
kubectl rollout status deployment/httpd-deploy -n datacenter
```

Check image reverted:

```bash
kubectl describe deploy httpd-deploy -n datacenter | grep Image
```

Expected output:

```
Image: httpd:2.4.25
```

---

## 6) Verify End-to-End

```bash
kubectl get all -n datacenter
```

Service access:

```
http://<Node-IP>:30008
```

---



***************************************************************************************

**maxUnavailable** and **maxSurge** control *how pods are replaced during a rolling update* in Kubernetes.

### Simple Explanation 👇

When you update a deployment, Kubernetes replaces old pods with new ones **gradually**, not all at once. These two settings control **speed vs availability**.

---

## 🔹 maxUnavailable

👉 **Maximum number of pods that can be unavailable (down) during the update.**

**In simple terms:**
How many pods are allowed to be **not running** at any moment.

### Example:

```yaml
replicas: 4
maxUnavailable: 1
```

✔ Kubernetes ensures **at least 3 pods are always running** during update.

If:

```yaml
maxUnavailable: 2
```

✔ Up to **2 pods can be down** at once.

---

## 🔹 maxSurge

👉 **Maximum number of extra pods that can be created above desired replicas during the update.**

**In simple terms:**
How many **extra temporary pods** Kubernetes can create during update.

### Example:

```yaml
replicas: 4
maxSurge: 1
```

✔ Kubernetes can temporarily run **5 pods** during update.

If:

```yaml
maxSurge: 2
```

✔ Kubernetes can temporarily run **6 pods**.

---

## 🔹 Your Case Example

```yaml
replicas: 2
maxSurge: 1
maxUnavailable: 2
```

### During rollout:

* Normal pods = **2**
* Max extra pods = **+1 → total 3**
* Max pods that can be down = **2**

So Kubernetes can:

* Bring up **1 new pod**
* Take down **up to 2 old pods**

This gives **faster rollout but lower availability risk**.

---

## 🔹 Real-World Analogy

Think of **office shift replacement**:

* **maxSurge** → Extra temporary employees allowed
* **maxUnavailable** → How many employees can be absent at once

---

## 🔹 Best Practice Values

| Environment | maxUnavailable | maxSurge |
| ----------- | -------------- | -------- |
| Production  | 0 or 1         | 1        |
| Dev / Test  | 1–2            | 1–2      |

---

If you want, I can also explain **RollingUpdate flow step-by-step with diagram style explanation**.


If you'd like, I can also provide **single-shot commands without YAML** or **a one-file manifest version** for faster exam-style execution.

