# GitOps Demo

Demonstrate GitOps on K8S using ArgoCD & Ansible.

# Pre-Requisites

1. A WSL2 environment with Ubuntu installed and updated.
2. Docker Desktop for Windows with K8S v1.32.2 enabled.
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

The virtual environment must be installed and active. Plase see the Development section of [Environments](./doc/environment.md) page for
information about the test environment.

### Installation

The playbook from this section will install the nginx ingress controller, and ArgoCD.

1. Run `ansible-playbook -i inventories/development/hosts.yml install.yml --ask-become-pass`
2. At the end of the playbook, ansible will print out the default user password.  This will be addressed
   later.  For now, only take note of it.
3. In a separate terminal, run `kubectl port-forward svc/argocd-server -n argocd 8080:443`
4. Open ArgoCD from browser pointed at url: `https://localhost:8080/`
5. Click past SSL Error. This is due to using a self-signed cert from ArgoCD service.
6. Login with the initial username and password from the previous section.

The default password will get locked down at a later step.

#### App Deployment

The playbook from this section will deploy an app via ArgoCD.  The app is the same app described in ArgoCD's getting
started guide.  Please see the reference section.

**Pre-requisite:** - Make sure the port forward is still running: `kubectl port-forward svc/argocd-server -n argocd 8080:443`

1. Run `ansible-playbook -i inventories/development/hosts.yml deploy_app.yml`
2. In a separate terminal, run `kubectl port-forward svc/guestbook-ui 8081:80`
3. Open two browser tabs:
  a. Go to: `https://localhost:8080/applications/argocd/guestbook` to see that ArgoCD delployed the test app.
  b. Go to: `http://localhost:8081/` to see the app home page itself.


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
