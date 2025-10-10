## 3-2-1 Backup Strategy

3 Copies, 2 media volumes (truenas-a and truenas-b), 1 off-site (truenas-c)

### off-site

Off-site backup is a replication task from the truenas-master to truenas-c, it is connected via VPN using wireguard.

Configure wireguard connection after boot on truenas-c

1. Create a `wg0.conf` in `/data/wg0.conf` only this directory is persistent.
2. Add the following content (see bitwarden for full config including keys) 
   ``` conf
    [Interface]
    PrivateKey = **removed**
    Address = 10.44.44.3/32
    DNS = 1.1.1.1

    [Peer]
    PublicKey = **removed**
    Endpoint = **removed**:51820
    AllowedIPs = 10.44.44.3/32,10.0.10.0/24,10.0.100.0/24,10.42.0.0/16
    PersistentKeepalive = 25
   ```

3. Go to `/ui/system/initshutdown` and configure the init command as follow:
      - Type: Command
      - Description: wireguard up at boot
      - When: Post Init
      - Command/Script: `/usr/bin/wg-quick up /data/wg0.conf`
      - Enabled: Yes
