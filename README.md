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

The virtual environment must be installed and active. Please see the Development section of [Environments](./doc/environment.md) page for
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

The default password will get locked down in the next step.

### Secure ArgoCD

This playbook will change the argocd password.

**Pre-requisite:**
  - Make sure the port forward is still running: `kubectl port-forward svc/argocd-server -n argocd 8080:443`

**Steps**

1. Create a file: `touch vault.yml`
2. Add to the file `argocd_admin_password: "(your pass)"`
3. Run `ansible-vault encrypt vault.yml`
4. Ansible will prompt the user for a vault password if it needs to.  Do so and take note of it. The contents of the file will
   also become encrypted and unreadable.
5. Next run: `ansible-playbook -i inventories/development/hosts.yml secure_argocd.yml --ask-vault-pass`
   - NOTE - The `--ask-vault-pass` flag will prompt the user for the same password created in the previous step.
6. The playbook restarts the argocd pod, so rerun `kubectl port-forward svc/argocd-server -n argocd 8080:443` in a separate terminal.
7. (Optional) - Login to ArgoCD and verify the new password now works: `https://localhost:8080/`

### App Deployment

The playbook from this section will deploy an app via ArgoCD.  The app is the same app described in ArgoCD's getting
started guide.  Please see the reference section.  The app is deployed to the default kubernetes namespace.

**Pre-requisites:**
   - Make sure the port forward is still running: `kubectl port-forward svc/argocd-server -n argocd 8080:443`
   - The vault with `argocd_admin_password` must have already been created in the previous section.

**Steps**

1. Run `ansible-playbook -i inventories/development/hosts.yml deploy_app.yml --ask-vault-pass`
2. In a separate terminal, run `kubectl port-forward svc/guestbook-ui 8081:80`
3. Open two browser tabs:
  a. Go to: `https://localhost:8080/applications/argocd/guestbook` to see that ArgoCD deployed the test app.
  b. Go to: `http://localhost:8081/` to see the app home page itself.

# Design Information

Please see the [Design](./doc/design.md) section for information detailing how this project was designed and implemented.

# Future Work Considerations

1. Automatically deployed application updates - Setup ArgoCD app synchronization based on a github webhook.
2. Write a new inventory type and run all of the playbooks in a CI/CD environment. This would simulate deploying to
   a cluster other than Docker Desktop for Windows.

# References

The following documentation is related to the technology used in this demonstration.

1. Load Balancer - Nginx Ingress Controller - https://docs.nginx.com/nginx-ingress-controller/
2. ArgoCD - https://argo-cd.readthedocs.io/en/stable/
3. K8S - https://kubernetes.io/docs/home/
4. Ansible K8S Module - https://docs.ansible.com/ansible/latest/collections/kubernetes/core/k8s_module.html
