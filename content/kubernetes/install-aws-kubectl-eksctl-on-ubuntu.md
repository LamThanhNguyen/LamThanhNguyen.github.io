---
date: 2025-06-08
description: "Step-by-step guide to installing AWS CLI, kubectl, and eksctl on Ubuntu 22.04.3 LTS (WSL)."
featured_image: "/images/install-aws-kubectl-eksctl-cli.png"
tags: ["kubernetes", "aws", "eks", "kubectl", "cli"]
title: "I. Install AWS CLI, kubectl, and eksctl"
---

Ubuntu 22.04.3 LTS - Windows Subsystem for Linux - AMD64

## 1. Install & Config AWS CLI
```bash
sudo snap install aws-cli --classic
aws --version
which aws
```
Configure AWS account
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

kubectl acts as a client to the Kubernetes API server, sending commands which are executed by the cluster to manage resources like pods, deployments, and services.

I installed kubectl CLI by following the official [AWS documentation for setting up kubectl & eksctl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html#linux_amd64_kubectl/ "Visit aws setup kubectl & eksctl!"). Ensure you check this documentation to verify the latest version and download links.

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
Q: Why do we already have `kubectl`, and then also have `eksctl`? Is it a duplicate feature?

A:
While `kubectl` and `eksctl` may seem similar because they both relate to Kubernetes and EKS, they serve very different purposes. They are not duplicates, but rather complementary tools within the EKS ecosystem:

**"`eksctl` manages infrastructure (clusters, node groups, IAM roles)."**

**"`kubectl` manages workloads (pods, deployments, services) on a running Kubernetes cluster."**

Analogy:

**"`eksctl` is like the construction crew, building the infrastructure of your Kubernetes cluster."**

**"`kubectl` is the interior decorator and maintenance staff, managing resources and workloads within the Kubernetes cluster."**

## 5. Install Helm
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
You can verify your helm:
```bash
helm version --template='{{ .Version }}{{ "\n" }}'
```

## 6. Install k9s
```bash
curl -LO https://github.com/derailed/k9s/releases/download/v0.50.6/k9s_Linux_amd64.tar.gz
tar -xzf k9s_Linux_amd64.tar.gz
sudo mv k9s /usr/local/bin/
rm k9s_Linux_amd64.tar.gz
```
You can verify your k9s:
```bash
k9s version
```
Some usefuls commands:
```bash
k9s
:ns
:service
:pods
esc
quit
```