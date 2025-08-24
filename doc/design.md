# Project Design

The design of this project is broken into three parts; installation, security, and app deployment.  Each part
is served by specific ansible roles, which break up tasks into logical components.  A development inventory is
kept for the development environment used.

## Goals

1. Deploy ArgoCD to K8S.
2. Deploy a load balancer (LB) to K8S.
3. Deploy a simple Test App to the K8S cluster, accessible via load balancer, and deployed using ArgoCD

## Assumptions

1. A K8S Cluster is already deployed to a target environment.
2. The target host is on a local area network.
3. The host which commands are issued from is separate from the target host.
4. The OS of the target host runs Ubuntu (Debian flavor) linux. Ubuntu linux was used on WSL2.

## Load Balancer Choice

The Nginx Ingress Controller was chosen as a load balancer for maximum flexibility.  This particular piece of technology is
able to automatically detect when an K8S `Ingress` object is created, updated or deleted and make the necessary linkages automatically.
That means less maintenance and oversight is required.  All the developers have to do is make sure their applications
are exposed when deployed, and nginx will do the rest.  Nginx is also a very widely known tool, so that also aided in
selecting it over other options.

## Files & Folder Organization

This section will describe the file and folder organization selected for this demonstration.

```
.
├── LICENSE
├── README.md
├── ansible.cfg
├── ansible.log
├── doc
│   ├── design.md
│   └── environment.md
├── inventories
│   └── development
│       ├── host_vars
│       │   └── target_host.yml
│       └── hosts.yml
├── roles
│   ├── check_kubernetes_install
│   │   ├── defaults
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yaml
│   ├── deploy_app
│   ├── install_argocd
│   ├── install_argocd_cli
│   ├── install_nginx_ingress_controller
│   └── secure_argocd
├── install.yml
├── secure_argocd.yml
├── requirements.txt
├── deploy_app.yml
└── vault.yml
```

- Ansible is configured via `ansible.cfg` and it's simply logging terminal output of ansible to `ansible.log` for debug purposes.
- All markdown documentation is kept in the `doc` folder
- Since there could be multiple different types of hosts, inventories were kept in the folder-based structure.  That way host_vars
  could be applied separately.  This is advantageous because the software was developed with Docker Desktop for Windows, but may
  be used elsewhere, such as a true linux environment.
- Roles are kept separately.  Each role in this repository keeps default variables and tasks.
- The files `install.yml`, `secure_argocd.yml`, and `deploy_app.yml` are kept separately for the sake of the demonstration.  These
  are the three entry-point playbooks.  Some organizations prefer to keep playbooks like this in a separate `playbooks/` folder,
  but these are at the top-level for simplicity-sake.
- requirements.txt keep python virtual environment requirements for items needed to run inside of the playbooks/roles.

## Playbooks

There are three playbooks. The playbooks were broken up for demonstration purposes.

1. `install.yml` - Installs nginx and argocd.
2. `secure_argocd.yml` - Changes the argocd password to something more secure.
3. `deploy_app.yml` - Deploys a test app via argocd.

## Roles & Variables

Roles are intended to provide reusable segments of ansible tasks which may be used in any other playbook.  They are used here to
show logical separation of tasks as well.  Each roles folder has a `defaults` and `tasks` folder.  The tasks are parameterized by
variable in the defaults folders.  Those parameters are for items, such as, kubectl path.  This path, among other parameterized variables,
might be different host-to-host.  Therefore, the `host_vars` in each inventory may override the default variable for each role.  In this
demonstration, this is used in several cases.  Please see [target_host.yml](../inventories/development/host_vars/target_host.yml) to
see which variables are overridden in this repository for the Docker Desktop for Windows K8S environment.

### Roles

1. `check_kubernetes_install` - Verifies kubernetes is installed and kubectl is at the expected version.
2. `deploy_app` - Deploys a test application via ArgoCD
3. `install_argocd` - Installs ArgoCD to K8S cluster.
4. `install_argocd_cli` - Installs the ArgoCD cli tool to the target host.
5. `install_nginx_ingress_controller` - Installs the nginx ingress controller to the K8S cluster.
6. `secure_argocd` - Changes the password of argocd from the initial to the

## Testing

Testing was accomplished by two methods. The first method of testing takes place in the ansible playbooks themselves.  There are opportunities
to verify certain truths.  The second method of testing is in a CI/CD pipeline.  This demonstration only tests code quality (aka linting) in its
CI/CD pipeline.  In the case of ansible, that is `ansible-lint`.  The following is additional testing details and the sort of things that were
caught as a result.

### Playbook Tasks

Instead of simply installing items and moving on, the author opted to also make sure the desired resources were actually installed as expected for
error checking.

1. The `check_kubernetes_install` role verifies K8S is installed and at the expected version.
2. The `install_nginx_ingress_controller` role has a task which verifies that the nginx ingress service is created, and that the pod image version
   matches what's expected.
3. The `install_argocd` role has a task which verifies the argocd ingress and pods were actually created.
4. The `deploy_app` role has a task which verifies the test application was actually deployed by argocd.

### CI/CD Tests

The `ansible-lint` and syntax checker were used to ensure good code quality.  The `ansible-lint` check is configured with the file `.ansible-lint.yml` and
through testing, the linter caught some idempotency misses.  Idempotency means that each time, every time, the ansible playbook is run the results are the
same.

## Error Handling

Error handling was built into the ansible playbook roles in the form of verifying task, such as making sure deployed pods were actually running on the cluster.

## Idempotence

All playbooks were written to be as idempotent as possible to ensure that ansible performed exactly the required behavior the first time, and then each subsequent
time has no further effect.  The goal is to achieve the desired state.

**Not Idempotent**

This task in the `install_argocd_cli` role is not idempotent because the command arbitrarily moves the downloaded argocd file to the installation path regardless.

```
- name: Move ArgoCD CLI to install path
  ansible.builtin.command:
    cmd: mv /tmp/argocd {{ argocd_cli_install_path }}
  become: true
```

**Idempotent**

This task in the `install_argocd_cli` role is idempotent because the command uses the `creates` qualifier and only moves the file in place if it's not already there.

```
- name: Move ArgoCD CLI to install path
  ansible.builtin.command:
    cmd: mv /tmp/argocd {{ argocd_cli_install_path }}
    creates: "{{ argocd_cli_install_path }}"
  become: true
```

Doc Reference: https://docs.ansible.com/ansible/2.10/collections/ansible/builtin/command_module.html#parameter-creates

## Security Discussion

Ansible Vault was used to secure the argocd app deployment.  The user creates a file `vault.yml` and populates it with the desired password, and the ansible
playbook [secure_argocd.yml](../secure_argocd.yml) changes the password from the initial admin password to the desired password.  This is accomplished by
adjusting the K8S secret for argocd.  When finished, the playbook also deletes the initial password K8S secret because it is no longer needed.

In addition to protecting the argocd deployment, if this demonstration were to be executed using an inventory to a remote host, then the ansible vault would
also need to protect the SSH keys.  Similarly any repository information may also need to be stored in the vault if ArgoCD was to be automatically configured
with a repository.