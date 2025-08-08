# Kubernetes Deployment Instructions for Django ToDo App

## 1. How to Deploy the App to Kubernetes

1. **Create the Namespace**

   The manifests use the `mateapp` namespace. Create it if it does not exist:

   ```sh
   kubectl create namespace mateapp
   ```

2. **Apply the Deployment, ClusterIP and HPA Manifests**

   ```sh
   kubectl apply -f .infrastructure/deployment.yml
   kubectl apply -f .infrastructure/clusterIp.yml
   kubectl apply -f .infrastructure/hpa.yml
   ```

3. **(Optional) Expose the App**

   To access the app externally, you may need to create a NodePort service:

   ```sh
   kubectl apply -f .infrastructure/nodeport.yml
   ```

4. **Check Status**

   ```sh
   kubectl get pods -n mateapp
   kubectl get hpa -n mateapp
   ```

## 2. Resource Requests and Limits

- **Requests:**
    - Memory: `64Mi`
    - CPU: `60m`
- **Limits:**
    - Memory: `128Mi`
    - CPU: `120m`

**Reasoning:**

- The app is lightweight (Django, small user base), so low resource requests are sufficient for idle/normal operation.
- Limits prevent a single pod from consuming excessive resources, ensuring fair usage and cluster stability.
- These values are based on typical usage for small Django apps and can be tuned as needed.

## 3. HPA (Horizontal Pod Autoscaler) Configuration

- **minReplicas:** 2
- **maxReplicas:** 5
- **Metrics:** CPU and Memory utilization, both at 70%

**Reasoning:**

- Minimum of 2 pods ensures high availability and zero downtime during rolling updates.
- Maximum of 5 pods allows scaling up under load, but prevents resource exhaustion.
- Autoscaling on both CPU and memory ensures the app responds to different types of load (CPU-bound or memory-bound).
- 70% utilization is a common threshold to trigger scaling before resources are saturated.

## 4. Deployment Strategy Configuration

- **Type:** RollingUpdate
- **maxUnavailable:** 1
- **maxSurge:** 1

**Reasoning:**

- RollingUpdate ensures zero downtime by updating pods incrementally.
- `maxUnavailable: 1` means at least one pod is always available during updates.
- `maxSurge: 1` allows one extra pod above the desired count during updates, speeding up rollout while maintaining
  availability.
- These settings balance availability and resource usage during deployments.

## 5. Accessing the App

- Two service manifests are provided:
    - `nodeport.yml` exposes the app on a NodePort (default: 30080). Use this for external access.
    - `clusterIp.yml` exposes the app internally within the cluster. Use this for internal communication between
      services.

### Access via NodePort

- Apply the NodePort service:
  ```sh
  kubectl apply -f .infrastructure/nodeport.yml
  ```
- Find the Node IP and port:
  ```sh
  kubectl get service todoapp -n mateapp
  ```
- Access the app at: `http://<NodeIP>:30080`

### Access via ClusterIP

- Apply the ClusterIP service:
  ```sh
  kubectl apply -f .infrastructure/clusterIp.yml
  ```
- The app will be accessible internally at `http://todoapp.todoapp.svc.cluster.local:80`
