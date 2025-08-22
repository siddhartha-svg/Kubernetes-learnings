````markdown
## Kubernetes Learning Plan

### 📅 Day 1 – Pod Disruption Budgets (PDBs)

**Concept**: Pod Disruption Budgets (PDBs) prevent all replicas of your application from going down during voluntary disruptions like node upgrades or maintenance. It ensures a minimum number of pods are available at all times.

**Practice**:
1.  Create a Deployment with 3 replicas.
2.  Apply a PDB allowing `minAvailable: 2`.
3.  Drain a node and observe how the PDB keeps at least 2 pods alive.

**YAML (pdb.yaml)**:
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
````

**Commands**:

```bash
kubectl apply -f pdb.yaml
kubectl get pdb
kubectl drain <node-name> --ignore-daemonsets --force
```

-----

### 📅 Day 2 – Resource Requests & Limits

**Concept**:

  * **Requests**: Guaranteed resources for a pod.
  * **Limits**: The maximum resources a pod can use.
    Using both prevents a single pod from consuming too many resources and impacting other workloads.

**Practice**:

1.  Create a pod with requests (`200m` CPU, `256Mi` memory) and limits (`500m` CPU, `512Mi` memory).
2.  Observe its behavior; if the application uses too much memory, it will be `OOMKilled` (Out of Memory Killed).

**YAML (resources.yaml)**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resourced-pod
spec:
  containers:
  - name: stress-container
    image: polinux/stress
    resources:
      requests:
        memory: "256Mi"
        cpu: "200m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

**Commands**:

```bash
kubectl apply -f resources.yaml
kubectl describe pod resourced-pod
kubectl top pod resourced-pod
```

-----

### 📅 Day 3 – Network Policies

**Concept**: By default, all pods can communicate with each other. Network Policies are a security measure that allows you to restrict traffic between pods based on labels, preventing unwanted connections.

**Practice**:

1.  Create two pods (one labeled `frontend`, one `backend`).
2.  Apply a Network Policy that only allows traffic from pods with the `frontend` label to the `backend` pod.

**YAML (network-policy.yaml)**:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

**Commands**:

```bash
kubectl apply -f network-policy.yaml
kubectl get networkpolicy
```

-----

### 📅 Day 4 – Init Containers

**Concept**: Init Containers are specialized containers that run and complete before the main application container starts. They are useful for pre-flight setup tasks like waiting for a database to be ready, downloading configuration files, or setting up file permissions.

**Practice**:

1.  Create a pod with an Init Container that sleeps for 10 seconds before the main NGINX container begins.
2.  Observe the pod's lifecycle to see the Init Container running first.

**YAML (init-container.yaml)**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  initContainers:
  - name: init-wait
    image: busybox
    command: ['sh', '-c', 'echo waiting... && sleep 10']
  containers:
  - name: main-app
    image: nginx
```

**Commands**:

```bash
kubectl apply -f init-container.yaml
kubectl describe pod init-demo
```

-----

### 📅 Day 5 – Autoscaling (HPA & VPA)

**Concept**:

  * **HPA (Horizontal Pod Autoscaler)**: Automatically scales the number of pod replicas up or down based on metrics like CPU or memory utilization.
  * **VPA (Vertical Pod Autoscaler)**: Automatically adjusts the resource requests and limits of a pod.

**Practice**:

1.  Deploy an application with CPU requests.
2.  Apply an HPA to scale the application between 2 and 5 replicas, based on the CPU usage staying around 50%.

**YAML (hpa.yaml)**:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

**Commands**:

```bash
kubectl apply -f hpa.yaml
kubectl get hpa
kubectl describe hpa myapp-hpa
```

-----

**✅ By the end of these 5 days, you’ll know**:

  * **Availability**: Pod Disruption Budgets (PDBs)
  * **Stability**: Resource Requests & Limits
  * **Security**: Network Policies
  * **Reliability**: Init Containers
  * **Scalability**: HPA & VPA

<!-- end list -->

```
```
