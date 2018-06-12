# cldemo-evpn-centralized

This demo shows the configuration for EVPN centralized routing. The exit leaves exit01/exit02 function as the EVPN routers while the leaves (leaf01-leaf04) function as EVPN bridges only. ARP/ND Suppression is enabled.

*server01/server03* are in *VLAN 13* while *server02/server04* are in *VLAN 24*. The corresponding VXLAN ID's are 13 (device *vni13*) and 24(device *vni24*)  respectively The IP addresses of the servers encode this information. So, server01's IP address is 10.1.3.101 while server04's IP address is 10.2.4.104. Both exit01/exit02 are the default gateways for both VLANs. They're configured with anycast gateway addresses and MACs for each VLAN as described in the book. The packets between leaf* and exit* go with a VXLAN header. The spine switches are the underlay. exit01/exit02 assume the use of VRFs. The EVPN traffic is in vrf *evpn-vrf* while the external world is reachable via *internet-vrf*. *edge01* acts as the firewall merging the two VRFs together. The firewall device is not configured with any firewall rules. Its merely intended to demonstrate traffic flow.

Users might notice that it is not possible to ping the individual exit leaves from a server. This is so because the exit leaves share the same gateway MAC and IP, and to laod balance traffic across both nodes without the need for an additional protocol such as VRRP, the exit leaves share the same VTEP IP. But this means that replies to ICMP pings originated from one exit leaf may end up on the other exit leaf. This happens because the return traffic may be load balanced to the other exit leaf due to their sharing a common VTEP IP. In other words, the individual MAC and IP addresses of the exit leaves are like singly-attached hosts but they share a common VTEP IP. MLAG resolves this by using the peer link to reach the correct exit leaf. So, if you wish to ping the individual exit leaves, configure MLAG between them. Routing works correctly to both the external world and across VLANs 13/24 without MLAG.

Single session EBGP is used to configure the underlay and the overlay. BGP Unnumbered is assumed throughout except with the firewall where the interfaces are numbered, though the BGP configuration itself uses interface names. 


## Quickstart: Run the demo

`
    ansible-playbook reset.yml
    ansible-playbook run-demo.yml
    ssh server01
	ping 10.1.3.103
	ping 10.2.4.104
	ping 10.0.0.253
	ping www.google.com
`

The pings are ordered to verify that:
* EVPN bridging works across racks
* EVPN RT-2 based routing works
* EVPN RT-5 routing works to reach the internet node
* EVPN RT-5 routing works to reach the Internet

## Viewing the Results

Lets look at a few queries to explore the network. 

### Listing local VNI information

    sudo vtysh -c 'show evpn vni'
    leaf01# sh evpn vni 
    VNI        Type VxLAN IF              # MACs   # ARPs   # Remote VTEPs  Tenant VRF
    24         L2   vni24                 9        11       2               Default-IP-Routing-Table
    13         L2   vni13                 9        11       2               Default-IP-Routing-Table
    104001     L3   None                  0        0        n/a             n/a
    
### Listing the VNI from BGP's Perspective ###

The information on a regular VTEP:

    cumulus@leaf01:mgmt-vrf:~$ sudo vtysh -c 'sh bgp l2vpn evpn vni' 
    Advertise Gateway Macip: Disabled
    Advertise All VNI flag: Enabled
    Number of L2 VNIs: 2
    Number of L3 VNIs: 0
    Flags: * - Kernel
      VNI        Type RD                    Import RT                 Export RT                 Tenant VRF
    * 24         L2   10.0.0.11:3           65011:24                  65011:24                 Default-IP-Routing-Table
    * 13         L2   10.0.0.11:2           65011:13                  65011:13                 Default-IP-Routing-Table
    cumulus@leaf01:mgmt-vrf:~$ 
  

