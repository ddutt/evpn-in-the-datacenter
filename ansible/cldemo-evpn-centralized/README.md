# cldemo-evpn-centralized

This demo shows the configuration for EVPN centralized routing. The exit leaves exit01/exit02 function as the EVPN routers while the leaves (leaf01-leaf04) function as EVPN bridges only. ARP/ND Suppression is enabled.

*server01/server03* are in *VLAN 13* while *server02/server04* are in *VLAN 24*. The corresponding VXLAN ID's are 13 (device *vni13*) and 24(device *vni24*)  respectively The IP addresses of the servers encode this information. So, server01's IP address is 10.1.3.101 while server04's IP address is 10.2.4.104. Both exit01/exit02 are the default gateways for both VLANs. They're configured with anycast gateway addresses and MACs for each VLAN as described in the book. The packets between leaf* and exit* go with a VXLAN header. The spine switches are the underlay. exit01/exit02 assume the use of VRFs. The EVPN traffic is in vrf *evpn-vrf* while the external world is reachable via *internet-vrf*. *edge01* acts as the firewall merging the two VRFs together. The firewall device is not configured with any firewall rules. Its merely intended to demonstrate traffic flow.

Unlike the description in the book however, MLAG is configured between the exit leaves. This is not required for routing to work. However, it is required if you wish to ping the individual exit leaves from a server. This is so because the exit leaves share the same gateway MAC and IP, and to laod balance traffic across both nodes without the need for an additional protocol such as VRRP, the exit leaves share the same VTEP IP. But this means that replies to ICMP pings originated from one exit leaf may end up on the other exit leaf. This happens because the return traffic may be load balanced to the other exit leaf due to their sharing a common VTEP IP. In other words, the individual MAC and IP addresses of the exit leaves are like singly-attached hosts but they share a common VTEP IP. MLAG resolves this by using the peer link to reach the correct exit leaf.

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




