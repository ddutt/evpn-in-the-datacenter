# cldemo-evpn-centralized

This demo shows how centralized routing can be configured and used with EVPN. The exit leaves exit01/exit02 function as the EVPN routers while the leaves (leaf01-leaf04) function as EVPN bridges only.

*server01/server03* are in *VLAN 13* while *server02/server04* are in *VLAN 24*. The corresponding VXLAN ID's are 13 (device *vni13*) and 24(device *vni24*)  respectively The IP addresses of the servers encode this information. So, server01's IP address is 10.1.3.101 while server04's IP address is 10.2.4.104. Both exit01/exit02 are the default gateways for both VLANs. They're configured with anycast gateway addresses and MACs for each VLAN as described in the book. The packets between leaf* and exit* go with a VXLAN header. The spine switches are the underlay. exit01/exit02 assume the use of VRFs. The EVPN traffic is in vrf *evpn-vrf* while the external world is reachable via *internet-vrf*. *edge01* acts as the firewall merging the two VRFs together. The firewall device is not configured with any firewall rules. Its merely intended to demonstrate traffic flow.

Unlike the description in the book however, MLAG is configured between the exit leaves. This is required to avoid ARP replies from ending up at the wrong exit leaf. For example, if exit01 ARPs for a host whose ARP reply ends up on exit02, exit02 ignores this ARP reply as it doesn't have any state that it originated an ARP request for the host's IP. FRR as of version 4.0.1 does not install ARP entries if the device is an SVI for that subnet as well. By running MLAG between the exit leaves, this problem is overcome. Only the VXLAN interfaces are associated with this MLAG.

Single session EBGP is used to configure the underlay and the overlay. BGP Unnumbered is assumed throughout except with the firewall where the interfaces are numbered, though the BGP configuration itself uses interface names. 

The Network Topology is depicted below.


## Topology ##
![EVPN Symmetric Model Demo](https://github.com/CumulusNetworks/cldemo-evpn-symmetric/blob/master/evpn_symmetric_demo.png)


Quickstart: Run the demo
------------------------

    ansible-playbook reset.yml
    ansible-playbook run-demo.yml
    ssh server01
    ping 10.1.3.103
	ping 10.2.4.204
    

Viewing the Results
-------
