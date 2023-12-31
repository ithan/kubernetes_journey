### Blog-Like Guide: Installing K3s and ArgoCD

Welcome to my personal Journey to DevOps! In this blog-like guide, I'll be documenting my journey to DevOps, starting with the installation of K3s and ArgoCD. This guide is meant to be a quick reference for myself, but I hope it can be useful to others as well.

#### Setting Up K3s
K3s is a lightweight Kubernetes distribution, perfect for edge, IoT, and CI/CD environments. Here's how to get it up and running:

# Quick Reference

## Useful links
- [K3s Documentation](https://rancher.com/docs/k3s/latest/en/)
- [ArgoCD Documentation](https://argoproj.github.io/argo-cd/)
- [Gitea Documentation](https://docs.gitea.io/en-us/)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Helm Cheat Sheet](https://helm.sh/docs/helm/helm/)
- [K9s cheat sheet](https://www.hackingnote.com/en/cheatsheets/k9s/)


## Useful tools

- [K9s](https://k9scli.io/): A terminal-based UI for interacting with your Kubernetes clusters.
- [Gitea](https://gitea.io/en-us/): A lightweight self-hosted Git service.
- [ArgoCD](https://argoproj.github.io/argo-cd/): A declarative, GitOps continuous delivery tool for Kubernetes.
- [K3s](https://k3s.io/): A lightweight Kubernetes distribution, perfect for edge, IoT, and CI/CD environments.
- [Helm](https://helm.sh/): A package manager for Kubernetes.
- [Kubectl](https://kubernetes.io/docs/reference/kubectl/overview/): The Kubernetes command-line tool.

## Useful commands
- [usefull_commands.md](USEFULL_COMMANDS.md)
    
## Quick Reference
- [quick_reference.md](QUICK_REFERENCE.md)




## Blog-Like Guide: Installing K3s and ArgoCD


1. **Install K3s**: First up, we're going to install K3s. This is as simple as running a single command:
   ```bash
   curl -sfL https://get.k3s.io | sh -
   ```
   This script downloads and executes the K3s installer.

2. **Configure Kubernetes**: After installation, you need to configure your Kubernetes settings. This often involves setting the correct permissions for the `k3s.yaml` file:

   ### Warning! Don't execute this yet, read the note below first.
   ```bash
   sudo chmod 644 /etc/rancher/k3s/k3s.yaml
   ```
   Also, remember to set the `KUBECONFIG` environment variable:
   ```bash
   export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
   ```

   :warning: **Note**: This is what I did, but I think this is not the best way to do it. I think it's better to copy the k3s.yaml file to your home directory and then set the KUBECONFIG environment variable to point to that file. This way you don't have to use sudo to run kubectl commands.

   You can do this by running the following commands:
   ```bash
   cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
   export KUBECONFIG=~/.kube/config
   ```

3. **Verify Your Installation**: To ensure K3s has been installed successfully, you can check your node status:
   ```bash
   kubectl get nodes
   ```
#### Enabling Cert-Manager
Cert-Manager is a Kubernetes add-on that automates the management and issuance of TLS certificates from various issuing sources. Let's enable it in our cluster:

1. First we apply the cert manager YAML file:

   Warning: Read the documentation before applying the YAML file. The YAML file might change in the future.
   Docs: [cert-manager.io/docs/installation/kubernetes/](https://cert-manager.io/docs/getting-started/)

   ```bash

   ```bash
   kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.yaml
   ```

2. Then we check if the cert-manager pods are running:
   ```bash
   kubectl get pods --namespace cert-manager
   ```

   The output should be something like this:
   ```bash
   NAME                                       READY   STATUS    RESTARTS   AGE
   cert-manager-{id}             1/1     Running   0          2m
   cert-manager-{id}   1/1     Running   0          2m
   cert-manager-webhook-{id}      1/1     Running   0          2m
   ```
   If the pods are not running, you can check the logs of the pods to see what's wrong:
   ```bash
   kubectl logs cert-manager-{id} -n cert-manager
   kubectl logs cert-manager-cainjector{id} -n cert-manager
   kubectl logs cert-manager-webhook-{id} -n cert-manager
   ```

   If the pods are running, then you can check the cert-manager version:
   ```bash
   kubectl describe pod cert-manager-{id} -n cert-manager | grep Image:
   ```
   

   Then we can apply the ClusterIssuer YAML file, we will apply 2, one for staging and one for production:
   ```bash
   kubectl apply -f cluster-issuer-staging.yaml
   kubectl apply -f cluster-issuer-prod.yaml
   ```

   Then we can check if the ClusterIssuers are running:
   ```bash
   kubectl get clusterissuers
   ```
   The output should be something like this:
   ```bash
   NAME              READY   AGE
   letsencrypt-staging   True    2m
   letsencrypt-prod      True    2m
   ```

## Installing ArgoCD
ArgoCD is a declarative, GitOps continuous delivery tool for Kubernetes. Let's dive into its setup:

1. **Namespace Creation**: ArgoCD will live in its own namespace for better organization and management. Create it using:
   ```bash
   kubectl create namespace argocd
   ```

2. **Deploy ArgoCD**: Deploy ArgoCD into your cluster with the following command:
   ```bash
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. **Accessing ArgoCD**: Once ArgoCD is installed, you'll need to expose it. This is typically done via an Ingress resource. Apply your Ingress configuration with:
   ```bash
   kubectl apply -f ingress.yaml
   ```

4. **Initial Configuration**: Retrieve the initial admin password for ArgoCD:
   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
   ```

5. **Using ArgoCD**: With ArgoCD installed and accessed, you're now ready to use it for deploying and managing applications in your Kubernetes clusters.
### Notes
- Since the master has only 2GB of RAM, and I wanted to install Argo, I had to find a way to tell kubernetes to use my node that has 8GB of ram to deploy argo.
For that I used the following command:
```bash
 kubectl label nodes k3s-agent-2 high-memory=true
```

where k3s-agent-2 is the name of the node that has 8GB of RAM.

Then I use the following commands to change the argo that I installed using the install.yaml file to use the node with 8GB of RAM:

| This will add a nodeSelector to the argocd-server deployment:
```bash
kubectl patch deployment argocd-server -n argocd --type='json' -p='[{"op": "add", "path": "/spec/template/spec/nodeSelector", "value": {"high-memory": "true"}}]'
```
   
| This will add resource limits to the argocd-server deployment:
```bash	
kubectl patch deployment argocd-server -n argocd --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/resources", "value": {"requests": {"memory": "1Gi", "cpu": "500m"}, "limits": {"memory": "4Gi", "cpu": "2000m"}}]'
```

Theoretically, this will make argo use the node with 8GB of RAM, but I'm not sure if it's working or not yet.

We can verify that argo is using the node with 8GB of RAM by running the following command:
```bash
kubectl get pods -n argocd -o wide
```

Then we check the node that each pod is running on. 
Argocd-server should display:
```bash
Name                  Ready   Status    Restarts   Age   IP           Node           NOMINATED NODE   READINESS GATES
argocd-server-{id}    1/1     Running   0          10m   {ip}         k3s-agent-2    <none>           <none>
```


## Bonus: Installing Gitea
Gitea is a lightweight self-hosted Git service. Here's a quick guide to install it in your K3s setup:

TODO: Add a guide to install gitea in k3s.