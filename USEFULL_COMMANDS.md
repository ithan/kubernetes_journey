- **Install K3s:** 
  ```bash
  curl -sfL https://get.k3s.io | sh -
  ```
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
- **Install a Chart:**
    ```bash
    helm install [name] [chart] 
    ```
- **Install a chart with a custom values file:**
    ```bash
    helm install [name] [chart] -f [values.yaml]
    ```
- **Update a chart with a custom values file:**
    ```bash
    helm upgrade [name] [chart] -f [values.yaml]
    ```
- **Uninstall a Chart:**
    ```bash
    helm uninstall [name]
    ```
- **Add a Helm Repository:**
    ```bash
    helm repo add [name] [URL]
    ```
- **Create a Namespace:**
    ```bash
    kubectl create namespace [name]
    ```
- **Get a Secret:**
    ```bash 
    kubectl -n [namespace] get secret [name]
    ```
- **List Ingresses:**
    ```bash
    kubectl get ingress -n [namespace]
    ```
- **List Certificates:**
    ```bash
    kubectl get certificates -n [namespace]
    ```
- **List all resources in a namespace:**
    ```bash
    kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get -n [namespace]
    ```
- **List all resources in all namespaces:**
    ```bash
    kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get --all-namespaces
    ```
- **View Pod Logs:**
    ```bash
    kubectl logs [pod-name] -n [namespace]
    ```
- **Get Events:**
    ```bash
    kubectl get events -n [namespace]
    ```
- **Get all pods in a namespace sorted by restart count:**
    ```bash
    kubectl get pods -n [namespace] --sort-by=.status.containerStatuses[0].restartCount
    ```