IPAM
----

IP Address Schema for Cumulus Networks Demos

| Device | Loopback (lo) IP | Eth0 (Management IP) | AS for BGP |
|-----|-----|-----|-----|
| Leaf01 | 10.0.0.11/32   | 192.168.0.11 | 65011 |
| Leaf02 | 10.0.0.12/32   | 192.168.0.12 | 65012 |
| Leaf03 | 10.0.0.13/32   | 192.168.0.13 | 65013 |
| Leaf04 | 10.0.0.14/32   | 192.168.0.14 | 65014 |
| Spine01 | 10.0.0.21/32   |  192.168.0.21 | 65020 |
| Spine02 | 10.0.0.22/32   |  192.168.0.22 | 65020 |
| Exit01 | 10.0.0.41/32   |  192.168.0.41 | 65041 | 
| Exit02 | 10.0.0.42/32   |  192.168.0.42 | 65042 |
| server01 | 10.0.0.31/32   |  192.168.0.31  |  65031 |
| server02 | 10.0.0.32/32   |  192.168.0.32  |  65032 |
| server03 | 10.0.0.33/32   |  192.168.0.33  |  65033 |
| server04 | 10.0.0.34/32   |  192.168.0.34  |  65034 |
| edge01 | 10.0.0.51/32   |  192.168.0.51  | 65051 |
| oob-mgmt-switch | UNASSIGNED   |  192.168.0.1 (bridge)  | UNASSIGNED |
| internet | 10.0.0.253/32 | 192.168.1.253 | 65253 |

MAC Address Management
----------------------

MAC Address Schema for Cumulus Networks Demos

| Device | Interface | MAC Address |
|-----|-----|-----|
| Server01 | eth0 | A0:00:00:00:00:31 |
| Server02 | eth0 | A0:00:00:00:00:32 |
| Server03 | eth0 | A0:00:00:00:00:33 |
| Server04 | eth0 | A0:00:00:00:00:34 |
| Leaf01 | eth0 | A0:00:00:00:00:11 |
| Leaf02 | eth0 | A0:00:00:00:00:12 |
| Leaf03 | eth0 | A0:00:00:00:00:13 |
| Leaf04 | eth0 | A0:00:00:00:00:14 |
| Spine01 | eth0 | A0:00:00:00:00:21 |
| Spine02 | eth0 | A0:00:00:00:00:22 |
| Exit01 | eth0 | A0:00:00:00:00:41 |
| Exit02 | eth0 | A0:00:00:00:00:42 |
| Edge01 | eth0 | A0:00:00:00:00:51 |
| internet | eth0 | A0:00:00:00:00:f3 |
| oob-mgmt-switch | swp1 | A0:00:00:00:00:61 |
