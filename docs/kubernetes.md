## Prerequisites

- Ansible installed on your local machine. See [Infrastructure as Code](infrastructure-as-code.md#installation) page.
- SSH access to the target machines where you want to install k3s.

## Add SSH keys on local device

1. Copy private and public key (stored in Bitwarden Vault)

    ```bash
    cp /path/to/my/key/ansible ~/.ssh/ansible
    cp /path/to/my/key/ansible.pub ~/.ssh/ansible.pub
    ```

2. Change permissions on the files

    ```bash
    sudo chmod 600 ~/.ssh/ansible
    sudo chmod 600 ~/.ssh/ansible.pub
    ```

3. Make ssh agent to actually use copied key

    ```bash
    ssh-add ~/.ssh/ansible
    ```

## Setting up VM hosts on Proxmox

1. Create a VM with the following partitions
      - / (10 GB) xfs
      - /var (50GB) xfs
      - /boot (1GB)

!!! note

    No swap partition is needed for kubernetes

2. Login to the VM and set hostname.
   ```console
   hostnamectl set-hostname <hostname>
   ```
3. Reboot.
4. Set static ip for the VM in the DHCP server.

5. Reboot.
6. Set this hostname in the ansible inventory `hosts.yaml` file.