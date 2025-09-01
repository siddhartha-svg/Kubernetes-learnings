

# Day 5 – Autoscaling (HPA & VPA)

## 📌 What I Learned

Kubernetes supports automatic scaling to handle variable workloads.

Two main types:

Horizontal Pod Autoscaler (HPA) → scales number of pods.

Vertical Pod Autoscaler (VPA) → adjusts pod resource requests/limits.


HPA works on CPU/Memory metrics (or custom metrics with Prometheus/metrics-server).

VPA is less common in production but very powerful for optimizing resource usage.




## 📖 Key Concepts

HPA → Uses metrics to decide when to add/remove pods.

Needs metrics-server running in the cluster.

VPA → Recommends or automatically adjusts pod CPU/memory.



---
```
🛠️ Practical Demo – HPA

Step 1 – Create a Deployment

apiVersion: apps/v1
kind: Deployment
metadata:
 name: myapp
spec:
 replicas: 1
 selector:
 matchLabels:
 app: myapp
 template:
 metadata:
 labels:
 app: myapp
 spec:
 containers:
 - name: myapp-container
 image: k8s.gcr.io/hpa-example
 ports:
 - containerPort: 80
 resources:
 requests:
 cpu: 100m
```
```
kubectl apply -f deployment.yaml
kubectl get pods -l app=myapp
```

---

Step 2 – Expose Deployment

kubectl expose deployment myapp --port=80 --target-port=80


---

## Step 3 – Create an HPA

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
 name: myapp-hpa
spec:
 scaleTargetRef:
 apiVersion: apps/v1
 kind: Deployment
 name: myapp
 minReplicas: 1
 maxReplicas: 5
 metrics:
 - type: Resource
 resource:
 name: cpu
 target:
 type: Utilization
 averageUtilization: 50
```
```
kubectl apply -f hpa.yaml
kubectl get hpa
```
```
Expected output:

NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE
myapp-hpa Deployment/myapp 10%/50% 1 5 1 10s
```

---

## Step 4 – Generate Load to Trigger Scaling

```
kubectl run -i --tty load-generator --image=busybox --restart=Never -- /bin/sh

Inside pod, run:

while true; do wget -q -O- http://myapp.default.svc.cluster.local; done

Keep this running to generate CPU load.
```

### In another terminal:

```
kubectl get hpa -w
kubectl get pods -l app=myapp -w
```

You’ll see pods scale up when CPU > 50% and down when CPU drops.



---

## 🛠️ Bonus – VPA (Concept Only for Now)

VPA adjusts requests/limits automatically.

Three modes:

Off (only gives recommendations).

Auto (applies recommendations automatically).

Initial (sets resources only at pod start).



## 👉 Requires installing VPA components (not default in clusters).


---

## ✅ Key Takeaways

HPA = scales pods horizontally (good for web apps).

VPA = scales resources vertically (good for batch jobs, ML workloads).

Always define requests/limits → required for scaling to work properly.

Combine HPA + VPA carefully (they can conflict if not tuned).

---
