# Environments

The environment used will covered in this section.  In all cases, some form of linux is used as the operating system to
work with.  In this case, Windows Subsystem for Linux running Ubuntu will serve as the terminal.

## Testing & Development Environment.

Docker Desktop for Windows will be used for simple development and testing.  Docker Desktop has the ability to quickly deploy and reset a
Kubernetes cluster, so it's an easy way to work with K8S.  The user shall interact with K8S running on windows via WSL2 Ubuntu.

### Windows Gotcha

The [development](../inventories/development/host_vars/target_host.yml) inventory there is a hostname given to argocd.  Windows DNS and Kubernetes running on Docker Desktop for Windows don't align well.  The user must make an entry in the
windows hosts file.

1. If the hostname in target_host.yml is `localhost'
2. Next go to `C:\Windows\System32\drivers\etc` and edit `hosts` with notepad in admin mode.
3. Find the IP address next to `host.docker.internal`
4. Make a new line at the bottom that is: `(ip from previous step) localhost`

## WSL Information

For the purpose of this demonstration, all references to WSL refer to the second version of WSL, which is WSL2.

If the user requires information regarding WSL2 setup, please see: https://learn.microsoft.com/en-us/windows/wsl/install

### /etc/wsl.conf

```
[boot]
systemd=true

[automount]
enabled = true
root = /
options = "metadata,uid=1000,gid=1000,umask=077,fmask=11,case=off"
```

### Windows .wslconfig

No special settings required, only ensure that enough CPU/RAM has been allocated.

## CI/CD Environment

This environment will use `ubuntu-latest` with python 3.10 installed.
