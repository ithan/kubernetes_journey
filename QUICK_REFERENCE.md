# Useful Commands Cheat Sheet

### Initial Setup and Git Operations
- **Install K3s:** 
  ```bash
  curl -sfL https://get.k3s.io | sh -
  ```

### Additional Important Commands
- **Set KUBECONFIG:** 
  ```bash
  export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
  ```

### Kubernetes Operations
- **Apply a YAML File:** 
  ```bash
  kubectl apply -f [filename.yaml]
  ```
- **Display Resources:** 
  ```bash
  kubectl get [resource]
  ```
- **Show Detailed Resource Info:** 
  ```bash
  kubectl describe [resource] [name]
  ```
- **Delete Resources:** 
  ```bash
  kubectl delete -f [filename.yaml]
  ```

### Helm for Package Management
- **Install a Chart:** 
  ```bash
  helm install [name] [chart]
  ```
- **Uninstall a Chart:** 
  ```bash
  helm uninstall [name]
  ```
- **Add a Helm Repository:** 
  ```bash
  helm repo add [name] [URL]
  ```

### Namespace and Secret Management
- **Create a Namespace:** 
  ```bash
  kubectl create namespace [name]
  ```
- **Get a Secret:** 
  ```bash
  kubectl -n [namespace] get secret [name]
  ```

### Ingress and Certificates
- **List Ingresses:** 
  ```bash
  kubectl get ingress -n [namespace]
  ```
- **List Certificates:** 
  ```bash
  kubectl get certificates -n [namespace]
  ```

### Debugging and Logs
- **View Pod Logs:** 
  ```bash
  kubectl logs [pod-name] -n [namespace]
  ```
- **Get Events:** 
  ```bash
  kubectl get events -n [namespace]
  ```

### Storage and Node Operations
- **Disk Space Usage:** 
  ```bash
  df -h
  ```
- **List Cluster Nodes:** 
  ```bash
  kubectl get nodes
  ```


```

