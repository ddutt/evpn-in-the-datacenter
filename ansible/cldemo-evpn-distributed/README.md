# cldemo-evpn-distributed

This demo shows the configuration for EVPN distributed routing. 

*server01/server03* are in *VLAN 13* while *server02/server04* are in *VLAN 24*. The corresponding VXLAN ID's are 13 (device *vni13*) and 24(device *vni24*) respectively. The IP addresses of the servers encode this information. So, server01's IP address is 10.1.3.101 while server04's IP address is 10.2.4.104. Both exit01/exit02 are the default gateways for both VLANs. They're configured with anycast gateway addresses and MACs for each VLAN as described in the book. The packets between leaf* and exit* go with a VXLAN header. The spine switches are the underlay. exit01/exit02 assume the use of VRFs. The EVPN traffic is in vrf *evpn-vrf* while the external world is reachable via *internet-vrf*. *edge01* acts as the firewall merging the two VRFs together. The firewall device is not configured with any firewall rules. Its merely intended to demonstrate traffic flow.

Unlike the centralized routing model, no MLAG is configured between the exit leaves as they're no longer the default gateway, and each leaf associated with the server is.

There is no symmetric and asymmetric modes shown in this demo. The main reason is that once you introduce external routing (RT-5) - for example, for the default route - FRR version 3.2 switches all routes to symmetric as the RT-5 configuration requires the use of an L3 VNI. FRR assumes that if an L3 VNI is configured, you're in symmetric routing mode. In FRR releases >= 4.0, there is an option to configure non-summarized routes to be advertised as RT-2 routes without an L3 VNI even in the presence of an L3 VNI, but this is not described in the book.

Single session EBGP is used to configure the underlay and the overlay. BGP Unnumbered is assumed throughout except with the firewall where the interfaces are numbered, though the BGP configuration itself uses interface names. 


Quickstart: Run the demo
------------------------

    ansible-playbook reset.yml
    ansible-playbook run-demo.yml
    ssh server01
	ping 10.1.3.103
	ping 10.2.4.104
	ping 10.0.0.253
	ping www.google.com
    

The pings are ordered to verify that:
* EVPN bridging works across racks
* EVPN RT-2 based routing works
* EVPN RT-5 routing works to reach the internet node
* EVPN RT-5 routing works to reach the Internet

## Viewing the Results

Many of the same commands described in the [centralized demo](../cldemo-evpn-centralized) work here as well, though the outputs may differ. The first thing you'll notice is that each leaf is also the first hop router and owns the gateway address for VNI (VLAN) 13. So, the command to see the gateway MAC advertisement, `show bgp l2vpn evpn route vni 13 mac 44:39:39:ff:00:13 ip 10.1.3.1` will fail as follows:

``` bash
cumulus@leaf01:mgmt-vrf:~$ sudo vtysh -c 'show bgp l2vpn evpn route vni 13 mac 44:39:39:ff:00:13 ip 10.1.3.1'
% Network not in table
cumulus@leaf01:mgmt-vrf:~$ 
```
    
Let's examine the output of the VNI list command:

``` bash
cumulus@leaf01:mgmt-vrf:~$ sudo vtysh -c 'show evpn vni'
VNI        Type VxLAN IF              # MACs   # ARPs   # Remote VTEPs  Tenant VRF
24         L2   vni24                 6        7        1               evpn-vrf
13         L2   vni13                 6        8        1               evpn-vrf
104001     L3   vxlan4001             3        3        n/a             evpn-vrf
cumulus@leaf01:mgmt-vrf:~$ 
```

One thought might be why do I see only a single remote VTEP. I do have the exit leaves and the other pair of leaves(leaf03/leaf04). Well, since there's no real use of asymmetric in this exampple, we just didn't configure VNIs 13 and 24 on the exit leaves, saving it state. With symmetric, you can have hosts talk with hosts in other VNIs even if the first hop router doesn't carry the destination VNI.

Next, we see that there's an L3 VNI declared with the number 104001. This is the VNI used in the VXLAN header between VTEPs as described in the symmetric model of chapter 5 in the book.

The next difference is that the VNIs are shown in the correct VRF, evpn-vrf on leaf01. This is because leaf01 is now an EVPN router too, not just an EVPN bridge.

### Viewing the Routing Table

The routing table on a leaf can be seen via the command "ip route show vrf evpn-vrf" as follows:

``` bash
cumulus@leaf01:mgmt-vrf:~$ ip ro show vrf evpn-vrf
default  proto bgp  metric 20 
        nexthop via 10.0.0.41  dev vlan4001 weight 1 onlink
        nexthop via 10.0.0.42  dev vlan4001 weight 1 onlink
unreachable default  metric 4278198272 
10.0.0.41  proto bgp  metric 20 
        nexthop via 10.0.0.41  dev vlan4001 weight 1 onlink
        nexthop via 10.0.0.42  dev vlan4001 weight 1 onlink
10.0.0.42  proto bgp  metric 20 
        nexthop via 10.0.0.41  dev vlan4001 weight 1 onlink
        nexthop via 10.0.0.42  dev vlan4001 weight 1 onlink
10.0.0.100  proto bgp  metric 20 
        nexthop via 10.0.0.41  dev vlan4001 weight 1 onlink
        nexthop via 10.0.0.42  dev vlan4001 weight 1 onlink
10.0.0.253  proto bgp  metric 20 
        nexthop via 10.0.0.41  dev vlan4001 weight 1 onlink
        nexthop via 10.0.0.42  dev vlan4001 weight 1 onlink
10.1.3.0/24 dev vlan13  proto kernel  scope link  src 10.1.3.11 
10.1.3.0/24 dev vlan13-v0  proto kernel  scope link  src 10.1.3.1 
10.1.3.103 via 10.0.0.134 dev vlan4001  proto bgp  metric 20 onlink 
10.2.4.0/24 dev vlan24  proto kernel  scope link  src 10.2.4.11 
10.2.4.0/24 dev vlan24-v0  proto kernel  scope link  src 10.2.4.1 
10.2.4.104 via 10.0.0.134 dev vlan4001  proto bgp  metric 20 onlink 
cumulus@leaf01:mgmt-vrf:~$ 
```
    
