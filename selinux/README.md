## SELinux module for containerized Omni

This module can be used to restrict the Omni container from accessing other services and files on the host. The policy was generated using udica and manually adjusted to allow the internal connections required to manage cluster nodes and serve the Omni web UI.
**This is alpha-level configuration, try it in a test environment first.**

### Dependencies

 - [policycoreutils](https://github.com/SELinuxProject/selinux)
 - [udica-templates](https://github.com/containers/udica) (part of container-selinux on Fedora/RHEL)

Installing dependencies on Fedora:
`dnf install policycoreutils container-selinux`

### Usage

Install the SELinux module:

`semodule -i omni.cil /usr/share/udica/templates/{base,net}_container.cil`

Label the ports used by the Omni service. Edit `omni-ports.semanage` first to match your setup.

`semanage import -f omni-ports.semanage`

Run the container with SELinux type `omni.process`. See below for some hints on how to do this. 

#### Using `podman run`

This could also work with `docker run`, but currently untested.

Add `--security-opt=label=type:omni.process` to set the correct label.

#### Using Kubernetes YAML (`podman kube play`)

Use the following security settings in the pod template spec:

```yaml
  securityContext:
    capabilities:
      add:
        - CAP_NET_ADMIN
    sysctls:
      - name: net.ipv4.conf.all.src_valid_mark
        value: 1
    seLinuxOptions:
      type: omni.process
```

### Testing status

Manually tested working on:

 - AlmaLinux 9 (2024-11-14), using Podman
 - Fedora 41 (2024-11-13), using Podman
