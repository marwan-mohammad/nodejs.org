# Overview

This project involves deploying the `Node.js application` from the Node.js GitHub repository 
to a local Kubernetes cluster. The deployment will use `ArgoCD` for continuous delivery. We 
will set up a `Jenkins` pipeline to automate the build and deployment process.

# Prerequisites

Vm1:

• Virtual Machine (VM) for Jenkins • Docker installed on the Jenkins VM

VM2 :

• Minikube installed on the second VM • ArgoCD installed on Minikube

# Machine for jenkins and Docker 

# install Docker

```bash
sudo apt update
sudo apt install curl
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt -y install lsb-release gnupg apt-transport-https ca-certificates curl software-properties-common
sudo apt -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-registry
sudo usermod -aG docker $USER
newgrp docker
```

# Install Jenkins on local host

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install fontconfig openjdk-17-jre -y
sudo apt install -y jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

After writing the Docker File , enter the Jenkins GUI and build this script:

    pipeline {
        agent any
    
        stages {
            stage("Checkout code") {
                steps {
                    script {
                        if (!fileExists('nodejs.org')) {
                            sh 'git clone https://github.com/marwan-mohammad/nodejs.org'
                        }
                        dir('nodejs.org') {
                            sh 'git fetch origin'
                            sh 'git checkout main'
                            sh 'git pull'
                        }
                    }
                }
            }
            stage("Install dependencies") {
                steps {
                    dir('nodejs.org') {
                        sh 'npm ci'
                    }
                }
            }
            stage("Run unit testing") {
                steps {
                    dir('nodejs.org') {
                        sh 'npm run test'
                    }
                }
            }
            stage("Dockerize") {
                steps {
                    dir('nodejs.org') {
                        sh 'docker build -t marwanmuhammad/nodejs.org .'
                    }
                }
            }
            stage("Push Docker image") {
                steps {
                    dir('nodejs.org') {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                            sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                            sh 'docker push marwanmuhammad/nodejs.org'
                        }
                    }
                }
            }
        }
    }

make sure to configure your docker-hub credentials, the output will be like this:

![Screenshot from 2024-10-25 17-22-21](H:\Screenshots\Screenshot from 2024-10-25 17-22-21.png)

![Screenshot from 2024-10-25 17-22-40](H:\Screenshots\Screenshot from 2024-10-25 17-22-40.png)

![Screenshot from 2024-10-25 17-22-47](H:\Screenshots\Screenshot from 2024-10-25 17-22-47.png)

![Screenshot from 2024-10-25 17-23-08](H:\Screenshots\Screenshot from 2024-10-25 17-23-08.png)

![Screenshot from 2024-10-25 17-23-13](H:\Screenshots\Screenshot from 2024-10-25 17-23-13.png)

![Screenshot from 2024-10-25 17-23-27](H:\Screenshots\Screenshot from 2024-10-25 17-23-27.png)

# Check your DockerHub

![Screenshot 2024-10-25 194413](H:\Screenshots\Screenshot 2024-10-25 194413.png)


# Another VM for Minikube and kubectl and Argocd

# install minikube and kubectl 

```bash
Install Kubernetes on Ubuntu/Debian 
#### Install Docker First if Installed Skip these steps
sudo apt update
sudo apt install curl
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-registry 
sudo usermod -aG docker $USER
newgrp docker

######### Install MiniKube (kubernetes platform)
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
ls
sudo dpkg -i minikube_latest_amd64.deb

minikube start --driver=docker --nodes=2
sudo snap install kubectl --classic
minikube kubectl -- get pods
kubectl cluster-info
kubectl get pods
minikube addons list
minikube dashboard &
kubectl get nodes
kubectl get pods -A

### install argo cd
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

to get password

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

and portforward
kubectl port-forward svc/argocd-server -n argocd 8080:443

```

#  deploy your app  



![Screenshot (749)](H:\Screenshots\Screenshot (749).png)

![Screenshot (748)](H:\Screenshots\Screenshot (748).png)


### Congratulations!

![WhatsApp Image 2024-10-01 at 19.36.36_828a14cc](H:\Screenshots\WhatsApp Image 2024-10-01 at 19.36.36_828a14cc.jpg)``