The same information on an exit leaf:

    cumulus@exit01:mgmt-vrf:~$ sudo vtysh -c 'show bgp l2vpn evpn vni'
    Advertise Gateway Macip: Enabled
    Advertise All VNI flag: Enabled
    Number of L2 VNIs: 2
    Number of L3 VNIs: 0
    Flags: * - Kernel
      VNI        Type RD                    Import RT                 Export RT                 Tenant VRF
    * 24         L2   10.0.0.41:3           65041:24                  65041:24                 evpn-vrf
    * 13         L2   10.0.0.41:2           65041:13                  65041:13                 evpn-vrf
    
  The fact the tenant VRFs are different on the two is not an issue since the VRF is only relevant on a router.
  
### Listing the MACs known to EVPN

    cumulus@leaf01:mgmt-vrf:~$ sudo vtysh -c 'show evpn mac vni all'
  
    VNI 24 #MACs (local and remote) 9
    
    MAC               Type   Intf/Remote VTEP      VLAN 
    00:03:00:22:22:01 local  bond02                24   
    02:03:00:44:44:02 remote 10.0.0.134           
    44:38:39:00:00:0c remote 10.0.0.135           
    02:03:00:44:44:01 remote 10.0.0.134           
    02:03:00:22:22:02 local  bond02                24   
    02:03:00:22:22:01 local  bond02                24   
    44:39:39:ff:00:24 remote 10.0.0.135           
    44:38:39:00:00:03 local  vlan24                24   
    44:38:39:00:00:4b remote 10.0.0.135           
    00:03:00:44:44:01 remote 10.0.0.134           
    
    VNI 13 #MACs (local and remote) 9
    
    MAC               Type   Intf/Remote VTEP      VLAN 
    00:03:00:33:33:02 remote 10.0.0.134           
    44:38:39:00:00:0c remote 10.0.0.135           
    02:03:00:33:33:01 remote 10.0.0.134           
    44:39:39:ff:00:13 remote 10.0.0.135           
    00:03:00:11:11:02 local  bond01                13   
    44:38:39:00:00:03 local  vlan13                13   
    02:03:00:33:33:02 remote 10.0.0.134           
    44:38:39:00:00:4b remote 10.0.0.135           
    02:03:00:11:11:02 local  bond01                13   
    02:03:00:11:11:01 local  bond01                13   
    cumulus@leaf01:mgmt-vrf:~$
    
### Listing the ARP cache known to EVPN

    cumulus@leaf01:mgmt-vrf:~$ sudo vtysh -c 'show evpn arp-cache vni all'

    VNI 24 #ARP (IPv4 and IPv6, local and remote) 11
    
    IP                      Type   MAC               Remote VTEP          
    fe80::4638:39ff:fe00:3  local  44:38:39:00:00:03
    10.2.4.104              remote 00:03:00:44:44:01 10.0.0.134           
    10.2.4.102              local  00:03:00:22:22:01
    10.2.4.1                remote 44:39:39:ff:00:24 10.0.0.135           
    fe80::203:ff:fe44:4401  remote 00:03:00:44:44:01 10.0.0.134           
    fe80::4638:39ff:fe00:4b remote 44:38:39:00:00:4b 10.0.0.135           
    10.2.4.12               remote 44:38:39:00:00:0c 10.0.0.135           
    fe80::4639:39ff:feff:24 remote 44:39:39:ff:00:24 10.0.0.135           
    fe80::4638:39ff:fe00:c  remote 44:38:39:00:00:0c 10.0.0.135           
    fe80::203:ff:fe22:2201  local  00:03:00:22:22:01
    10.2.4.11               remote 44:38:39:00:00:4b 10.0.0.135           
    
    VNI 13 #ARP (IPv4 and IPv6, local and remote) 11
    
    IP                      Type   MAC               Remote VTEP          
    fe80::4638:39ff:fe00:3  local  44:38:39:00:00:03
    10.1.3.101              local  00:03:00:11:11:02
    10.1.3.103              remote 00:03:00:33:33:02 10.0.0.134           
    fe80::203:ff:fe33:3302  remote 00:03:00:33:33:02 10.0.0.134           
    10.1.3.1                remote 44:39:39:ff:00:13 10.0.0.135           
    10.1.3.11               remote 44:38:39:00:00:4b 10.0.0.135           
    fe80::203:ff:fe11:1102  local  00:03:00:11:11:02
    fe80::4638:39ff:fe00:4b remote 44:38:39:00:00:4b 10.0.0.135           
    fe80::4639:39ff:feff:13 remote 44:39:39:ff:00:13 10.0.0.135           
    10.1.3.12               remote 44:38:39:00:00:0c 10.0.0.135           
    fe80::4638:39ff:fe00:c  remote 44:38:39:00:00:0c 10.0.0.135           
    cumulus@leaf01:mgmt-vrf:~$ 
    
