### Blog-Like Guide: Installing K3s and ArgoCD

Welcome to my personal Journey to DevOps! In this blog-like guide, I'll be documenting my journey to DevOps, starting with the installation of K3s and ArgoCD. This guide is meant to be a quick reference for myself, but I hope it can be useful to others as well.

#### Setting Up K3s
K3s is a lightweight Kubernetes distribution, perfect for edge, IoT, and CI/CD environments. Here's how to get it up and running:

A I forgot before,so some details:

# Where are you installing this cluster?

I have a hetzner dedicated server with Ubuntu, ~65GB of RAM, 1TB of SSD and 4 cores, 8 threads each.

We pay ~60 euros per month for it. 

Do you need a server like this? hell no! k3s can run a raspberry pi, so you can get a raspberry pi and install k3s on it and you will be fine, or even just a virtual machine with 2GB of RAM and 2 cores will be fine.

but I plan to use this server for some light hosting so its worth it for me.

# What are you installing?
   I will let you know once I figure it out :D 
   for now:
   - ArgoCD - done
   - Gitea - done ( sort off? I need to figure out proper dabase setup eventually, no promises though )
   - Vault - wip
   - Some observability tools ( Prometheus, Grafana, Loki, Tempo ... ) will figure out the details later.
   - Some nodejs apps
   - Some build tools so we can push -> build -> deploy our apps in the same cluster.

# Quick note on mental sanity
   I'm not a devops engineer, I'm a full stack engineer that wants to learn devops, so I will make mistakes, I will do things wrong, I will do things that don't make sense, but I will learn from them and I will get better at this. So if you are reading this and you are a devops engineer, please don't judge me too hard, I'm trying my best :D but feel free to give me feedback and tell me what I'm doing wrong, everyone who may read this in the future will thank you for it.

   If you are starting your journey to devops with me, be aware! I tried this 2 time already, and I got frustrated and gave up the first time. Its not an easy journey, but if you take it one step at a time, and push yourself to at least have 1 thing working at a time, you will enjoy the journey and you will learn a lot.

   At least that's what is happening to me now! 

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
   sudo chmod 644 ~/.kube/config
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

   Now we want to create a namespace for cert-manager:

   Then we apply the cert-manager YAML file:

   ```bash
   kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.yaml
   ```

   I personally downloaded the YAML file and applied it using the following command:
   ```bash
   kubectl apply -f cert-manager.yaml
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

NOTE: Even though we call it ingress, its not actually an ingress, its an IngressRoute.
k3s uses Traefik as the default ingress controller, and Traefik uses IngressRoute instead of Ingress.

4. **Apply the certificate**
   ```bash
   kubectl apply -f certificate-production.yaml
   ```

5. **Initial Configuration**: Retrieve the initial admin password for ArgoCD:
   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
   ```

6. **Verify Installation**: Verify that ArgoCD is running with:
   ```bash
   kubectl get pods -n argocd
   ```
7. **Verify the ingress**
   ```bash
   kubectl get ingressroute -n argocd
   ```

If we try to use ArgoCD now, we will get to_many_redirects error. To fix this we will follow https://github.com/argoproj/argo-cd/issues/2953

Where they say that we need to set --insecure flag to argocd-server.

The easiest way to do this is to edit the argocd-server deployment and add the flag to the command.

To do this we run the following command:
```bash
kubectl edit deployment argocd-server -n argocd
```

This will open a vim editor, we need to find the line that starts with command: and add --insecure to the end of the line.
To search for the args line, we can use the following shortcut in vim:
```bash
/args
```

Then get yourself in edit mode by pressing i, and add --insecure to the end of the line.

I added:
```bash
   --insecure
   --/shared/app # This will enable the use of the shared app feature.
   --staticassets # This will enable the use of the static assets feature.
```

Then we can save and exit vim by pressing esc and then typing :wq and pressing enter.


8. **Using ArgoCD**: With ArgoCD installed and accessed, you're now ready to use it for deploying and managing applications in your Kubernetes clusters.
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


