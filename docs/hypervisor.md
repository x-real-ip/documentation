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

### 5. Create the node certificates

The web interface and API of every node use a certificate from the stamx CA.
Skip this step when `~/certs/proxmox` on the desktop already holds valid
certificates.

!!! note "Run this on the desktop, not on a node"

    The CA private key must never leave the desktop. OpenTofu reads the
    certificates from `~/certs/proxmox` and uploads them to the nodes in the
    next step.

1.  Create the directory and go into it.

    ```sh
    mkdir -p ~/certs/proxmox && cd ~/certs/proxmox
    ```

2.  Copy the CA certificate from the ansible repository.

    ```sh
    cp ~/git-repositories/infrastructure/ansible/roles/ca_trust/files/stamx-ca-certificate.crt ca.crt
    ```

3.  Create a private key and a certificate request for every node.

    === "pve-a"

        ```sh
        openssl req -new -newkey rsa:4096 -nodes -keyout pve-a.key -out pve-a.csr \
          -subj "/O=stamx/CN=pve-a.lan.stamx.nl" \
          -addext "subjectAltName=DNS:pve-a.lan.stamx.nl,DNS:pve-a,IP:10.0.99.2" \
          -addext "basicConstraints=CA:FALSE" \
          -addext "keyUsage=critical,digitalSignature,keyEncipherment" \
          -addext "extendedKeyUsage=serverAuth"
        ```

    === "pve-b"

        ```sh
        openssl req -new -newkey rsa:4096 -nodes -keyout pve-b.key -out pve-b.csr \
          -subj "/O=stamx/CN=pve-b.lan.stamx.nl" \
          -addext "subjectAltName=DNS:pve-b.lan.stamx.nl,DNS:pve-b,IP:10.0.99.3" \
          -addext "basicConstraints=CA:FALSE" \
          -addext "keyUsage=critical,digitalSignature,keyEncipherment" \
          -addext "extendedKeyUsage=serverAuth"
        ```

4.  Sign the requests with the CA. Replace `/path/to/stamx-ca-private.key`
    with the location of the CA private key.

    === "pve-a"

        ```sh
        openssl x509 -req -in pve-a.csr -CA ca.crt -CAkey /path/to/stamx-ca-private.key \
          -CAcreateserial -copy_extensions copyall -days 825 -sha256 -out pve-a.crt
        ```

    === "pve-b"

        ```sh
        openssl x509 -req -in pve-b.csr -CA ca.crt -CAkey /path/to/stamx-ca-private.key \
          -CAcreateserial -copy_extensions copyall -days 825 -sha256 -out pve-b.crt
        ```

5.  Check the certificates. Both must report `OK`.

    ```sh
    openssl verify -CAfile ca.crt pve-a.crt pve-b.crt
    ```

6.  Remove the requests and protect the private keys.

    ```sh
    rm pve-a.csr pve-b.csr && chmod 600 pve-a.key pve-b.key
    ```

!!! note

    The certificates are valid for 825 days. To renew them, repeat this step
    and run `tofu apply`.

### 6. Apply OpenTofu

Apply the `proxmox` stack, see [OpenTofu usage](infrastructure-as-code.md#usage).

This configures the APT repositories (enterprise off, no-subscription on), DNS,
`/etc/hosts`, the time zone, the bridges `vmbr1` (wan) and `vmbr2` (lan), the
SDN VLANs, the `vm-storage` storage, the `lsi-hba` PCI mapping, the pools, the
node certificates and the cluster options. The installer only creates `vmbr0`.

The state in git still knows the resources from before the reinstall, so
OpenTofu only creates what is missing.

!!! warning "Reinstalling the whole cluster"

    HA, storage replication and pool membership refer to guests. After a
    reinstall of both nodes those guests do not exist yet, so leave them out
    of the first apply:

    ```sh
    tofu apply \
      -exclude=proxmox_haresource.opnsense \
      -exclude=proxmox_replication.guest \
      -exclude=proxmox_pool_membership.guest
    ```

    Run a plain `tofu apply` once the guests are back. Reinstalling a single
    node does not need this, because the guests keep running on the other
    node.

### 7. Finish each node

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

HA and storage replication are managed by OpenTofu in
`opentofu/stacks/proxmox/ha.tf`. Changes made in the web interface are not in
the code and get lost on a reinstall.

1.  Add the guest to `replicated_guests` in `locals.tf`, so its disks are
    replicated to `pve-b` every 15 minutes.
2.  Add a `proxmox_haresource` for the guest to `ha.tf`, and add it to the
    `resources` of a HA rule if it should prefer a node.
3.  [Apply OpenTofu](infrastructure-as-code.md#usage).

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
