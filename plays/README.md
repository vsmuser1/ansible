# Setup Ansible Connectivity

When running Ansible through the CLI:
- login to Ansible host using a DEVAD/MONAD account
- create an SSH key, *if* it does not exist, by running `ssh-keygen`:
  ```shell
  $ ssh-keygen -o -a "100" -t "ed25519" -f "~/.ssh/id_ed25519" -C "DevOpsPlatformSupport@moneris.com"
  ```
- copy the public key to the target host by running:
  ```shell
  $ ssh-copy-id -i ~/.ssh/id_ed25519.pub {{user}}@{{target_ip}}
  ```
- add the target Nexus host IP address under `../inventory/{{env}}/hosts` file as shown below (example):
  ```
  [nexus]
  10.182.14.18
  ```

This will setup the connection between the Ansible host and the target Nexus host.

# How Are Playbooks Organized

The `plays` directory contains two playbooks one for installing and the other for uninstalling Nexus.

- `plays/`
  - `install.yaml` -- used for installing Nexus
  - `uninstall.yaml` -- used for uninstalling Nexus

## Verify Playbooks Before Execution

To identify syntax errors, use `--check` flag (replacing `{{env}}` with either `dev` or `prod`):
```shell
$ ansible-playbook plays/install.yaml --inventory inventory/{{env}} --user {{user}} --ask-vault-pass --check
Vault password:
```

## Run The Install Playbook

To install Nexus run the install playbook (replacing `{{env}}` with either `dev` or `prod`):
```shell
$ ansible-playbook plays/install.yaml --inventory inventory/{{env}} --user {{user}} --ask-vault-pass
Vault password:
```

### Example Installation Playbook

This is an example of how to construct the `install.yaml`:

```yaml
---
# Nexus OSS - Installation

- hosts: "nexus"
  become: yes
  gather_facts: true
  vars_files:
    - "../vault/{{ env }}/secrets.yaml"
  tasks:
    - name: "Print environment being provisioned"
      ansible.builtin.debug:
        msg:
          - "ENVIRONMENT: {{ env }}"
    - name: "Display all variables/facts known for a host"
      ansible.builtin.debug:
        var: hostvars[inventory_hostname]
        verbosity: 4
    - name: "Nexus OSS - Pre-installation"
      import_role:
        name: "nexus-pre-install"
    - name: "Nexus OSS - Installation"
      import_role:
        name: "nexus-install"
    - name: "Nexus OSS - Post-installation"
      import_role:
        name: "nexus-post-install"
```

## Post-installation Tasks

The Ansible playbook does as much as it can for the post-installs steps, but some steps are still manual.
Refer to the internal documentation for the manual steps.

## Run The Uninstall Playbook

To uninstall Nexus run the uninstall playbook (replacing `{{env}}` with either `dev` or `prod`):
```shell
$ ansible-playbook plays/uninstall.yaml --inventory inventory/{{env}} --user {{user}}
```
