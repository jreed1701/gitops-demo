# GitOps Demo

Demonstrate GitOps on K8S using ArgoCD & Ansible.

# Quick Start Guide

TODO - Give use instructions.

# Goals

1. Deploy ArgoCD to K8S.
2. Deploy a load balancer (LB) to K8S.
3. Deploy a simple Flask Web App to the K8S cluster, accessible via LB, and updated using ArgoCD

# Assumptions

1. A K8S Cluster is already deployed to a target environment.  This could be achieved via Infrastructure as Code (IaC); eg Terraform -> AWS -> EKS.
2. The target host is on a local area network.
3. The host which commands are issued from is separate from the target host.
4. The target host is secured via SSH key.

# Environment

TODO - Write this up

# Design

TODO - Write this up

# References

The following documentation is related to the technology used in this demonstration.

1. Load Balancer - Nginx Ingress Controller - https://docs.nginx.com/nginx-ingress-controller/
2. ArgoCD - https://argo-cd.readthedocs.io/en/stable/
3. K8S - https://kubernetes.io/docs/home/