## Installation of K3d, kubectl with an ArgoCD deployment on Kubernetes
##### *The RHEL 9.1 was selected for demonstration purposes*   

### K3d installation
 - wget:
 `wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash`
  - curl:
 `curl  -s  https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh  |  bash`
 ####
    k3d version

### kubectl installation

    cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
    enabled=1
    gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    EOF

    sudo dnf install -y kubectl
    alias ku=kubectl
    source <(kubectl complation bash|sed s/kubectl/ku/g)
    ku version
### Two worker nodes cluster creation

    k3d cluster create <nameofthecluster> --agents 2
    k3d cluster list
    k3d node list
    
### ArgoCD deployment

    ku create namespace argocd
    ku get ns
    ku apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)
    ku get all -n argocd
    ku port-forward svc/argocd-server -n argocd 8080:443&
    ku -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"| base64 -d; echo
  - Open a browser to the Argo CD internal UI, and login by visiting the `localhost:8080` in a browser 
  - Use the further credentials: **admin** for login and **password, which appeared into the terminal** after the last command you've issued following the instruction, for password
  - After logging in, click the **+ New App** button to create the application
  
  ![argocd](argocd.png)
 
 ### For handy interaction with ArgoCD applications via CLI, it is required to install the ArgoCD CLI
 
    curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
    sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
    rm argocd-linux-amd64
   
   - type `argocd admin initial-password -n argocd` to get current password
   - type `argocd login localhost:8080 --insecure --username <USERNAME> --password <PASSWORD>` to login via CLI
   - type `argocd account update-password` to change the password
   
   ### Creating the example guestbook application using ArgoCD CLI

    ku create ns guestbook
    argocd app create guestbook --repo https://github.com/argoproj/argocd-example-apps.git --path helm-guestbook --dest-server https://kubernetes.default.svc --dest-namespace guestbook
    argocd app sync guestbook
    argocd app get guestbook
    ku -n guestbook get deploy guestbook-helm-guestbook -o wide
    ku -n guestbook get svc
    ku -n guestbook get pod
    ku -n guestbook get ep
    ku port-forward -n guestbook svc/guestbook-helm-guestbook 8081:80&
    
  Now, when the application is deployed into the Kubernetes cluster, use `localhost:8081` in your browser to reach it
  
  ![guestbook](guestbook1.png)
  ![guestbook](guestbook.png)
  ![guestbook](guestbook2.png)
  
### To remove the argo-app, ArggoCD and your cluster, use the following commands:

    argocd app delete guestbook
    ku delete ns guestbook
    ku delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    k3d cluster delete <nameofthecluster>


> **If you require more advanced documentation in relation to ArgoCD use the following link to [get started](https://argo-cd.readthedocs.io/en/stable/getting_started/) with**
