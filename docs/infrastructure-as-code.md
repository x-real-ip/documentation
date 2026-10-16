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

4. Clone the infrastructure repository and go into the `ansible` directory
   ``` bash
    git clone https://github.com/x-real-ip/infrastructure.git
    cd infrastructure/ansible
   ```

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

Run the playbooks from the `ansible` directory of the [infrastructure](https://github.com/x-real-ip/infrastructure) repository.

| Playbook | Command | Comment |
| ------------------ | ----------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| desktop | `ANSIBLE_CONFIG=./ansible.cfg ansible-playbook -K --ask-vault-password playbooks/desktop.yaml` | Set the Debian desktop desired state |
| k3s_apply-apps-with-truenas-storage  | `ansible-playbook --ask-vault-password playbooks/k3s_apply-apps-with-truenas-storage.yaml`  | Apply all k8s resources that has storage=truenas label |
| k3s_install_cluster_bare | `ansible-playbook --ask-vault-password playbooks/k3s_install_cluster_bare.yaml` | Install or update k3s on all nodes without installing additional deployments |
| k3s_install_cluster_minimal | `ansible-playbook --ask-vault-password playbooks/k3s_install_cluster_minimal.yaml` | Install or update k3s on all nodes including additional deployments |
| k3s_remove-apps-with-truenas-storage | `ansible-playbook --ask-vault-password playbooks/k3s_remove-apps-with-truenas-storage.yaml` | Delete all k8s resources that has storage=truenas label |
| k3s_render_yaml | `ansible-playbook --ask-vault-password playbooks/k3s_render_yaml.yaml` | Ouput all kubernetes k8s yamls from jinja2 |
| k3s_rolling-update-nodes | `ansible-playbook --ask-vault-password playbooks/k3s_rolling-update-nodes.yaml` | Update the os packages on all k3s nodes |
| k3s_start_all_pods | `ansible-playbook playbooks/k3s_start_all_pods.yaml` | Uncordon nodes |
| k3s_stop_all_pods | `ansible-playbook playbooks/k3s_stop_all_pods.yaml` | Cordon and drain nodes |
| known_hosts | `ansible-playbook playbooks/known_hosts.yaml` | Add devices to the local known_hosts file |
| local_self-signed-certificate_generate | `ansible-playbook playbooks/local_self-signed-certificate_generate.yaml` | Generate a self-signed certificate |
| non-root-user | `ansible-playbook -k --ask-vault-password playbooks/non-root-user.yaml` | Add a non root user |
| proxmox_migrate-vms-to-master | `ansible-playbook playbooks/proxmox_migrate-vms-to-master.yaml` | Migrate all possible VM's to the Proxmox node that is marked as master |
| proxmox_wake-up-standby-node | `ansible-playbook playbooks/proxmox_wake-up-standby-node.yaml` | Send a magic packet to wakeup the standby (not master) Proxmox node |
| shelly_update-firmware | `ansible-playbook --ask-vault-password playbooks/shelly_update-firmware.yaml` | Toggle allow internet firewall rule for NOT VLAN, update and set desired state of all Shelly devices |
| truenas_shares | `ansible-playbook playbooks/truenas_shares.yaml` | Configure all NFS and ISCSI shares on the truenas hosts |
| truenas_switch-master | `ansible-playbook --ask-vault-password playbooks/truenas_switch-master.yaml` | Switch the master from A to B or the otherway around |
| truenas_snapshot-tasks | `ansible-playbook --ask-vault-password playbooks/truenas_snapshot-tasks.yaml` | Apply desired snapshots tasks to the truenas server |

## OpenTofu

The configuration of the infrastructure itself lives in the `opentofu`
directory of the [infrastructure](https://github.com/x-real-ip/infrastructure)
repository. Every directory under `opentofu/stacks` has its own state and is
planned and applied on its own.

| Stack | Manages |
| ----- | ------- |
| `proxmox` | Proxmox cluster, see [Hypervisor](hypervisor.md#install-proxmox) |

### Installation

The [desktop playbook](#playbooks) installs OpenTofu from its APT repository
and adds the stamx CA to the trust store.

### Secrets

Both are stored in Bitwarden and only ever passed as environment variables.

| Environment variable | Contents |
| -------------------- | -------- |
| `TF_VAR_state_passphrase` | Passphrase that encrypts the state in git, at least 16 characters. Without it the state cannot be read. |
| `PROXMOX_VE_API_TOKEN` | `tofu@pve!provider=<value>`, see [Create the OpenTofu API token](hypervisor.md#4-create-the-opentofu-api-token). |

### Usage

1.  Go to the stack and export the secrets. Use single quotes, because the
    token contains a `!`. A leading space keeps the line out of the shell
    history.

    ```sh
    cd opentofu/stacks/proxmox
     export TF_VAR_state_passphrase='...'
     export PROXMOX_VE_API_TOKEN='tofu@pve!provider=...'
    ```

2.  Review the plan and apply it.

    ```sh
    tofu init
    tofu plan
    tofu apply
    ```

3.  Commit the updated `terraform.tfstate`. It is encrypted.

### Adopt existing resources

To bring something that already exists under OpenTofu, add a temporary
`imports.tf` with an `import` block for it. The plan must report only imports
and no changes; if it shows a change, align the code with reality first.
Remove `imports.tf` after the apply.
