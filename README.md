Ansible Playbook: Setup MicroK8s on Raspberry Pi 5
==================================================

![RPi5](assets/rp5-board-1.png)

This Ansible playbook automates the setup of MicroK8s on a Raspberry Pi 5 running Ubuntu 23.04 LTS. It includes tasks to update the system, configure networking, disable swap, install MicroK8s, join worker nodes, enable essential MicroK8s addons, enable MetalLB for load balancing, enable Kata Containers, and configure GitHub Container Registry (GHCR) private registries.

## Disclaimer
Disclaimer: My `ansible-fu` is weak 🙈 so this project is intentionally simple in structure. Expert users are welcome to contribute improvements via pull requests (PRs).


# Usage

1. Clone this repository to your local machine:

    ```bash
    git clone https://github.com/josephbarnett/ansible-rpi5-microk8s-cluster.git
    ```

2. Navigate to the cloned repository directory:

    ```bash
    cd ansible-rpi5-microk8s-cluster
    ```

3. Update the inventory file inventory.ini with the appropriate IP addresses of your master and worker nodes, and specify the necessary variables.

4. Run the Ansible playbook using the following command:

    ```bash
    ansible-playbook -i inventory.ini playbook.yml
    ```

5. **[MANUAL]** Run `microk8s add-node` on the master node.
   
   You will expect output as follows, you need the line ending with `--worker`
    ```bash
    From the node you wish to join to this cluster, run the following:
    microk8s join 172.18.1.56:25000/fobarfoobarfobarfoobarfobarfoobarfobarfoobar

    Use the '--worker' flag to join a node as a worker not running the control plane, eg:
    microk8s join 172.18.1.56:25000/fobarfoobarfobarfoobarfobarfoobarfobarfoobar --worker

    If the node you are adding is not reachable through the default interface you can use one of the following:
    microk8s join 172.18.1.56:25000/fobarfoobarfobarfoobarfobarfoobarfobarfoobar
    microk8s join 172.18.1.57:25000/fobarfoobarfobarfoobarfobarfoobarfobarfoobar
    ```
6. **[MANUAL]** On each worker node, run that command - and wait for the cluster to complete initialization
    ```bash
    microk8s join 172.18.1.56:25000/fobarfoobarfobarfoobarfobarfoobarfobarfoobar
    Contacting cluster at 172.18.1.56

    The node has joined the cluster and will appear in the nodes list in a few seconds.

    This worker node gets automatically configured with the API server endpoints.
    If the API servers are behind a loadbalancer please set the '--refresh-interval' to '0s' in:
        /var/snap/microk8s/current/args/apiserver-proxy
    and replace the API server endpoints with the one provided by the loadbalancer in:
        /var/snap/microk8s/current/args/traefik/provider.yaml
    ```

    **On the master node, you can validate success using the following**:
    ```bash
    microk8s kubectl get nodes -o wide
    NAME    STATUS   ROLES    AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION     CONTAINER-RUNTIME
    pi4p0   Ready    <none>   33m     v1.28.7   172.18.1.56   <none>        Ubuntu 23.10   6.5.0-1005-raspi   containerd://1.6.28
    pi5p0   Ready    <none>   4m28s   v1.28.7   172.18.1.63   <none>        Ubuntu 23.10   6.5.0-1005-raspi   containerd://1.6.28
    ```

## Inventory Template

Below is a template for the inventory file (inventory.ini):

```ini
[master]
master_host ansible_host=<master_ip_address>

[master:vars]
ansible_user=<master_ssh_user>
metallb_ip_range_start=<metallb_ip_range_start>
metallb_ip_range_end=<metallb_ip_range_end>
ghcr_username=<github_username>
ghcr_pat=<github_personal_access_token>

[worker]
worker1 ansible_host=<worker_ip_address>
worker2 ansible_host=<worker_ip_address>

[worker:vars]
ansible_user=<worker_ssh_user>
```

> Replace `<master_ip_address>`, `<worker_ip_address>`, `<master_ssh_user>`, `<worker_ssh_user>`, `<metallb_ip_range_start>`, `<metallb_ip_range_end>`, `<github_username>`, and `<github_personal_access_token>` with the appropriate values for your setup.

## Playbook Structure
The playbook playbook.yml consists of the following tasks:

1. Update APT repositories and packages.
1. Enable Ethernet interface.
1. Disable swap.
1. Install MicroK8s.
1. Join worker nodes to the cluster.
1. Enable standard MicroK8s addons.
1. Enable MetalLB MicroK8s addon.
1. Enable Kata MicroK8s addon.
1. Configure GitHub Container Registry (GHCR) private registries.

## Handlers
There are handlers defined in the playbook to apply network configuration changes and Metallb config:

1. Apply netplan configuration.
1. Apply MetalLB config.

## Requirements
* Ansible installed on the local machine.
* Raspberry Pi 4 or 5 devices running Ubuntu 23.04 LTS.
* SSH access to the master and worker nodes with sudo privileges.

## License
This playbook is licensed under the [APACHE](LICENSE) License. Feel free to modify and distribute it according to your needs.

