---

# Day 1 – Pod Disruption Budgets (PDBs)

## 📌 What I Learned

A Pod Disruption Budget (PDB) ensures that a minimum number of pods are always running during voluntary disruptions (e.g., node drain, upgrades).

It prevents situations where all replicas of an application get evicted at the same time.

PDB does not protect against involuntary disruptions (node crash, kernel panic).



---

## 📖 Key Concepts

minAvailable → Minimum number of pods that must remain available.

maxUnavailable → Maximum number of pods that can be taken down.

Works together with Deployments, ReplicaSets, StatefulSets.



---

## 🛠️ Practical Demo

Step 1 – Create a Deployment with 3 replicas

```
apiVersion: apps/v1
kind: Deployment
metadata:
 name: myapp
spec:
 replicas: 3
 selector:
 matchLabels:
 app: myapp
 template:
 metadata:
 labels:
 app: myapp
 spec:
 containers:
 - name: nginx
 image: nginx

kubectl apply -f deployment.yaml
kubectl get pods -l app=myapp
```

---

## Step 2 – Apply a Pod Disruption Budget

```
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
 name: myapp-pdb
spec:
 minAvailable: 2
 selector:
 matchLabels:
 app: myapp

kubectl apply -f pdb.yaml
kubectl get pdb

Expected output:

NAME MIN AVAILABLE MAX UNAVAILABLE ALLOWED DISRUPTIONS AGE
myapp-pdb 2 N/A 1 10s

```

---

## Step 3 – Test by draining a node

kubectl drain <node-name> --ignore-daemonsets --force

Kubernetes will evict pods but will never go below 2 running pods.



---

## ✅ Key Takeaways

PDB = High Availability during voluntary disruptions.

Always set PDBs for production workloads (especially critical apps).

Use kubectl get pdb to check allowed disruptions.



---
