

# Day 2 – Resource Requests & Limits

## 📌 What I Learned

Resource Requests: Minimum guaranteed CPU & memory for a pod.

Resource Limits: Maximum CPU & memory a pod can consume.

Without requests/limits, a single pod may consume excessive resources and cause instability in the cluster.

If a container exceeds memory limit, it gets OOMKilled.

If it exceeds CPU limit, it gets throttled (not killed).





## 📖 Key Concepts

CPU unit:

1000m = 1 CPU core.

500m = half CPU core.


Memory unit:

Mi = Mebibyte (1024^2 bytes).

Gi = Gibibyte (1024^3 bytes).




---

## 🛠️ Practical Demo

Step 1 – Pod with resource requests & limits
```
apiVersion: v1
kind: Pod
metadata:
 name: resourced-pod
spec:
 containers:
 - name: stress-container
 image: polinux/stress
 args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]
 resources:
 requests:
 memory: "256Mi"
 cpu: "200m"
 limits:
 memory: "512Mi"
 cpu: "500m"
```
```
kubectl apply -f resources.yaml
kubectl get pod resourced-pod
```

---

Step 2 – Check pod resource usage

kubectl describe pod resourced-pod
kubectl top pod resourced-pod

kubectl describe shows requests/limits.

kubectl top shows live CPU/Memory usage.



---

## Step 3 – Observe behavior

If container tries to use > 512Mi memory → OOMKilled.

If container tries to use > 500m CPU → CPU throttled, not killed.



---

## ✅ Key Takeaways

Always set requests → ensures fair scheduling.

Always set limits → prevents resource hogging.

Critical for cluster stability + autoscaling.

Best practice: define requests/limits in all production workloads.



---
