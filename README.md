# Site-To-Site-VPN
Lab Configuration Cisco IOS VPN IPSEC site-to-site, pre-shared, with NAT overload between private networks

# Introduction
In this lab we will do a configuration lab in Cisco IOS of a site-to-site, pre-shared IPSEC VPN topology with NAT overload between two private networks. It demonstrates a configuration using crypto-map.

# Prerequisites 
- EIGRP
- NAT

# Giving Ip Addresses
For the first step you can go ahead and put all the ip addresses you need on every device (static addressing)

# Green network 
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

# Yellow networks 
you will just set the gateaway information for every interface it's statique routing so there is nothing more to put.

# Blue networks 

In this two blue network we are going to set the basis for the VPN connecion 

## For every network do 
In this network we have one router and this router will be the one managing the vpn to go from his side to ther other side</br>
for exampl from [192.168.1.0/24] to [192.168.2.0/24]

### The cyrpto command 
It's tool provided by Cisco to configure IpSec(Ip Security) on a network, IPSec provides security for transmission of sensitive information over unprotected networks such as the Internet. IPSec provides a robust security solution and is standards-based. IPSec provides data authentication and anti-replay services in addition to data confidentiality services.

#### Setting the policy 
You can see the policy as a set of rules that will determinse how two diffrent entitier in two diffrent network will comunicate.
```
crypto isakmp policy 1
 encr aes 256
 authentication pre-share
 group 5
 lifetime 3600
```
in this set of rules the most two obscure one are the **policy 1** and  **group 5**, policy 1 means that  priority of the policy the lowest the number 
the hightiest the priorit, for group it's about diffie-hellman strength, The identifier is used by two IPsec peers to derive a shared secret without transmitting it to each other. The Group sets the strength of the algorithm in bits. The default is Group 5. The lower the Diffie-Hellman group number, the less CPU time it requires to be executed. The higher the D-H group number, the greater the security level.
there is only 3 options:

 1. Group 2 (1024-bit)</br>
 2. Group 5 (1536-bit)</br>
 3. Group 14 (2048-bit)

and the last weird paramter is authentication this means what authentication method we are going to use, here we pick pre-share means we are going to use a simple, password-based key to authenticate.
#### Setting the Authentication key
To set the authentication key use the crypto command chained by isakmp key here is an exmaple 
```
crypto isakmp key <keyString> (ex: badhak) address <remote-host> (ex: 200.1.2.1)
```
#### Setting the  transform set
A transform set is an acceptable combination of security protocols, algorithms and other settings 
```
crypto ipsec transform-set <name-of-the-set> (ex: TR-R2-TO-R1) transforms (ex :esp-aes 256 esp-sha-hmac)  
```
#### Setting the ACL that will allow the connection
```
ip access-list extended VPN-ACL <- just the name
permit ip <current-network-ip> <current-network-inverse-mask> (ex: 192.168.1.0 0.0.0.255)  <remote-network-ip> <remote-network-inverse-mask> (ex: 192.168.2.0 0.0.0.255)
```
#### Setting the ACL that will make sure machines can connect to the internet and that any connection from the current to the remote doens't go throught the internet
```
ip access-list extended NAT-ACL <- just the name
deny ip <current-network-ip> <current-network-inverse-massk> <remote-network-ip> <remote-network-inverse-massk> (ex: 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255)  <- connection from the current to the remote doens't go throught the internet
permit ip <current-network-ip> (ex: 192.168.1.0 0.0.0.255)  any <- make sure machines can connect to the internet (any to say any address)
```

#### Making sure multiple machines can communicate with the outside
```
ip nat inside source list NAT-ACL interface GigabitEthernet0/0/1 overload
```
Here we tell the router to translate the packets from the addresses described in ACL NAT-ACL and replace the source IP address with the one configured on the GigabitEthernet0/0/1 interface, overriding it to allow more than one machine to communicate with the outside
#### Configurating the default route to send packet to a given interface address
```
ip route 0.0.0.0 0.0.0.0 <the-interface-addres> (ex: 200.1.1.2)
```
#### Note set the inside and outside interfacess 
go to the interface and add ether ``ip nat outside`` or ``ip nat inside``