### Listing the BGP details of a remote MAC entry

Similar to "show bgp ipv4 unicast route <prefix>" or "show ip bgp <prefix>", you can use "show bgp l2vpn evpn route vni <vni> mac <mac>" like so:

    cumulus@leaf01:mgmt-vrf:~$ sudo vtysh -c 'show bgp l2vpn evpn route vni 13 mac 00:03:00:33:33:02' 
    BGP routing table entry for [2]:[0]:[0]:[48]:[00:03:00:33:33:02]
    Paths: (3 available, best #3)
      Not advertised to any peer
      Route [2]:[0]:[0]:[48]:[00:03:00:33:33:02] VNI 13
      Imported from 10.0.0.14:2:[2]:[0]:[0]:[48]:[00:03:00:33:33:02]
      65020 65014
        10.0.0.134 from spine02(swp52) (10.0.0.22)
          Origin IGP, localpref 100, valid, external
          Extended Community: RT:65014:13 ET:8
          AddPath ID: RX 0, TX 174
          Last update: Mon Jun 11 18:43:22 2018
    
      Route [2]:[0]:[0]:[48]:[00:03:00:33:33:02] VNI 13
      Imported from 10.0.0.13:2:[2]:[0]:[0]:[48]:[00:03:00:33:33:02]
      65020 65013
        10.0.0.134 from spine02(swp52) (10.0.0.22)
          Origin IGP, localpref 100, valid, external
          Extended Community: RT:65013:13 ET:8
          AddPath ID: RX 0, TX 170
          Last update: Mon Jun 11 18:43:22 2018
    
      Route [2]:[0]:[0]:[48]:[00:03:00:33:33:02] VNI 13
      Imported from 10.0.0.13:2:[2]:[0]:[0]:[48]:[00:03:00:33:33:02]
      65020 65013
        10.0.0.134 from spine01(swp51) (10.0.0.21)
          Origin IGP, localpref 100, valid, external, bestpath-from-AS 65020, best
          Extended Community: RT:65013:13 ET:8
          AddPath ID: RX 0, TX 166
          Last update: Mon Jun 11 18:43:22 2018
    
    
    Displayed 3 paths for requested prefix

You can see that the prefixes have an extended community tag is an  RT of value *65014:1* or *65014:13*. ET is the encapsulation type extended community as defined in [RFC8365](https://tools.ietf.org/html/rfc8365) and its value of 8 indicates that we're using VXLAN encapsulation type as per section 12 of that RFC.

Similarly, you can see that this mac entry is associated with the default router MAC extended community. First find the MAC and IP address associated with the default gateway for a VNI. You can do this by either looking at the default route's nexthop on a server and then consulting that nexthop's MAC address in the ARP/ND cache like so:

    cumulus@server03:~$ ip ro show
    default via 10.1.3.1 dev uplink 
    10.1.3.0/24 dev uplink  proto kernel  scope link  src 10.1.3.103 
    192.168.0.0/24 dev eth0  proto kernel  scope link  src 192.168.0.33 
    cumulus@server03:~$ ip n show 10.1.3.1
    10.1.3.1 dev uplink lladdr 44:39:39:ff:00:13 REACHABLE
    cumulus@server03:~$ 

Then, you can look at the MAC/IP entry (FRR requires you to specify both a MAC and its associated IP address if the entry has been advertised with both) on the leaf associated with that VTEP like this:

    cumulus@leaf03:mgmt-vrf:~$ sudo vtysh -c 'show bgp l2vpn evpn route vni 13 mac 44:39:39:ff:00:13 ip 10.1.3.1'
    BGP routing table entry for [2]:[0]:[0]:[48]:[44:39:39:ff:00:13]:[32]:[10.1.3.1]
    Paths: (4 available, best #4)
      Not advertised to any peer
      Route [2]:[0]:[0]:[48]:[44:39:39:ff:00:13]:[32]:[10.1.3.1] VNI 13
      Imported from 10.0.0.42:2:[2]:[0]:[0]:[48]:[44:39:39:ff:00:13]:[32]:[10.1.3.1]
      65020 65042
        10.0.0.135 from spine02(swp52) (10.0.0.22)
          Origin IGP, localpref 100, valid, external
          Extended Community: RT:65042:13 ET:8 Default Gateway
          AddPath ID: RX 0, TX 78
          Last update: Mon Jun 11 18:42:48 2018
    
      Route [2]:[0]:[0]:[48]:[44:39:39:ff:00:13]:[32]:[10.1.3.1] VNI 13
      Imported from 10.0.0.42:2:[2]:[0]:[0]:[48]:[44:39:39:ff:00:13]:[32]:[10.1.3.1]
      65020 65042
        10.0.0.135 from spine01(swp51) (10.0.0.21)
          Origin IGP, localpref 100, valid, external
          Extended Community: RT:65042:13 ET:8 Default Gateway
          AddPath ID: RX 0, TX 77
          Last update: Mon Jun 11 18:42:48 2018
    
      Route [2]:[0]:[0]:[48]:[44:39:39:ff:00:13]:[32]:[10.1.3.1] VNI 13
      Imported from 10.0.0.41:2:[2]:[0]:[0]:[48]:[44:39:39:ff:00:13]:[32]:[10.1.3.1]
      65020 65041
        10.0.0.135 from spine02(swp52) (10.0.0.22)
          Origin IGP, localpref 100, valid, external
          Extended Community: RT:65041:13 ET:8 Default Gateway
          AddPath ID: RX 0, TX 70
          Last update: Mon Jun 11 18:42:48 2018
    
      Route [2]:[0]:[0]:[48]:[44:39:39:ff:00:13]:[32]:[10.1.3.1] VNI 13
      Imported from 10.0.0.41:2:[2]:[0]:[0]:[48]:[44:39:39:ff:00:13]:[32]:[10.1.3.1]
      65020 65041
        10.0.0.135 from spine01(swp51) (10.0.0.21)
          Origin IGP, localpref 100, valid, external, bestpath-from-AS 65020, best
          Extended Community: RT:65041:13 ET:8 Default Gateway
          AddPath ID: RX 0, TX 69
          Last update: Mon Jun 11 18:42:48 2018
    
    
    Displayed 4 paths for requested prefix
    cumulus@leaf03:mgmt-vrf:~$ 

You can see that the extended community has the "Default Gateway" tag. Unfortunately, there doesn't seem to be a command to just like the MACs with the "Default Gateway" community.

In case you're wondering why there are 4 entries, a careful examination of the fields will yield the answer. *leaf03*(in this case) receives the advertisements from each of the spines (can't be distinguished because both have the same ASN, 65020) and that makes two. Each exit leaf originates an advertisement since each of them is a default gateway, and this makes the total announcements count 4.


### Listing Type-3 Routes

To list the type-3 routes that have been advertised, you can start at a high level with `sudo vtysh -c "show bgp l2vpn evpn route type multicast"` (type-3 routes are called multicast in the EVPN RFC). Or you can look for all the advertisers for a specific VNI with `sudo vtysh -c "show bgp l2vpn evpn route vni 13 type multicast"`, which produces output like this:


    cumulus@leaf01:mgmt-vrf:~$ sudo vtysh -c "show bgp l2vpn evpn route vni 13 type multicast"
    BGP table version is 25, local router ID is 10.0.0.11
    Status codes: s suppressed, d damped, h history, * valid, > best, i - internal
    Origin codes: i - IGP, e - EGP, ? - incomplete
    EVPN type-2 prefix: [2]:[ESI]:[EthTag]:[MAClen]:[MAC]:[IPlen]:[IP]
    EVPN type-3 prefix: [3]:[EthTag]:[IPlen]:[OrigIP]
    EVPN type-5 prefix: [5]:[ESI]:[EthTag]:[IPlen]:[IP]
    
       Network          Next Hop            Metric LocPrf Weight Path
    *> [3]:[0]:[32]:[10.0.0.112]
                    10.0.0.112                         32768 i
    *  [3]:[0]:[32]:[10.0.0.134]
                    10.0.0.134                             0 65020 65014 i
    *  [3]:[0]:[32]:[10.0.0.134]
                    10.0.0.134                             0 65020 65013 i
    *  [3]:[0]:[32]:[10.0.0.134]
                    10.0.0.134                             0 65020 65014 i
    *> [3]:[0]:[32]:[10.0.0.134]
                    10.0.0.134                             0 65020 65013 i
    *  [3]:[0]:[32]:[10.0.0.135]
                    10.0.0.135                             0 65020 65042 i
    *  [3]:[0]:[32]:[10.0.0.135]
                    10.0.0.135                             0 65020 65041 i
    *  [3]:[0]:[32]:[10.0.0.135]
                    10.0.0.135                             0 65020 65042 i
    *> [3]:[0]:[32]:[10.0.0.135]
                    10.0.0.135                             0 65020 65041 i
    
    Displayed 3 prefixes (9 paths) (of requested type)
    cumulus@leaf01:mgmt-vrf:~$ 
    
And then you can dig deeper by asking for the details of a specific VTEP with `sudo vtysh -c "show bgp l2vpn evpn route vni 24 multicast 10.0.0.135"` which produces output like this:

    cumulus@leaf01:mgmt-vrf:~$ sudo vtysh -c "show bgp l2vpn evpn route vni 24 multicast 10.0.0.135"
    BGP routing table entry for [3]:[0]:[32]:[10.0.0.135]
    Paths: (4 available, best #4)
      Not advertised to any peer
      Route [3]:[0]:[32]:[10.0.0.135]
      Imported from 10.0.0.42:3:[3]:[0]:[32]:[10.0.0.135]
      65020 65042
        10.0.0.135 from spine01(swp51) (10.0.0.21)
          Origin IGP, localpref 100, valid, external
          Extended Community: RT:65042:24 ET:8
          AddPath ID: RX 0, TX 127
          Last update: Tue Jun 12 10:42:28 2018
    
      Route [3]:[0]:[32]:[10.0.0.135]
      Imported from 10.0.0.41:3:[3]:[0]:[32]:[10.0.0.135]
      65020 65041
        10.0.0.135 from spine01(swp51) (10.0.0.21)
          Origin IGP, localpref 100, valid, external
          Extended Community: RT:65041:24 ET:8
          AddPath ID: RX 0, TX 107
          Last update: Tue Jun 12 10:42:28 2018
    
      Route [3]:[0]:[32]:[10.0.0.135]
      Imported from 10.0.0.42:3:[3]:[0]:[32]:[10.0.0.135]
      65020 65042
        10.0.0.135 from spine02(swp52) (10.0.0.22)
          Origin IGP, localpref 100, valid, external
          Extended Community: RT:65042:24 ET:8
          AddPath ID: RX 0, TX 56
          Last update: Tue Jun 12 10:42:28 2018
    
      Route [3]:[0]:[32]:[10.0.0.135]
      Imported from 10.0.0.41:3:[3]:[0]:[32]:[10.0.0.135]
      65020 65041
        10.0.0.135 from spine02(swp52) (10.0.0.22)
          Origin IGP, localpref 100, valid, external, bestpath-from-AS 65020, best
          Extended Community: RT:65041:24 ET:8
          AddPath ID: RX 0, TX 55
          Last update: Tue Jun 12 10:42:28 2018
    
    
    Displayed 4 paths for requested prefix

How do we know that this encodes the VNI 13? Because of the RT extended community. We ignore the ASN (65041 or 65042) and the remaining string represents the VNI as per the RFC, and as described in chapter 3 of the book in the section on Route Target.
