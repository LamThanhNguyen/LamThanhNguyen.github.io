---
date: 2025-06-08T10:58:08-04:00
description: "Kubernetes"
featured_image: "/images/install-aws-kubectl-eksctl-cli.png"
tags: ["kubernetes"]
title: "I. Install aws, kubeclt, eksclt CLIs"
---

Ubuntu 22.04.3 LTS - Windows Subsystem for Linux - AMD64

## 1. Install & Config AWS CLI
```bash
sudo snap install aws-cli --classic
aws --version
which aws
```
Config aws account
```bash
aws configure
AWS Access Key ID [None]: xxxxxxxxxxxx
AWS Secret Access Key [None]: xxxxxxxxxxxxxxxx+xxxxxx
Default region name [None]: ap-southeast-1
Default output format [None]: json
```
You can verify your aws account:
```bash
cat ~/.aws/credentials
cat ~/.aws/config
aws ec2 describe-vpcs
```

## 2. Install kubectl CLI
kubectl is the command-line interface (CLI) tool used to interact with Kubernetes clusters. It allows developers and operators to manage cluster resources, inspect states, deploy applications, and perform administrative tasks.

kubectl acts as a client to the Kubernetes API server. It sends commands to the API server, which then carries out the requested actions on the cluster's components such as pods, deployments, services, etc.


I'm install kubectl CLI following [aws setup kubectl & eksctl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html#linux_amd64_kubectl/ "Visit aws setup kubectl & eksctl!"). Please read the article to update the version and link download kubectl.
```bash
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.32.3/2025-04-17/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
```
You can verify your kubectl:
```bash
kubectl version --client
which kubectl
```
Config kubectl to communicate with EKS server
```bash
aws eks update-kubeconfig --region region-code --name my-cluster
cat ~/.kube/config
```
If you want manage multiple cluster and switch between contexts:
```bash
kubectl config get-contexts
kubectl config use-context <context-name>
kubectl config rename-context arn:aws:eks:ap-southeast-1:111122223333:cluster/dev dev
kubectl config use-context dev
kubectl config set-context dev --namespace=my-namespace
kubectl config current-context
aws sts get-caller-identity
```

## 3. Install eksctl CLI
eksctl is a command-line interface (CLI) tool for creating and managing Amazon EKS (Elastic Kubernetes Service) clusters.

```bash
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin
echo '. <(eksctl completion bash)' >> ~/.bashrc
source ~/.bashrc
```
You can verify your eksctl:
```bash
eksctl version
which eksctl
```

## 4. Curious question
Q: Why do we already have kubectl, and then we also have eksctl? Is it a duplicate feature?

A:
While kubectl and eksctl may seem similar because they both relate to Kubernetes and EKS, they serve very different purposes. They are not duplicates, but rather complementary tools in the EKS ecosystem.
- eksctl: Provision and manage the EKS infrastructure itself (cluster, nodegroups, IAM roles, etc.)
- kubectl: Interact with the Kubernetes cluster once it's already running (deploy apps, inspect pods, manage workloads)

Imagine you're building a house:
- eksctl is the construction crew: It lays the foundation, builds the walls, installs utilities — i.e., it sets up the cluster infrastructure.
- kubectl is the interior decorator and operator: It arranges furniture, fixes appliances, manages daily operations — i.e., it controls and manages the workloads inside the Kubernetes cluster.