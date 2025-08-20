# GitOps Demo

Demonstrate GitOps on K8S using ArgoCD & Ansible.

# Pre-Requisites

1. A WSL2 environment with Ubuntu installed and updated.
2. Docker Desktop for Windows with K8S enabled.
3. Ansible == v2.10.8
4. Python >= v3.10.x
5. ansible-lint >= 25.8.1

# Quick Start Guide

## Virtual Environment

The author prefers to use python in virtual environments for ease of management. In the case of this repo, the repo itself
doesn't need to be a python package, so requirements are installed manually.

1. python3 -m venv venv
2. . ./venv/bin/activate
3. pip install -r requirements.txt

## Running Playbooks

The virtual environment must be installed and active.

### Installation

See the Development section of [Environments](./doc/environment.md) page.

1. ansible-playbook -i inventories/development/hosts.yml install.yml --ask-become-pass
2. kubectl port-forward svc/argocd-server -n argocd 8080:443
3. Open ArgoCD from browser URL https://localhost:8080/
4. Click past SSL Error. This is due to using a self-signed cert from ArgoCD service.

# Design Information

Please see the [Design](./doc/design.md) section for information detailing how this project was designed and implemented.

# Future Work Considerations

1. The ansible playbook merely points out the initial admin password for argocd. Ansible could be automated to change it
   to a desired password provided by the user in a file or a secret carrying mechanism; eg K8S secret or Hashicorp Vault.

# References

The following documentation is related to the technology used in this demonstration.

1. Load Balancer - Nginx Ingress Controller - https://docs.nginx.com/nginx-ingress-controller/
2. ArgoCD - https://argo-cd.readthedocs.io/en/stable/
3. K8S - https://kubernetes.io/docs/home/
4. Ansible K8S Module - https://docs.ansible.com/ansible/latest/collections/kubernetes/core/k8s_module.html
