# cldemo-evpn-distributed

This demo shows the configuration for EVPN distributed routing. 

*server01/server03* are in *VLAN 13* while *server02/server04* are in *VLAN 24*. The corresponding VXLAN ID's are 13 (device *vni13*) and 24(device *vni24*) respectively. The IP addresses of the servers encode this information. So, server01's IP address is 10.1.3.101 while server04's IP address is 10.2.4.104. Both exit01/exit02 are the default gateways for both VLANs. They're configured with anycast gateway addresses and MACs for each VLAN as described in the book. The packets between leaf* and exit* go with a VXLAN header. The spine switches are the underlay. exit01/exit02 assume the use of VRFs. The EVPN traffic is in vrf *evpn-vrf* while the external world is reachable via *internet-vrf*. *edge01* acts as the firewall merging the two VRFs together. The firewall device is not configured with any firewall rules. Its merely intended to demonstrate traffic flow.

Unlike the centralized routing model, no MLAG is configured between the exit leaves. Since each leaf is also the first hop router for the VLANs, there is no need to synchronize the neighbor tables across the exit leaves.

There is no symmetric and asymmetric modes shown in this demo. The main reason is that once you introduce external routing (RT-5) - for example, for the default route - FRR version 3.2 switches all routes to symmetric as the RT-5 configuration requires the use of an L3 VNI. FRR assumes that if an L3 VNI is configured, you're in symmetric routing mode. In FRR releases >= 4.0, there is an option to configure non-summarized routes to be advertised as RT-2 routes without an L3 VNI even in the presence of an L3 VNI, but this is not described in the book.

Two mistakes in the configuration shown in chapter 6 from exit leaves configuration has been fixed in this repository. The first mistake is that the specification of this L3 VNI in frr.conf. The specific lines that are missing are from exit01's configuration on page 73 are:

   vrf evpn-vrf
      vni 104001
   !

The second mistake is not announcing the internal server subnets to the internet router so that pings to the external world work. The following lines need to be added to the BGP instance associated with evpn-vrf under the AFI/SAFI ipv4 unicast:
   network 10.1.3.0/24
   network 10.2.4.0/24

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

