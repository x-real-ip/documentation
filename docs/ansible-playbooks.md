# Install Ansible

Remove old Debian package

```bash
sudo apt remove ansible -y
```

# Ansible playbooks

| Playbook                             | Command                                                                                   | Comment                                                       |
| ------------------------------------ | ----------------------------------------------------------------------------------------- | ------------------------------------------- |
| non-root-user                        | ansible-playbook -k --ask-vault-password playbooks/non-root-user.yaml                     | Add a non root user |
| desktop                              | ansible-playbook -K --ask-vault-password playbooks/desktop.yaml                           | Set the Debian desktop desired state |
| truenas-shares                       | ansible-playbook playbooks/truenas_shares.yaml                                            | Configure all NFS and ISCSI shares on the truenas hosts |
| truenas_switch-master                | ansible-playbook --ask-vault-password playbooks/truenas_switch-master.yaml                | Switch the master from A to B or the otherway around |
| shelly_update-firmware               | ansible-playbook playbooks/shelly_update-firmware.yaml                                    | Update and set desired state of all Shelly devices |
| k3s_rolling-update-nodes             | ansible-playbook --ask-vault-password playbooks/k3s_rolling-update-nodes.yaml             | Update the os packages on all k3s nodes |
| k3s_install_cluster_minimal          | ansible-playbook --ask-vault-password playbooks/k3s_install_cluster_bare.yaml             | Install or update k3s on all nodes without installing additional deployments |
| k3s_install_cluster_minimal          | ansible-playbook --ask-vault-password playbooks/k3s_install_cluster_minimal.yaml          | Install or update k3s on all nodes including additional deployments |
| k3s_remove-apps-with-truenas-storage | ansible-playbook --ask-vault-password playbooks/k3s_remove-apps-with-truenas-storage.yaml | Delete all k8s resources that has storage=truenas label |
| k3s_stop_all_pods                    | ansible-playbook playbooks/k3s_stop_all_pods.yaml                                         | Cordon and drain nodes |
