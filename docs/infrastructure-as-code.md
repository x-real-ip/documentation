## Ansible

### Installation

1. Remove old Debian package

    ``` bash
    sudo apt remove ansible -y
    ```
2. Install pipx

    ``` bash
    sudo apt update
    sudo apt install pipx
    pipx ensurepath
    ```

3. Install the minimal `ansible-core` package using pipx

    ``` bash
    pipx install ansible-core
    ```

4. Clone the infrastructure repository
   ``` bash
    git clone https://github.com/x-real-ip/infrastructure.git
   ```

5. Navigate to the `ansible` directory

5. Install collections from the requirements.yaml located in the Ansible dir (optional)

    ``` bash
    ansible-galaxy collection install -r requirements.yaml
    ```

    !!! note "requirements.yaml example"

        ``` yaml { .no-copy }
        collections:
          - name: ansible.posix
          - name: kubernetes.core
          - name: community.general
        ```


### Playbooks

Navigate to the `ansible` directory where the [infrastructure](https://github.com/x-real-ip/infrastructure.git) repository is cloned.

| Playbook                             | Command                                                                                   | Comment                                                                      |
| ------------------------------------ | ----------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| non-root-user                        | `ansible-playbook -k --ask-vault-password playbooks/non-root-user.yaml`                     | Add a non root user                                                          |
| desktop                              | `ansible-playbook -K --ask-vault-password playbooks/desktop.yaml`                           | Set the Debian desktop desired state                                         |
| truenas-shares                       | `ansible-playbook playbooks/truenas_shares.yaml`                                            | Configure all NFS and ISCSI shares on the truenas hosts                      |
| truenas_switch-master                | `ansible-playbook --ask-vault-password playbooks/truenas_switch-master.yaml`                | Switch the master from A to B or the otherway around  |
| truenas_snapshot-tasks               | `ansible-playbook --ask-vault-password playbooks/truenas_snapshot-tasks.yaml`               | Apply desired snapshots tasks to the truenas server |
| shelly_update-firmware               | `ansible-playbook playbooks/shelly_update-firmware.yaml`                                    | Update and set desired state of all Shelly devices |
| k3s_rolling-update-nodes             | `ansible-playbook --ask-vault-password playbooks/k3s_rolling-update-nodes.yaml`             | Update the os packages on all k3s nodes                                      |
| k3s_install_cluster_bare          | `ansible-playbook --ask-vault-password playbooks/k3s_install_cluster_bare.yaml`             | Install or update k3s on all nodes without installing additional deployments |
| k3s_install_cluster_minimal          | `ansible-playbook --ask-vault-password playbooks/k3s_install_cluster_minimal.yaml`          | Install or update k3s on all nodes including additional deployments          |
| k3s_remove-apps-with-truenas-storage | `ansible-playbook --ask-vault-password playbooks/k3s_remove-apps-with-truenas-storage.yaml` | Delete all k8s resources that has storage=truenas label                      |
| k3s_stop_all_pods                    | `ansible-playbook playbooks/k3s_stop_all_pods.yaml`                                         | Cordon and drain nodes                                                       |
