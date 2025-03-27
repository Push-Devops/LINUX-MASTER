# ISSUE : MACHINE GOT THE IP ADDRESS BUT WE CANNOT GO OUTSIDE / CANNOT CONECT THE MACHINE THROUGH SSH

1. check if you are on correct network interface


[root@mylinux log]# ifconfig
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.37  netmask 255.255.255.0  broadcast 192.168.0.255

so here we have only one inerface but suppose we have 3 interface enp0s3 , enp0s4 , enp0s5 and we should assign the ip 1902.168.0.37 to enp0s5 
But we assigned it to enp0s3 , so make sure ip is assigned to correct interface

# how to check if we are assigning it to correct interface
  we got tools like ethtool

    [root@mylinux log]# ethtool enp0s3
  Settings for enp0s3:
          Supported ports: [ TP ]
          Supported link modes:   10baseT/Half 10baseT/Full
                                  100baseT/Half 100baseT/Full
                                  1000baseT/Full
          Supported pause frame use: No
          Supports auto-negotiation: Yes
          Supported FEC modes: Not reported
          Advertised link modes:  10baseT/Half 10baseT/Full
                                  100baseT/Half 100baseT/Full
                                  1000baseT/Full
          Advertised pause frame use: No
          Advertised auto-negotiation: Yes
          Advertised FEC modes: Not reported
          Speed: 1000Mb/s
          Duplex: Full
          Auto-negotiation: on
          Port: Twisted Pair
          PHYAD: 0
          Transceiver: internal
          MDI-X: Unknown
          Supports Wake-on: umbg
          Wake-on: d
          Current message level: 0x00000007 (7)
                                 drv probe link
          Link detected: yes
  [root@mylinux log]#

Link detected: yes --> means cable at back is connected 



  [root@mylinux ~]# mii-tool enp0s3
  enp0s3: no autonegotiation, 1000baseT-FD flow-control, link ok
  [root@mylinux ~]#


3. Check if the subnet mask is correct in ifconfig

4. Check gateway , 


  [root@mylinux log]# netstat -rnv
  Kernel IP routing table
  Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
  0.0.0.0         192.168.0.1     0.0.0.0         UG        0 0          0 enp0s3
  192.168.0.0     0.0.0.0         255.255.255.0   U         0 0          0 enp0s3
  [root@mylinux log]#

Run ping gateway ip and check if its listening 

5. 
