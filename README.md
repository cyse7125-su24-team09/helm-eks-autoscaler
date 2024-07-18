# helm-eks-autoscaler
Sure, here is a README for your EKS Cluster Autoscaler Helm repository:

---

# EKS Cluster Autoscaler Helm Chart

This Helm chart deploys the Cluster Autoscaler on an Amazon EKS cluster. The Cluster Autoscaler automatically adjusts the size of the Kubernetes cluster when there are insufficient resources to schedule new pods or when nodes are underutilized.

## Prerequisites

- Kubernetes 1.12+
- Helm 3.0+
- An existing EKS cluster

## Installation

1. **Add Helm repository**

   ```sh
   helm repo add <repo-name> <repo-url>
   helm repo update
   ```

2. **Install the chart**

   ```sh
   helm install <release-name> <repo-name>/cluster-autoscaler -n kube-system
   ```

## Testing Autoscaler

1. **Deploy a resource-intensive application:**

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: resource-hog
   spec:
     replicas: 5
     selector:
       matchLabels:
         app: resource-hog
     template:
       metadata:
         labels:
           app: resource-hog
       spec:
         containers:
         - name: busybox
           image: busybox
           resources:
             requests:
               cpu: "1000m"
               memory: "2Gi"
             limits:
               cpu: "1000m"
               memory: "2Gi"
           command: ["sh", "-c", "while true; do echo 'hogging resources'; sleep 30; done"]
   ```

2. **Monitor the cluster-autoscaler logs:**

   ```sh
   kubectl logs -f deployment/cluster-autoscaler -n kube-system
   ```

3. **Verify node scaling:**

   ```sh
   kubectl get nodes
   kubectl get pods
   ```