## Install helm
Helm is a package manager for Kubernetes. It allows you to deploy applications in your cluster with a single command. Here's how to install it:

:warning: **Note**: This may be outdated, check the [docs](https://helm.sh/docs/intro/install/) for the latest installation instructions.
1. **Install Helm**: First, install Helm using the following command:
   ```bash
   curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
   chmod 700 get_helm.sh
   ./get_helm.sh
   ```

2. **Verify Installation**: Verify that Helm is installed with:
   ```bash
   helm version
   ```
   The output should be something like this:
   ```bash
   version.BuildInfo{Version:"v3.13.1", GitCommit:"3547a4b5bf5edb5478ce352e18858d8a552a4110", GitTreeState:"clean", GoVersion:"go1.20.8"}
   ```

## Installing Gitea
Gitea is a lightweight self-hosted Git service. Here's a quick guide to install it in your K3s setup:

For this one, we will use helm with custom values.yaml file.

First lets add the help

1. **Add the helm repo**
   ```bash
   helm repo add gitea-charts https://dl.gitea.com/charts/
   ```
2. **Update the helm repo**
   ```bash
   helm repo update
   ```
3. **CD into the gitea-charts directory**
   ```bash
   cd gitea
   ```
:warning: **Note**: Here if you are not a lazy bastard as I am, you should setup a proper database infrastrcture, but since I'm still learning and I don't want to get into that yet, I will just use the default values and pray for the best.

4. **Install gitea**
   ```bash
   helm install gitea gitea-charts/gitea -f values.yaml -n gitea --create-namespace
   ```
   [ Updated to include -n gitea ] :D 

4.5 **Fix your mistake**
  If you are stupid like me, you will forgot to select the namespace when you install gitea, and you will end up with gitea in the default namespace. So lets uninstall it and reinstall it in the correct namespace.
  ```bash
   helm uninstall gitea
   helm install gitea gitea-charts/gitea -f values.yaml -n gitea --create-namespace
   ```
[ Updated to include --create-namespace, since I forgot to add it the first time ]

5. **Apply the certificate and the ingress**
   ```bash
   kubectl apply -f certificate-production.yaml
   kubectl apply -f ingress.yaml
   ```

6. **Verify the ingress**
   ```bash
   kubectl get ingressroute -n gitea
   ```

7. **Verify the certificate**
   ```bash
   kubectl get certificate -n gitea
   ```

TADA! You have gitea installed in your cluster and it should accessible from the internet. 
If its not accessible, well... GPT your error and hope for the best, at least that's what I do :D

but I will let github copilot fill some ideas on what can be wrong here.

8. **Debug the ingress**
   ```bash
   kubectl describe ingressroute -n gitea
   ```

9. **Debug the certificate**
   ```bash
   kubectl describe certificate -n gitea
   ```

10. **Debug the pods**
   ```bash
   kubectl get pods -n gitea
   kubectl describe pod -n gitea
   kubectl logs -n gitea
   ```
   If you want to see the logs of a specific pod, you can use the following command:
   ```bash
   kubectl logs -n gitea gitea-{id}
   ```
   where {id} is the id of the pod that you want to see the logs of.
   Remember, you can get the id of the pods by running:
   ```bash
   kubectl get pods -n gitea
   ```
For further steps, you can follow the official documentation: https://docs.gitea.com/installation/install-on-kubernetes 
I wish you the best of luck!

## Secrets secrets dubby doo, I've got a secret for you ( AKA Vault from HashiCorp )

Vault is a tool for securely accessing secrets. A secret is anything that you want to tightly control access to, such as API keys, passwords, certificates, and more. Vault provides a unified interface to any secret, while providing tight access control and recording a detailed audit log.

We will integrate this sucker with ArgoCD, so we can use it to store our secrets and then use them in our applications.
Its actually a super cool tool; but a pain to setup the permissions for it.

But I will let you deal with the permissions and suffer on your own, but I will give you some hints on how to do it.

First though, lets install vault.


