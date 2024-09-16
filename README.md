# K3s Ansible Setup for Raspberry Pi Cluster

This Ansible project automates the setup of a k3s Kubernetes cluster on a Raspberry Pi 4 cluster. It handles OS updates, installs necessary packages, and deploys k3s using the [official k3s setup script](https://github.com/k3s-io/k3s/) on master and worker nodes.

### Prerequisites

Before setting up a static IP or installing K3s on your Raspberry Pi, ensure the following files are adjusted:
1. Adjust cmdline.txt
Modify `/boot/cmdline.txt` to improve networking stability. Add `cgroup_memory=1` and `cgroup_enable=memory` at the end of the line to enable cgroup memory management, which is required for K3s.
`sudo nano /boot/cmdline.txt`
exemple line: `console=serial0,115200 console=tty1 root=PARTUUID=xxxx-xxxx-xxxx cgroup_memory=1 cgroup_enable=memory`
Save and close the file.

2. Adjust config.txt
Modify `/boot/config.txt` to enable 64-bit support, which is often required for K3s to run optimally. Add the following line under the [all] section: `sudo nano /boot/config.txt` Add under [all]:
```
[all]
arm_64bit=1
```
This enables the 64-bit mode on your Raspberry Pi, allowing K3s to take full advantage of the system's capabilities.

3. Reboot the Raspberry Pi
After making changes to cmdline.txt and config.txt, reboot the Raspberry Pi to apply them. `sudo reboot`

### Setting a Static IP Address on Raspberry Pi

1. **Find Network Interface Name**
Run the command below to list network interfaces and identify the one you want to set as static (usually the "ethernet" type).
`sudo nmcli -p connection show`

2. **Set Static IP, Gateway, and DNS**
Use the commands below to set the IP address, gateway, and DNS for your chosen network interface (replace "Wired connection 1" with your interface name if different).

```
sudo nmcli c mod "Wired connection 1" ipv4.addresses 10.0.90.100/24 ipv4.method manual
sudo nmcli con mod "Wired connection 1" ipv4.gateway 10.0.90.1
sudo nmcli con mod "Wired connection 1" ipv4.dns 10.0.90.1
```
To use multiple DNS servers:
`sudo nmcli con mod "Wired connection 1" ipv4.dns "8.8.8.8,8.8.4.4"`
To assign multiple IP addresses:
`sudo nmcli c mod "Wired connection 1" ipv4.addresses "10.0.90.100/24, 10.0.0.221/24, 10.0.0.222/24" ipv4.method manual`

3. **Restart Network Connection**
Apply the new settings by restarting the network connection.
`sudo nmcli c down "Wired connection 1" && sudo nmcli c up "Wired connection 1"`
__Note:__ If using SSH, this may disconnect your session if the IP changes.
4. **View Network Configuration**
To view the configuration settings for your network connection, use:
`nmcli -p connection show "Wired connection 1"`

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
1. **Install Ansible**
Follow the instructions [here](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) to install Ansible on your system.

2. **Update Variables:**
No hardcoded tokens are used in the playbook; it automatically retrieves the K3s token after installation on the master node.
3. **Run the Playbook:**
Execute the following command to run the playbook, entering the root password when prompted:
`ansible-playbook -i inventory/hosts.ini playbooks/site.yml --ask-become-pass` and provide the root password when asked
`BECOME password: ******`
4. **Copy Kube Config File**
After the playbook completes, the kube config file will be copied locally. You can then copy or merge it into the `~/.kube/config`path on your local computer, allowing you to connect to the cluster remotely.

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
