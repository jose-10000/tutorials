# How to bring an application to kubernetes with gitops (ARGOCD)
On this tutorial I want to show a way for deploy an application to a kubernetes cluster.
We are going to use, docker, kompose(this tool convert a docker-compose file to kubernetes manifests), kubernetes(k3s) and argocd.
This time we are going to use kubectl, in the future we'll use k9s. It's a terminal based UI to interact with your Kubernetes clusters.

## What is gitops?
GitOps uses Git repositories as a single source of truth to deliver infrastructure as code. Submitted code checks the CI process, while the CD process checks and applies requirements for things like security, infrastructure as code, or any other boundaries set for the application framework. All changes to code are tracked, making updates easy while also providing version control should a rollback be needed.
- Reference: https://www.redhat.com/en/topics/devops/what-is-gitops

## What is ArgoCD?
Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes.
- Reference: https://argoproj.github.io/argo-cd/

## Prerequisites
- Knowledge about docker, kubernetes and git.
- Install docker, kubectl, kompose and argocd.
- A linux machine, or a virtual machine with a linux distribution, wsl is also an option, but I recommend using a virtual machine.

```bash

#######################
### Install Docker ####
#######################
# For ubuntu
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc


# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo gpasswd -a $USER docker

```

```bash
#######################
### Install kubectl ###
#######################
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

#######################
##### Install k3s #####
#######################
curl -sfL https://get.k3s.io | sh -

# Wait a few seconds and check if the pods are running or completed
kubectl get pods --all-namespaces

#######################
# Get the kubeconfig ##
#######################
mkdir -p ~/.kube
sudo cat /etc/rancher/k3s/k3s.yaml > ~/.kube/config

#######################
### Install kompose ###
#######################
curl -L https://github.com/kubernetes/kompose/releases/download/v1.33.0/kompose-linux-amd64 -o kompose
chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose

#######################
#### Deploy argocd ####
#######################
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait a few seconds and check if the pods are running.
kubectl get pods -n argocd

# Port forward, so we can access the argocd server, then open another terminal
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get the argocd admin password, and copy it
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Login. Open your browser and go to, then login with the password you copied, the username is admin.
http://localhost:8080

```

## Steps
1. Clone the repo, and go to the directory. You can make a fork of the repo, and clone your forked repo. https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo

```bash
git clone https://github.com/jose-10000/tutorials.git
cd tutorials/1
```

2. Change directory to the app directory, and build the docker image.

```bash
cd tutorials/1/docker/hello-nginx
sudo docker build -t hello-world:v1 .
```

3. Push the image to docker hub. You need to create an account on docker hub, and login. Change the username jose10000 to your username. https://docs.docker.com/security/for-developers/access-tokens/

```bash
sudo docker login
sudo docker tag hello-world:v1 jose10000/hello-world:v1
sudo docker push jose10000/hello-world:v1
```

  - 3.1 With docker compose we are going to verify if everything is fine. 

```bash
docker compose -f docker-compose.yaml up -d
```
  - Open your browser and go to http://localhost:8090

  - 3.2 Stop the containers.

```bash
docker compose -f docker-compose.yaml down
```

4.  Now we are going to convert the docker-compose file to kubernetes manifests, and move the files to the k8s directory. We'll try to maintain a tree folder structure.

```bash
kompose convert -f docker-compose.yaml
mv hello-world-deployment.yaml ../../k8s/deployments/hello-nginx/
mv hello-world-service.yaml ../../k8s/deployments/hello-nginx/

```
5. Change directory to the k8s directory, and apply the manifests, then check if the pods are running and make a port forward to access the service.

```bash

# Go to the k8s directory
cd ../../k8s/deployments/hello-nginx/
# Or, you can use this command, depending on where you are.
cd tutorials/1/k8s/deployments/hello-nginx 
# Apply the manifests, the dot means  apply all the files in the directory, you can also use -f to specify a file, or a directory.
kubectl apply -f .
# Check if the pods are running
kubectl get pods
# Port forward, so we can access the service, then open another terminal
kubectl port-forward svc/hello-world  8091:8090

# Check if the service is running, open your browser and go to http://localhost:8091
```

- 5.1 If everything is fine, you can delete the resources.

```bash
kubectl delete -f .
```

6. In order to use ArgoCD  we need to create an application, and a repository. We are going to use the same repository, and the same application name. 
- On the argocd/hello-nginx folder you can find the application.yaml file.

```bash
# First push the k8s directory to the repository, you can use your forked repo. Go to your top level directory(tutorials), and push the changes.
git add .
git commit -m "Add k8s manifests"
git push origin main
```








# What next?
- For a better understanding of the different parts that are part of a Kubernetes cluster, I recommend reading this part of the official documentation. 
  - English https://kubernetes.io/docs/concepts/overview/working-with-objects/
  -  Spanish https://kubernetes.io/es/docs/concepts/overview/working-with-objects/
