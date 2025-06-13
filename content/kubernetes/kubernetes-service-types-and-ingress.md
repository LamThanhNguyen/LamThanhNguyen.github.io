---
date: 2025-06-12T10:58:08-04:00
description: "Understanding Kubernetes Service Types & Ingress"
featured_image: "/images/k8s-service-types-and-ingress.png"
tags: ["kubernetes", "aws", "load balancer", "service type", "ingress"]
title: "II. Understanding Kubernetes Service Types & Ingress"
---

## 1. Introduction
- **Why exposing services matters in Kubernetes**: 
    - Your workloads are useless if no one can reach them. Service abstraction lets you decouple pod lifecycles from stable endpoints.

- Overview of Service types vs. Ingress:
    - Services give every workload an IP; Ingress turns a set of Services into a coherent, externally reachable API surface.

## 2. Kubernetes Service Types Explained
### a. ClusterIP
    - Exposes the Service on a cluster-internal IP
    - Accessible **only inside** the cluster
    - Typical use: 
        - Internal-only services 
        - Microservices talking to each other
### b. NodePort
    - Exposes the Service on each node’s IP at a static port (the NodePort)
    - Accessible from outside the cluster: same VPC or the internet (depending on firewall/Security Group rules)
    - Reachable via **NodeIP:NodePort** (port range 30000 - 32767)
    - Typical use:
        - Simple setups, debugging, or in environments without cloud LBs. 
        - Quick test or direct access, not recommended for production internet-facing apps.
### c. LoadBalancer
    - Creates a cloud load balancer (cloud provider integration, e.g., AWS ALB).
    - Assigns a public IP and routes traffic to your Service.
    - Accessible from the internet.
    - Typical use: exposing production workloads to the internet (websites, APIs).
### d. ExternalName
    - Maps the Service to an external DNS name.
    - Used for routing cluster traffic to resources outside Kubernetes.
    - Not used for exposing services to the internet.
    - Typical use: referencing services outside the cluster by DNS name(database,SaaS endpoints).

## 3. Ingress
- Ingress is a Kubernetes resource that manages external HTTP/HTTPS traffic
    - Receives incoming requests from outside the cluster
    - Routes them to the right Service based on rules (URL path, host, headers…)
- Typical flow: User → ALB/NLB/NGINX (provisioned by the Ingress Controller) → Service → Pods.
- Ingress Controller:
    - A pod/deployment running inside your cluster
    - monitors your Ingress resources.
    - configures and manages actual routing rules on a cloud load balancer
- Ingress Class:
    - a way to specify which Ingress Controller is responsible for handling a particular Ingress resource

## 4. Curious question
Q: Why Do You Need Ingress?
A: Without Ingress, to expose multiple applications or services externally, you would need many individual LoadBalancer services, which becomes expensive and complex quickly.

Ingress provides a unified, cost-effective way to expose multiple services through a single entry point.

Q: What is the benefits of using Ingress?
A:
    - Cost Efficiency: One load balancer, multiple services.
    - Centralized Management: Manage routing rules in a single place.
    - Advanced HTTP Routing: URL/host-based routing, SSL termination, header-based routing.
    - Scalability: Easily manage large-scale applications.

Q: When to Use Ingress?
A:
    - You have multiple services to expose externally.
    - You require advanced HTTP routing features (like SSL termination, URL path-based routing, etc.).
    - You aim to optimize costs and simplify networking.

Q: Why do we already have `LoadBalancer` service then we have `Ingress + ALB`?

A:
    - A LoadBalancer Service provisions one dedicated cloud load balancer per Service—perfect for a single workload but costly at scale.
    - In short, use LoadBalancer for quick, single‑service exposure or non‑HTTP protocols; use Ingress + ALB when you need to expose many HTTP services efficiently with advanced routing features.