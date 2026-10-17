# Ansible Playbook

## Project: CloudForge — Multi-Node Infrastructure Automation Using Ansible on AWS

This playbook installs Nginx on the two Amazon Linux managed nodes, starts the service, and enables it to start automatically after a reboot.

## Playbook

Save the following content as `playbooks/install-nginx.yml` in your existing project:

```yaml
---
- name: Install and start Nginx on managed nodes
  hosts: nodes
  become: yes

  tasks:
    - name: Install Nginx
      dnf:
        name: nginx
        state: present

    - name: Start Nginx and enable it at boot
      systemd:
        name: nginx
        state: started
        enabled: yes
```

## Explanation

| Setting | Meaning |
| --- | --- |
| `hosts: nodes` | Runs the playbook on the servers in the existing `nodes` inventory group. |
| `become: yes` | Uses sudo privileges to install packages and manage services. |
| `dnf` | Manages packages on the Amazon Linux nodes. |
| `state: present` | Ensures the Nginx package is installed. |
| `systemd` | Manages the Nginx service. |
| `state: started` | Ensures Nginx is running. |
| `enabled: yes` | Enables Nginx to start automatically after a reboot. |

## Run the Playbook

Run these commands from the existing Ansible controller:

```bash
cd ~/ansible-project
ansible-playbook playbooks/install-nginx.yml
```

If your existing `group_vars/nodes/vault.yml` is encrypted and no other Vault password source is configured, use:

```bash
ansible-playbook playbooks/install-nginx.yml --ask-vault-pass
```

## Expected Result

Nginx is installed, running, and enabled at boot on both managed nodes. Running the playbook again leaves an already installed and running service in its desired state.
