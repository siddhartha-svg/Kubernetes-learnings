# Day 4 – Init Containers

## 📌 What I Learned

Init Containers are special containers that run before the main app container starts.

They always run to completion before any app container starts.

Useful for:

Waiting for dependencies (DB, service).

Downloading configs/artifacts.

Setting file permissions.

Running one-time setup scripts.




---

## 📖 Key Concepts

A pod can have multiple init containers (executed sequentially).

If an init container fails → pod restarts until it succeeds.

After completion, init containers are removed (not visible in kubectl get pods, only in events/logs).



---

## 🛠️ Practical Demo

Step 1 – Pod with Init Container
```
apiVersion: v1
kind: Pod
metadata:
 name: init-demo
spec:
 initContainers:
 - name: init-wait
 image: busybox
 command: ['sh', '-c', 'echo "Waiting for 10s..." && sleep 10']
 - name: init-setup
 image: busybox
 command: ['sh', '-c', 'echo "Init setup complete"']
 containers:
 - name: main-app
 image: nginx
 ports:
 - containerPort: 80
```

```
kubectl apply -f init-container.yaml
kubectl get pod init-demo
kubectl describe pod init-demo
```

---

## Step 2 – Observe Pod Lifecycle

While init containers run → Pod status = Init:0/2, Init:1/2, etc.

Once all init containers finish → Pod moves to Running.

```
kubectl logs init-demo -c init-wait
kubectl logs init-demo -c init-setup
```

---

## Step 3 – Verify Main App Container

```
kubectl exec -it init-demo -- curl localhost:80
```
If init containers succeed → main Nginx app runs.



---

## ✅ Key Takeaways

Init containers = preparation step before app containers.

They guarantee environment setup before the main app starts.

Common in DB migrations, secrets fetching, dependency checks.

Best practice: keep init containers small and fast.



---
