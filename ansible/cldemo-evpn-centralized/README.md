# cldemo-evpn-symmetric

This demo shows one approach of running VXLAN Routing with EVPN.  It uses the distributed architecture with symmetric IRB model. (Note Cumulus supports asymmetric model as well).  Using this technique employs the VXLAN Routing directly on the ToR, using EVPN for both VLAN/VXLAN bridging as well as VXLAN and external routing.  

In this demo, each server is configured on a VLAN, with a total of two VLANs for the setup.  MLAG is also set up between servers and the leafs.

Each Leaf is configured with an anycast gateway, and the servers default gateways are pointing towards the corresponding leaf switch IP gateway address.  We create two tenant VNIs (corresponding to two VLANs/VXLANs), which are bridged to corresponding VLANs.

Additionally, we create a L3VNI that provides for the inter-VXLAN routing (using Type 2 routes), as well as to the external "internet" using EVPN Type 5 routes.  All VNIs are located in VRF1. 

We advertise a default route via the BGP ipv4 address family originating from the "internet" router into the exit switch's VRF1.  The exit switches then advertise the default route to the ToRs via a EVPN Type 5 route.  All other inter-VXLAN routing occurs via Type 2 routes learned over the L3VNI.

The Network Topology is depicted below.


## Topology ##
![EVPN Symmetric Model Demo](https://github.com/CumulusNetworks/cldemo-evpn-symmetric/blob/master/evpn_symmetric_demo.png)




Software in Use:
----------------

On Spines, Leafs, Exits and Internet:

 - Cumulus v3.5.0

On Servers:

 - Ubuntu 16.04 
  




Quickstart: Run the demo
------------------------

Before running this demo, install VirtualBox and Vagrant. The currently supported versions of VirtualBox and Vagrant can be found on the [cldemo-vagrant](https://github.com/CumulusNetworks/cldemo-vagrant) page.  

    git clone https://github.com/cumulusnetworks/cldemo-vagrant
    cd cldemo-vagrant
    vagrant up oob-mgmt-server oob-mgmt-switch 
    vagrant up leaf01 leaf02 leaf03 leaf04 spine01 spine02 exit01 exit02 internet server01 server02 server03 server04
    vagrant ssh oob-mgmt-server
    git clone https://github.com/CumulusNetworks/cldemo-evpn-symmetric
    cd cldemo-evpn-symmetric
    ansible-playbook run-demo.yml
    ssh server01
    ping 172.16.1.1 



   
    

Viewing the Results
-------


View the EVPN Routing Table:

   

     cumulus@leaf01:mgmt-vrf:~$ net show bgp evpn route
     BGP table version is 17, local router ID is 10.0.0.11
     Status codes: s suppressed, d damped, h history, * valid, > best, i - internal
     Origin codes: i - IGP, e - EGP, ? - incomplete
     EVPN type-2 prefix: [2]:[ESI]:[EthTag]:[MAClen]:[MAC]:[IPlen]:[IP]
     EVPN type-3 prefix: [3]:[EthTag]:[IPlen]:[OrigIP]
     Network          Next Hop            Metric LocPrf Weight Path
     Route Distinguisher: 10.0.0.11:2
     *> [2]:[0]:[0]:[48]:[44:38:39:00:00:03]
                     10.0.0.112                         32768 i                           
     *> [2]:[0]:[0]:[48]:[44:38:39:00:00:03]:[32]:[10.1.3.101]
                     10.0.0.112                         32768 i
     *> [2]:[0]:[0]:[48]:[44:38:39:00:00:03]:[128]:[fe80::4638:39ff:fe00:3]
                     10.0.0.112                         32768 i
     *> [2]:[0]:[0]:[48]:[46:38:39:00:00:03]
                     10.0.0.112                         32768 i
     *> [2]:[0]:[0]:[48]:[46:38:39:00:00:17]
     *> [3]:[0]:[32]:[10.0.0.112]
                     10.0.0.112                         32768 i
     Route Distinguisher: 10.0.0.11:3
     **-SNIP-**
     Route Distinguisher: 10.0.0.41:2
     *  [5]:[0]:[0]:[0.0.0.0]
                        10.0.0.41                              0 65020 65041 25253 i
     *> [5]:[0]:[0]:[0.0.0.0]
                        10.0.0.41                              0 65020 65041 25253 i
     Route Distinguisher: 10.0.0.42:2
     *  [5]:[0]:[0]:[0.0.0.0]
                        10.0.0.42                              0 65020 65042 25253 i
     *> [5]:[0]:[0]:[0.0.0.0]
                        10.0.0.42                              0 65020 65042 25253 i
                                    
                                   
 





Check the Kernel routing table on the leaf in the VRF:

    cumulus@leaf01:mgmt-vrf:~$ ip route show vrf vrf1
    default  proto bgp  metric 20
    nexthop via 10.0.0.42  dev vlan4001 weight 1 onlink
    nexthop via 10.0.0.41  dev vlan4001 weight 1 onlink
    unreachable default  metric 4278198272
    10.1.3.0/24 dev vlan13  proto kernel  scope link  src 10.1.3.11
    10.1.3.0/24 dev vlan13-v0  proto kernel  scope link  src 10.1.3.1
    10.1.3.103 via 10.0.0.134 dev vlan4001  proto bgp  metric 20 onlink
    10.2.4.0/24 dev vlan24  proto kernel  scope link  src 10.2.4.11
    10.2.4.0/24 dev vlan24-v0  proto kernel  scope link  src 10.2.4.1
    10.2.4.104 via 10.0.0.134 dev vlan4001  proto bgp  metric 20 onlink
   

Ping from Server01 to Server04 on (different VLANs and racks), and ping from server01 to Internet Router

 

               
                               
    cumulus@server01:~$ ping 10.2.4.104
    PING 10.2.4.104 (10.2.4.104) 56(84) bytes of data.
    64 bytes from 10.2.4.104: icmp_seq=1 ttl=62 time=6.56 ms
    64 bytes from 10.2.4.104: icmp_seq=2 ttl=62 time=2.63 ms
    64 bytes from 10.2.4.104: icmp_seq=3 ttl=62 time=2.17 ms
    ^C
    --- 10.2.4.104 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2002ms
    rtt min/avg/max/mdev = 2.178/3.792/6.563/1.969 ms
    PING 172.16.1.1 (172.16.1.1) 56(84) bytes of data.
    64 bytes from 172.16.1.1: icmp_seq=1 ttl=62 time=2.53 ms
    64 bytes from 172.16.1.1: icmp_seq=2 ttl=62 time=2.92 ms
    









    
    



