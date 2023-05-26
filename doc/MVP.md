## AsciiArtify demo deployment 
##### *The RHEL 9.2 was selected for demonstration purposes*   

### Application declerative setup

Argo CD applications, projects and settings can be defined declaratively using Kubernetes manifests. These can be updated using `kubectl apply`, without needing to touch the `argocd` command-line tool.

The Application CRD is the Kubernetes resource object representing a deployed application instance in an environment. It is defined by two key pieces of information:
-   `source`  reference to the desired state in Git (repository, revision, path, environment)
-   `destination`  reference to the target cluster and namespace. For the cluster one of server or name can be used, but not both (which will result in an error). Under the hood when the server is missing, it is calculated based on the name and used for any operations.

#### An Argo CD application YAML file is as follows:

    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: demo
      namespace: argocd
    spec:
      project: default
      
      source:
        repoURL: https://github.com/den-vasyliev/go-demo-app.git
        targetRevision: HEAD 
        path: helm
      destination:
        server: https://kubernetes.default.svc 
        namespace: demo
    
      syncPolicy:
        syncOptions:
        - CreateNamespace=true  
    
        automated:
          selfHeal: true
          prune: true


- We want Argo CD to basically automatically sync any changes in the git repository, but default that is turned off, so if you change and push something to the Git repo Argo CD doesn’t automatically fetch that but we can enable it by using **automated** attribute  **(configure Argo CD to poll the changes in Git repo in 3 min intervals)**, and inside that automated attribute we have two more options.

- The first one, we can configure Argo CD to undo or overwrite any manual changes to the cluster, so if we did kubectl apply manually in the cluster Argo CD will basically overwrite and sync it with the Git repository state instead and we can enable it using a **selfHeal** attribute set  to **true** *(by default , changes made to the live cluster will not trigger automated sync (automatic self-healing))*

- And finally, if we rename a component for example or if we delete some *.yaml file we want also Argo CD to delete that component or that old component in the cluster as well, and that’s gonna be an attribute called **prune** and we gonna set it to **true**  (by default , automatic sync will not delete resources)

Now we can apply this with `kubectl apply -n argocd -f application.yaml` and Argo CD will start deploying the **AsciiArtify demo** application.

### Short demo:
[![asciicast](/shortdemo.png)](https://asciinema.org/a/587645)

![argocd](/demo.png)

![argocd](/demo1.png)

![argocd](/demo2.png)

![argocd](/demo3.png)

![argocd](/app-demo.png)

![argocd](/app-demo-pod.png)

> #### As a result AsciiArtify demo app is deployed to the Kubernetes cluster with Argo CD

