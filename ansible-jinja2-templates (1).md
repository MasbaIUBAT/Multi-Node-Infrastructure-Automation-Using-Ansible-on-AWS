# Ansible Jinja2 Templates

## Project: CloudForge — Multi-Node Infrastructure Automation Using Ansible on AWS

## Overview

This guide extends the existing Nginx playbook with a Jinja2 template. One HTML template creates a customized web page for each Amazon Linux 2023 managed node.

Ansible replaces `{{ inventory_hostname }}` with the node's name from the inventory. Node 1 displays `node1`, and Node 2 displays `node2`.

The results below are expected results. Complete the steps and verify both nodes before marking this phase as completed.

## 1. Objective

- Understand how Jinja2 variables work inside a template.
- Deploy a customized Nginx web page to both managed nodes.
- Maintain one template instead of editing a separate page on each server.

## 2. Existing Setup

Use the existing Ubuntu Ansible controller, inventory, SSH authentication, and privilege escalation configuration.

| Inventory name | Operating system | Private IP |
|---|---|---|
| node1 | Amazon Linux 2023 | 172.31.19.5 |
| node2 | Amazon Linux 2023 | 172.31.30.81 |

Both nodes must belong to the existing `nodes` inventory group. Use your current IP addresses if the instances have changed.

## 3. Files Used

| File | Purpose |
|---|---|
| `templates/index.html.j2` | Shared HTML template |
| `playbooks/install-nginx.yml` | Existing Nginx playbook with one template task added |

On the Ansible controller, run:

```bash
cd ~/ansible-project
mkdir -p templates playbooks
```

## 4. Create the Jinja2 Template

Create `templates/index.html.j2`:

```bash
nano templates/index.html.j2
```

Add:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>CloudForge</title>
</head>
<body>
    <h1>Welcome to CloudForge</h1>
    <p>This server is {{ inventory_hostname }}</p>
    <p>Deployed using Ansible and Jinja2.</p>
</body>
</html>
```

`{{ inventory_hostname }}` is an Ansible variable. Its value is the inventory name, such as `node1` or `node2`; it does not have to match the operating system hostname. The `.j2` extension identifies the source file as a Jinja2 template.

## 5. Update the Nginx Playbook

Edit the existing `playbooks/install-nginx.yml`:

```bash
nano playbooks/install-nginx.yml
```

Use this complete playbook. It keeps the existing installation and service tasks and adds one task to deploy the page:

```yaml
---
- name: Install Nginx and deploy a customized web page
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

    - name: Deploy the Jinja2 web page
      template:
        src: ../templates/index.html.j2
        dest: /usr/share/nginx/html/index.html
        owner: root
        group: root
        mode: '0644'
```

This uses the default Nginx document root for this Amazon Linux lab. If your Nginx configuration uses a different document root, change `dest` to that location's `index.html`.

## 6. Understand the Template Task

| Setting | Meaning |
|---|---|
| `template` | Processes the template and copies the generated file to each node |
| `src` | Template on the controller; `../templates/` points from the playbook directory to the project's template directory |
| `dest` | Generated HTML file on each managed node |
| `owner` and `group` | Sets file ownership to `root` |
| `mode: '0644'` | Allows the owner to edit and other users to read the file |

The template is processed separately for each node. Updating a static HTML page does not require restarting Nginx.

## 7. Run the Playbook

From the existing project directory on the controller:

```bash
cd ~/ansible-project
ansible-playbook playbooks/install-nginx.yml
```

This uses the inventory configured in your existing `ansible.cfg`.

If your existing `group_vars/nodes/vault.yml` is encrypted and you have not configured another Vault password source, run:

```bash
ansible-playbook playbooks/install-nginx.yml --ask-vault-pass
```

## 8. Verify Both Web Pages

From the controller, run:

```bash
curl http://172.31.19.5
curl http://172.31.30.81
```

The nodes must allow HTTP traffic on port 80 from the controller's security group or private IP. If a request times out, check the network access and Nginx service.

Expected text within the HTML response:

| Node | Expected text |
|---|---|
| node1 | `This server is node1` |
| node2 | `This server is node2` |

Run the playbook again without editing the template. If the nodes already match the requested state, these tasks should report `ok` with no changes.

## 9. What This Demonstrates

- One template generates different content for different servers.
- Ansible automatically substitutes each node's inventory name.
- A single playbook installs Nginx, manages its service, and deploys the page.
- Editing the shared template and rerunning the playbook updates both nodes.

## 10. Completion Checklist

Mark these items after testing:

- [ ] Created `templates/index.html.j2`.
- [ ] Added the template task to the existing Nginx playbook.
- [ ] Successfully ran the playbook against both nodes.
- [ ] Verified that each web page displays the correct node name.
- [ ] Reran the unchanged playbook and confirmed no changes were needed.

After verification, describe the achievement as:

> Used Jinja2 templates with Ansible to deploy customized Nginx web pages across two AWS EC2 nodes.

For GitHub, upload this Markdown guide as documentation. Also add the actual `templates/index.html.j2` file and updated `playbooks/install-nginx.yml` so the implementation is available alongside the explanation.
