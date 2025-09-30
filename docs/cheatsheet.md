## TrueNAS

### Rename volume

```
zfs rename r01_1tb/k8s/{current zvol name} r01_1tb/k8s/{new zvol name}
```

### Resize VM disk

```
sudo dnf install cloud-utils-growpart
```

```
sudo growpart /dev/sda 2
```

```
sudo pvresize /dev/sda2
```

```
sudo lvextend -l +100%FREE /dev/mapper/rl-root
```

```
sudo xfs_growfs /
```

## Repair iSCSI share

### Script

1. Run the following script to start a interactive script that repair the iSCSI
   share

   ```bash
   curl -sSL https://raw.githubusercontent.com/x-real-ip/infrastructure/refs/heads/main/scripts/repair_iscsi_volume.sh | bash

   ```

### Manually

1. Make sure that the container that uses the volume has stopped.
2. SSH into one of the nodes in the cluster and start discovery

    ```bash
    sudo iscsiadm -m discovery -t st -p truenas-master.lan.stamx.nl && \
    read -p "Enter the disk name: " DISKNAME && \
    export DISKNAME
    ```

3. Login to target

    ```bash
    sudo iscsiadm --mode node --targetname iqn.2005-10.org.freenas.ctl:${DISKNAME} --portal truenas-master.lan.stamx.nl --login && \
    sleep 5 && \
    lsblk && \
    read -p "Enter the device ('sda' for example): " DEVICENAME && \
    export DEVICENAME
    ```

4. Create a local mount point & mount to replay logfile

    ```bash
    sudo mkdir -vp /mnt/data-0 && sudo mount /dev/${DEVICENAME} /mnt/data-0/
    ```

5. Unmount the device

    ```bash
    sudo umount /mnt/data-0/
    ```

6. Run check / ncheck

    ```bash
    sudo xfs_repair -n /dev/${DEVICENAME}; sudo xfs_ncheck /dev/${DEVICENAME}
    echo "If filesystem corruption was corrected due to replay of the logfile, the xfs_ncheck should produce a list of nodes and pathnames, instead of the errorlog."
    ```

7. If needed run xfs repair

    ```bash
    sudo xfs_repair /dev/${DEVICENAME}
    ```

8. Logout from target

    ```bash
    sudo iscsiadm --mode node --targetname iqn.2005-10.org.freenas.ctl:${DISKNAME} --portal storage-server-lagg.lan.stamx.nl --logout
    echo "Volumes are now ready to be mounted as PVCs."
    ```

## Rsync

Run a Rsync exact copy

```bash
sudo rsync -axHAWXS --numeric-ids --info=progress2 /mnt/sourcePart/ /mnt/destPart
```

## Bitnami Sealed Secret

Raw mode

```
echo -n foo | kubeseal --cert "./sealed-secret-tls-2.crt" --raw --scope cluster-wide
```

Create TLS (unencrypted) secret

```
kubectl create secret tls cloudflare-tls --key origin-ca.pk --cert origin-ca.crt --dry-run=client -o yaml > cloudflare-tls.yaml
```

Encrypt secret with custom public certificate.

```console
kubeseal --cert "./sealed-secret-tls-2.crt" --format=yaml < secret.yaml > sealed-secret.yaml
```

Add sealed secret to configfile secret

```console
echo -n <mypassword_value> | kubectl create secret generic <secretname> --dry-run=client --from-file=<password_key>=/dev/stdin -o json | kubeseal --cert ./sealed-secret-tls-2.crt -o yaml \
-n democratic-csi --merge-into <secret>.yaml
```

Raw sealed secret

`strict` scope (default):

```console
echo -n foo | kubeseal --raw --from-file=/dev/stdin --namespace bar --name mysecret
AgBChHUWLMx...
```

`namespace-wide` scope:

```console
echo -n foo | kubeseal --cert ./sealed-secret-tls-2.crt --raw --from-file=/dev/stdin --namespace bar --scope namespace-wide
AgAbbFNkM54...
```

`cluster-wide` scope:

```console
echo -n foo | kubeseal --cert ./sealed-secret-tls-2.crt --raw --from-file=/dev/stdin --scope cluster-wide
AgAjLKpIYV+...
```

Include the `sealedsecrets.bitnami.com/namespace-wide` annotation in the `SealedSecret`

```yaml
metadata:
  annotations:
    sealedsecrets.bitnami.com/namespace-wide: "true"
```

Include the `sealedsecrets.bitnami.com/cluster-wide` annotation in the `SealedSecret`

```yaml
metadata:
  annotations:
    sealedsecrets.bitnami.com/cluster-wide: "true"
```

[Github](https://github.com/bitnami-labs/sealed-secrets)

[AWS Bitnami tutorial](https://aws.amazon.com/blogs/opensource/managing-secrets-deployment-in-kubernetes-using-sealed-secrets/)

[Blogpost Tutorial](https://itsmetommy.com/2020/06/26/kubernetes-sealed-secrets/)

## Kubernetes

Drain and terminate all pods gracefully on the node while marking the node as unschedulable

```console
kubectl drain --ignore-daemonsets --delete-emptydir-data <nodename>
```

Make the node unschedulable

```console
kubectl cordon <nodename>
```

Make the node schedulable

```console
kubectl uncordon <nodename>
```

Convert to BASE64

```console
echo -n '<value>' | base64
```

Decode a secret with config file data

```console
kubectl get secret <secret_name> -o jsonpath='{.data}' -n <namespace>
```

Create secret from file

```console
kubectl create secret generic <secret name> --from-file=<secret filelocation> --dry-run=client  --output=yaml > secrets.yaml
```

Restart Pod

```console
kubectl rollout restart deployment <deployment name> -n <namespace>
```

Change PV reclaim policy

```console
kubectl patch pv <pv-name> -p "{\"spec\":{\"persistentVolumeReclaimPolicy\":\"Retain\"}}"
```

Shell into pod

```console
kubectl exec -it <pod_name> -- /bin/bash
```

Copy to or from pod

```console
kubectl cp <namespace>/<pod>:/tmp/foo /tmp/bar
```
