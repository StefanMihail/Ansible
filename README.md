# K3s Ansible Setup for Raspberry Pi Cluster

This Ansible project automates the setup of a k3s Kubernetes cluster on a Raspberry Pi 4 cluster. It handles OS updates, installs necessary packages, and deploys k3s using the [official k3s setup script](https://github.com/k3s-io/k3s/) on master and worker nodes.

### Inventory

Define your hosts in `inventory/hosts.ini`:

```ini
[master]
master-node ansible_host=10.0.90.100

[workers]
worker-node-01 ansible_host=10.0.90.101
worker-node-02 ansible_host=10.0.90.102
worker-node-03 ansible_host=10.0.90.103
worker-node-04 ansible_host=10.0.90.104
worker-node-05 ansible_host=10.0.90.105

[k3s_cluster:children]
master
workers
```

### Configuration

`ansible.cfg`

Ensure ansible.cfg points to your inventory and has correct SSH settings.

```
[defaults]
inventory = inventory/hosts.ini
remote_user = pi
private_key_file = ~/.ssh/id_rsa
host_key_checking = False
```

### Usage

1. Install Ansible [from details navigate here](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
2. Update Variables:
	No hardcoded tokens are used; the playbook dynamically retrieves the k3s token post-installation on the master node.
3. Run the Playbook:
	`ansible-playbook playbooks/site.yml`

### Roles
**common:** Updates the OS and installs vim on all nodes.
**k3s_master:** Installs k3s on the master node and retrieves the node token.
**k3s_worker:** Installs k3s agents on worker nodes using the retrieved token.

### Security
The k3s token is stored securely on the master node and fetched by worker nodes during setup.
Ensure SSH keys are managed securely and the `k3s_token` file is protected.

### Troubleshooting
**SSH Issues:** Verify SSH access from your macOS machine to all Raspberry Pi nodes.
**Token Retrieval:** Ensure the k3s_master role successfully retrieves and saves the k3s token.