We see that the default route has been learnt via nexthops 10.0.0.41 and 10.0.0.42. Those are the addresses of the exit leaves. And if you examine the ARP/ND cache, you'll see that we have MAC addresses associated with those entries:

``` bash
cumulus@leaf01:mgmt-vrf:~$ ip neigh show 10.0.0.41
10.0.0.41 dev vlan4001 lladdr 06:07:02:85:b7:a2 offload REACHABLE
cumulus@leaf01:mgmt-vrf:~$ ip neigh show 10.0.0.42
10.0.0.42 dev vlan4001 lladdr 44:38:39:00:00:0c offload REACHABLE
cumulus@leaf01:mgmt-vrf:~$ 
```
    
And an examination of those MAC addresses tells us that those entries are reachable via the respective VTEPs:

``` bash
cumulus@leaf01:mgmt-vrf:~$ bridge fdb show | grep 06:07:02:85:b7:a2
06:07:02:85:b7:a2 dev vxlan4001 vlan 4001 offload master bridge 
06:07:02:85:b7:a2 dev vxlan4001 dst 10.0.0.41 self offload 
cumulus@leaf01:mgmt-vrf:~$ bridge fdb show | grep 44:38:39:00:00:0c
44:38:39:00:00:0c dev vxlan4001 vlan 4001 offload master bridge 
44:38:39:00:00:0c dev vxlan4001 dst 10.0.0.42 self offload 
cumulus@leaf01:mgmt-vrf:~$ 
```

And the circular use of IP address 10.0.0.41 and 10.0.0.42 is resolved by looking at the underlay routing table for these addresses:

``` bash
cumulus@leaf01:mgmt-vrf:~$ ip ro show 10.0.0.41
10.0.0.41  proto bgp  metric 20 
        nexthop via 169.254.0.1  dev swp51 weight 1 onlink
        nexthop via 169.254.0.1  dev swp52 weight 1 onlink
cumulus@leaf01:mgmt-vrf:~$ ip ro show 10.0.0.42
10.0.0.42  proto bgp  metric 20 
        nexthop via 169.254.0.1  dev swp51 weight 1 onlink
        nexthop via 169.254.0.1  dev swp52 weight 1 onlink
cumulus@leaf01:mgmt-vrf:~$ 
```

In other words, the default route (or the *internet* node's loopback, 10.0.0.253) is reachable via 10.0.0.41 and 10.0.0.42 which have entries in the ARP/ND cache giving them a MAC address which resolves to a VTEP reachable via the underlay multipathed via the spines. 

## Looking at RT-5 routes

There is no way to unfortunately display just the information for a prefix with type 5. You can get the dump of all the information associated with all routes via `show bgp l2vpn evpn route type prefix`. From here you can determine the specific RD (which doesn't provide the info for other RDs announcing that path as it is in this case) you're interested in and get the detailed info via `show bgp l2vpn evpn route rd 10.0.0.41:2` (for example) and weed out the default route which ends up looking like this:

``` bash
BGP routing table entry for 10.0.0.41:2:[5]:[0]:[0]:[0]:[0.0.0.0]
Paths: (2 available, best #2)
  Advertised to non peer-group peers:
  spine01(swp51) spine02(swp52)
  Route [5]:[0]:[0]:[0]:[0.0.0.0] VNI 104001
  65020 65041 65530 65041 25253
    10.0.0.41 from spine01(swp51) (10.0.0.21)
      Origin IGP, localpref 100, valid, external
      Extended Community: RT:65041:104001 ET:8 Rmac:06:07:02:85:b7:a2
      AddPath ID: RX 0, TX 29
      Last update: Tue Jun 12 09:28:24 2018

  Route [5]:[0]:[0]:[0]:[0.0.0.0] VNI 104001
  65020 65041 65530 65041 25253
    10.0.0.41 from spine02(swp52) (10.0.0.22)
      Origin IGP, localpref 100, valid, external, bestpath-from-AS 65020, best
      Extended Community: RT:65041:104001 ET:8 Rmac:06:07:02:85:b7:a2
      AddPath ID: RX 0, TX 9
      Last update: Tue Jun 12 09:28:23 2018
```

You can see that this route has been announced with an extended community of Router MAC (Rmac), besides the usual RT and ET (encapsulation type of 8 which is VXLAN as per section 12 of [RFC 8365](https://tools.ietf.org/html/rfc8365)). 
