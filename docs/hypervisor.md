## Install Proxmox

These steps (re)install one node or the whole cluster. Everything the Proxmox
API can configure is managed by OpenTofu in the
[infrastructure](https://github.com/x-real-ip/infrastructure/tree/main/opentofu)
repository. The steps below are the manual work before and after `tofu apply`.

### 1. Installer

Use the graphical installer and enter the following values.

=== "pve-a"

    | Setting                 | Value                                   |
    | ----------------------- | --------------------------------------- |
    | Target disk             | `Force MP500` (112 GB NVMe)             |
    | Filesystem              | `ext4`                                  |
    | Advanced LVM options    | `swapsize` = `8`, `maxvz` = `0`         |
    | Hostname (FQDN)         | `pve-a.lan.stamx.nl`                    |
    | Management interface    | Onboard NIC                             |
    | Pin interface names     | Enabled (`nic0`, `nic1`, `nic2`)        |
    | IP address (CIDR)       | `10.0.99.2/24`                          |
    | Gateway                 | `10.0.99.1`                             |
    | DNS server              | `10.0.99.1`                             |

=== "pve-b"

    | Setting                 | Value                                   |
    | ----------------------- | --------------------------------------- |
    | Target disk             | `KINGSTON SUV400S37240G` (240 GB SATA)  |
    | Filesystem              | `ext4`                                  |
    | Advanced LVM options    | `swapsize` = `8`, `maxvz` = `0`         |
    | Hostname (FQDN)         | `pve-b.lan.stamx.nl`                    |
    | Management interface    | Onboard NIC                             |
    | Pin interface names     | Enabled (`nic0`, `nic1`, `nic2`)        |
    | IP address (CIDR)       | `10.0.99.3/24`                          |
    | Gateway                 | `10.0.99.1`                             |
    | DNS server              | `10.0.99.1`                             |

!!! warning

    Only select the boot disk. The other NVMe holds the `vm-storage` ZFS pool
    with the VM disks and must not be touched by the installer.

!!! note

    `maxvz = 0` skips the data volume, so there is no `local-lvm` storage to
    remove afterwards. VM disks live on `vm-storage`.

!!! note

    The OpenTofu stack expects the interface names `nic0` (onboard), `nic1` and
    `nic2` (PCI card). If the installer does not offer interface pinning, run
    `pve-network-interface-pinning generate` after the installation and reboot.

### 2. Prepare each node

1.  Add the Ansible SSH key. Run this from the desktop.

    ```sh
    ssh-copy-id -i ~/.ssh/ansible.pub root@10.0.99.2
    ```

2.  Extend the root logical volume over the free space. The installer caps root
    at 96 GiB and leaves some space unallocated.

    ```sh
    lvextend -r -l +100%FREE pve/root
    ```

3.  Import the existing ZFS pool. Never create it, it holds the VM disks.

    ```sh
    zpool import -f vm-storage
    ```

### 3. Create the cluster

1.  Create the cluster on `pve-a`.

    ```sh
    pvecm create pve-cluster
    ```

2.  Join `pve-b`. This asks for the root password of `pve-a`.

    ```sh
    pvecm add 10.0.99.2
    ```

!!! note "Reinstalling a single node"

    The remaining node still knows the reinstalled node. Remove it there before
    joining, for example for `pve-b`:

    ``` sh
    pvecm delnode pve-b
    rm -r /etc/pve/nodes/pve-b
    ```

### 4. Create the OpenTofu API token

Run this once on any cluster node, for example over SSH from the desktop, and
store `tofu@pve!provider=<value>` in Bitwarden. The value is shown only once.

```sh
ssh root@10.0.99.2 'pveum user add tofu@pve --comment "OpenTofu" \
  && pveum aclmod / -user tofu@pve -role Administrator \
  && pveum user token add tofu@pve provider --privsep 0'
```

!!! note

    Users, ACLs and tokens live in `/etc/pve`, which is replicated to every
    node in the cluster, so running this once is enough. Do it after creating
    the cluster: a joining node replaces its `/etc/pve` with the one from the
    cluster.

### 5. Apply OpenTofu

Run this from the `opentofu/stacks/proxmox` directory of the infrastructure
repository.

```sh
export TF_VAR_state_passphrase='...'
export PROXMOX_VE_API_TOKEN='tofu@pve!provider=...'

tofu init
tofu plan
tofu apply
```

This configures the APT repositories (enterprise off, no-subscription on), DNS,
the time zone, the bridges `vmbr1` (wan) and `vmbr2` (lan), the `vm-storage`
storage and the cluster options. The installer only creates `vmbr0`.

The state in git still knows the resources from before the reinstall, so
OpenTofu only creates what is missing.

### 6. Finish each node

1.  Update the system. This needs the no-subscription repository from the
    previous step.

    ```sh
    apt update && apt dist-upgrade -y
    ```

2.  Enable Wake-on-LAN on the onboard NIC. Add the `up` line to the `nic0`
    stanza in `/etc/network/interfaces`.

    ```txt { .no-copy }
    auto nic0
    iface nic0 inet manual
        up /sbin/ethtool -s nic0 wol g
    ```

3.  Run the
    ['Proxmox VE CPU Scaling Governor'](https://community-scripts.github.io/ProxmoxVE/scripts?id=scaling-governor)
    script and set it to `powersave`.

4.  [Add the QDevice](#add-qdevice).

5.  Set up two-factor authentication for `root` via `Datacenter` ->
    `Permissions` -> `Two Factor`.

## High Availability

### Add QDevice

#### On the Qdevice

1.  This command sets the root password, enables root login in the SSH
    configuration, and restarts the SSH service.

    ```bash
    passwd root && sed -i 's/#PermitRootLogin/PermitRootLogin yes/' /etc/ssh/sshd_config && systemctl restart sshd
    ```

2.  Install corosync packages

    ```bash
    apt install corosync-qnetd corosync-qdevice
    ```

#### On the Proxmox node(s)

1.  Install corosync-qdevice packages. Repeat for each node in your cluster.

    ```bash
    apt update && apt install corosync-qdevice -y
    ```

2.  Check current status

    ```bash
    pvecm status
    ```

    the output should look like this

    ```txt { .no-copy }
    Cluster information
    -------------------
    Name:             pve-cluster
    Config Version:   2
    Transport:        knet
    Secure auth:      on

    Quorum information
    ------------------
    Date:             Wed Mar 26 13:41:51 2025
    Quorum provider:  corosync_votequorum
    Nodes:            2
    Node ID:          0x00000002
    Ring ID:          1.16
    Quorate:          Yes

    Votequorum information
    ----------------------
    Expected votes:   2
    Highest expected: 2
    Total votes:      2
    Quorum:           2
    Flags:            Quorate

    Membership information
    ----------------------
        Nodeid      Votes Name
    0x00000001          1 10.0.99.2
    0x00000002          1 10.0.99.3 (local)
    ```

3.  Add the QDevice to the cluster. Run this on one of the Proxmox nodes. Change
    the IP address to the IP of the Qdevice

    ``` sh
    pvecm qdevice setup 10.0.99.102 -f
    ```

4.  Once this is completed check if the Qdevice has been added to the cluster.

    ```sh
    pvecm status
    ```

    The output should now look similar like this:

    ```txt { .no-copy }
    Cluster information
    -------------------
    Name:             pve-cluster
    Config Version:   3
    Transport:        knet
    Secure auth:      on

    Quorum information
    ------------------
    Date:             Wed Mar 26 13:51:33 2025
    Quorum provider:  corosync_votequorum
    Nodes:            2
    Node ID:          0x00000002
    Ring ID:          1.16
    Quorate:          Yes

    Votequorum information
    ----------------------
    Expected votes:   3
    Highest expected: 3
    Total votes:      3
    Quorum:           2
    Flags:            Quorate Qdevice

    Membership information
    ----------------------
        Nodeid      Votes    Qdevice Name
    0x00000001          1    A,V,NMW 10.0.99.2
    0x00000002          1    A,V,NMW 10.0.99.3 (local)
    0x00000000          1            Qdevice
    ```

### Make a VM HA

1. Turn on Replication for the VM that you want to make High Available.
2. Go to `Datacenter` -> `HA` and add a Resource.

### Mount USB stick (backups)

1.  Find the USB stick device
    ```bash
    lsblk
    ```

2.  Create mount directory
    ```bash
    mkdir -p /mnt/pve/backup_usb-stick
    ```

3.  Mount (change `<sd?>` to the correct device)
    ```bash
    mount /dev/<sd?> /mnt/pve/backup_usb-stick
    ```
