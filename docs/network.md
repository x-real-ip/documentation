## Switch Ports and VLAN Assignments

| Device                  | Patch Port | Switch Port | VLAN Untagged | VLAN Tagged                  | PVID |
|--------------------------|------------|-------------|---------------|------------------------------|------|
| WAN NTU                 | 4          | 1           |               | 300                          | 300  |
| wallpatch-basement      | 10         | 2           | 99            |                              | 99   |
| wallpatch-livingroom    | 8          | 3           | 99            | 10, 20, 30, 40               | 99   |
| tado                    | -          | 4           | 30            |                              | 30   |
| k3s-wkr-01              | 16         | 5           | 100           |                              | 100  |
| ap-outdoor              | 6          | 6           | 99            | 10, 20, 30, 40               | 99   |
| ap-upstairs             | 7          | 7           | 99            | 10, 20, 30, 40               | 99   |
| ap-downstairs           | 18         | 8           | 99            | 10, 20, 30, 40               | 99   |
| shield                  | 9          | 9           | 30            |                              | 30   |
|                          |            | 10          | 100           |                              | 100  |
| k3s-wkr-02              | 17         | 11          | 99            |                              | 99   |
| tv-01                   | 12         | 12          | 30            |                              | 30   |
| wallpatch-livingroom    | 13         | 13          | 100           |                              | 100  |
| inverter                | 14         | 14          | 30            |                              | 30   |
| pve-a - WAN-A           | -          | 15          |               | 300                          | 300  |
| pve-b - WAN-B           | -          | 16          |               | 300                          | 300  |
| pfsense pve-a - LAN-A   | -          | 17          | 99            | 10, 20, 30, 40, 99, 100      | 99   |
| pfsense pve-b - LAN-B   | -          | 18          | 99            | 10, 20, 30, 40, 99, 100      | 99   |
| pve-a - management (C)  | -          | 19          | 99            |                              | 99   |
| pve-b - management (D)  | -          | 20          | 99            |                              | 99   |
|                          | 19         | 21          | 99            |                              | 99   |
|                          |            | 22          | 99            |                              | 99   |
|                          |            | 23          | 100           |                              | 100  |
|                          |            | 24          | 100           |                              | 100  |
|                          |            | 25          | 99            |                              | 99   |
|                          |            | 26          | 99            |                              | 99   |
|                          |            | 27          | 99            |                              | 99   |
|                          |            | 28          | 99            |                              | 99   |

---

## VLANs

| VLAN | Description  |
|------|--------------|
| 300  | wan          |
| 10   | private      |
| 20   | guest        |
| 30   | iot          |
| 40   | not          |
| 99   | management   |
| 100  | systems      |

---

## Management

| Device           | VLAN | MAC               | IP         |
|------------------|------|-------------------|------------|
| firewall / router| 99   |                   | 10.0.99.1  |
| pve-a            | 99   |                   | 10.0.99.2  |
| pve-b            | 99   |                   | 10.0.99.3  |
| switch           | 99   |                   | 10.0.99.4  |
| ap-upstairs      | 99   | b0:4e:26:12:35:fa | 10.0.99.31 |
| ap-downstairs    | 99   | b0:4e:26:86:08:aa | 10.0.99.32 |
| ap-outdoor       | 99   | 68:ff:7b:0a:40:1a | 10.0.99.33 |
