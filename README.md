# Site-To-Site-VPN
Lab Configuration Cisco IOS VPN IPSEC site-to-site, pre-shared, with NAT overload between private networks

#Introduction
In this lab we will do a configuration lab in Cisco IOS of a site-to-site, pre-shared IPSEC VPN topology with NAT overload between two private networks. It demonstrates a configuration using crypto-map.

#Prerequisites 
- EIGRP
- NAT

#Giving Ip Addresses
For the first step you can go ahead and put all the ip addresses you need on every device (static addressing)

#Green network 
This Green network will be simulate the internet we will start by configuring EIGRP, so enter the follwing commands 
## FAI - R3 
```
router eigrp 90
network 10.1.1.0 0.0.0.3
network 10.1.1.4 0.0.0.3
```

## FAI - R1
```
router eigrp 90
passive-interface GigabitEthernet0/0/0
network 200.1.1.0 0.0.0.3
network 10.1.1.0 0.0.0.3
```

## FAI - R2
```
router eigrp 90
passive-interface GigabitEthernet0/0/0
network 10.1.1.4 0.0.0.3
network 200.1.2.0 0.0.0.3
```

