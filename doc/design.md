# Project Design

The design of this project is broken into two parts; installation and GitOps Configuration.

# Goals

1. Deploy ArgoCD to K8S.
2. Deploy a load balancer (LB) to K8S.
3. Deploy a simple Flask Web App to the K8S cluster, accessible via LB, and updated using ArgoCD

# Assumptions

1. A K8S Cluster is already deployed to a target environment.  This could be achieved via Infrastructure as Code (IaC); eg Terraform -> AWS -> EKS.
2. The target host is on a local area network.
3. The host which commands are issued from is separate from the target host.
4. The OS of the target host runs Ubuntu (Debian flavor) linux.

## Installation

An ansible playbook will be used to deploy the necessary tools to a server. This playbook will primarily
install:

1. ArgoCD for GitOps
2. Nginx Ingress Controller for Load Balancing

### Ansible Setup

The goal will be to follow doc recommendations as much as possible. In the case of Ansible, the Sample directory layout was chosen. Please see [Sample Directory Layout](https://docs.ansible.com/ansible/latest/tips_tricks/sample_setup.html#sample-directory-layout) documentation.  The reason why this was chosen is for simplicity sake.

### Roles

To demonstrate breakdown of ansible roles, each object of installation will be its own role.

### Testing

Ansible may tell us if the software has been versioned, but a common concern is that the software was installed
at a specific version, so testing will not only verify the software was installed but also at the correct version.

### Security

There will ultimately be secrets which need protecting, so ansible vault will be used to guard them. The
following items will be protected by vault:

1. The target host's SSH key.
2. A GitHub personal access token.