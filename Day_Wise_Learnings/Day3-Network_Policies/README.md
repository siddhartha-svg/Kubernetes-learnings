---

# Day 3 – Network Policies

## 📌 What I Learned

By default, all pods in Kubernetes can talk to each other → not secure.

NetworkPolicies define who can talk to whom (ingress/egress rules).

Works only if a CNI plugin that supports NetworkPolicy is installed (e.g., Calico, Cilium).

Similar to firewall rules at the pod level.



---

## 📖 Key Concepts

Ingress rules → Control incoming traffic to a pod.

Egress rules → Control outgoing traffic from a pod.

PodSelector → Apply policy to specific pods.

If a pod is selected by a policy with no allow rules, all traffic is denied by default.



---

## 🛠️ Practical Demo

Step 1 – Create 2 Pods: frontend & backend

```
apiVersion: v1
kind: Pod
metadata:
 name: frontend
 labels:
 app: frontend
spec:
 containers:
 - name: frontend
 image: nginx
---
apiVersion: v1
kind: Pod
metadata:
 name: backend
 labels:
 app: backend
spec:
 containers:
 - name: backend
 image: nginx
```
```
kubectl apply -f pods.yaml
kubectl get pods -l app
```

---

## Step 2 – Create a Network Policy (allow only frontend → backend)

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
 name: allow-frontend-to-backend
spec:
 podSelector:
 matchLabels:
 app: backend
 ingress:
 - from:
 - podSelector:
 matchLabels:
 app: frontend

kubectl apply -f network-policy.yaml
kubectl get networkpolicy
```

---

## Step 3 – Test Access
```
1. Exec into frontend pod → should reach backend.
kubectl exec -it frontend -- curl backend:80

2. Exec into another pod (not frontend) → should be blocked.
kubectl run testpod --image=busybox --rm -it -- sh

## Inside pod
wget --spider --timeout=1 backend:80
```

---

## ✅ Key Takeaways

NetworkPolicy = Pod-level firewall inside the cluster.

Default behavior: all traffic allowed until a policy is applied.

Good practice:

Whitelist communication (explicitly allow only needed traffic).

Block everything else.


Essential for Zero Trust Security in Kubernetes.



---

