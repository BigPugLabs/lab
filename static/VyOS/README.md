# Static configs for Vyos Routers

Below are the commands to manually get to this point

## Install

Download rolling release iso from [vyos.io](https://vyos.io/)  
Boot a (1 cpu/1GB ram/2GB disk/2 nic to separate bridges) VM from iso  
Login with user: `vyos` pass: `vyos`  
Type `install image` and follow defaults  
reboot for good measure  

## vyos-prox
switch to configure mode  
$ `configure`

setup some interfaces
```
set interfaces ethernet eth0 address '192.168.0.6/24'
set interfaces ethernet eth0 description 'outside'
set interfaces ethernet eth1 address '10.0.5.1/24'
set interfaces ethernet eth1 description 'prox lab'
```
Below to set a vlan
```
set interfaces ethernet eth1 vif 53 address '10.0.53.0/31'
set interfaces ethernet eth1 vif 53 description 'dns vlan'
```

we need to do something to allow stuff behind the router to access 
the internet, if we don't stuff on the private lan can't access 
stuff on the wider internet (although it can still access stuff directly 
connected to the router). *Hints* - This is likely due to L2 arp
```
set nat source rule 10 outbound-interface 'eth0'
set nat source rule 10 translation address 'masquerade'
set protocols static route 0.0.0.0/0 next-hop 192.168.0.1
```
Finally give it a name, commit the changes and then save so the changes 
persist
```
set system host-name 'vyos-prox'
commit
save
```

## Adding Wireguard links between two hosts

Note the following command is in operation mode  
Run on both systems to get the key pairs used later  
$ `generate pki wireguard key-pair`

On vyos-prox
```
set interfaces wireguard wg0 address '10.21.21.0/31'
set interfaces wireguard wg0 description 'ptp to vyos-metis'
set interfaces wireguard wg0 peer vyos-metis address '192.168.0.16'
set interfaces wireguard wg0 peer vyos-metis allowed-ips '10.21.21.0/31'
set interfaces wireguard wg0 peer vyos-metis allowed-ips '10.15.0.0/16'
set interfaces wireguard wg0 peer vyos-metis allowed-ips '10.0.0.0/8'
set interfaces wireguard wg0 peer vyos-metis port '51820'
set interfaces wireguard wg0 peer vyos-metis public-key '\<KEY FROM REMOTE SYSTEM\>'
set interfaces wireguard wg0 port '51820'
set interfaces wireguard wg0 private-key '\<KEY FROM COMMAND ABOVE\>'
```
- address is the ptp link we are setting up
- allowed-ips are ip ranges on the remote side we want to be allowed to talk to

then to add static routes
```
set protocols static route 10.15.0.0/16 next-hop 10.21.21.1
```

On vyos-metis is similar but we are bringing up the link from this system
as it has less permanence, be sure to generate keys on this end as well!  
$ `generate pki wireguard key-pair`

```
set interfaces wireguard wg0 address '10.21.21.1/31'
set interfaces wireguard wg0 description 'ptp to vyos-prox'
set interfaces wireguard wg0 peer vyos-prox address '192.168.0.6'
set interfaces wireguard wg0 peer vyos-prox allowed-ips '10.21.21.0/31'
set interfaces wireguard wg0 peer vyos-prox allowed-ips '10.5.0.0/16'
set interfaces wireguard wg0 peer vyos-prox persistent-keepalive '15'
set interfaces wireguard wg0 peer vyos-prox port '51820'
set interfaces wireguard wg0 peer vyos-prox public-key '\<KEY FROM REMOTE SYSTEM\>'
set interfaces wireguard wg0 port '51820'
set interfaces wireguard wg0 private-key '\<KEY FROM COMMAND ABOVE\>'
```

then static routing if needed
```
set protocols static route 10.5.0.0/16 next-hop 10.21.21.0
```

## DHCP pools

We wanted some dhcp pools on vyos-metis to simplify a VM setup
```
set service dhcp-server shared-network-name METIS subnet 10.15.0.0/24 default-router '10.15.0.1'
set service dhcp-server shared-network-name METIS subnet 10.15.0.0/24 name-server '192.168.0.53'
set service dhcp-server shared-network-name METIS subnet 10.15.0.0/24 range 0 start '10.15.0.100'
set service dhcp-server shared-network-name METIS subnet 10.15.0.0/24 range 0 stop '10.15.0.200'
```

## BGP setup

TODO writeup, cleanup wireguard allowed-ips  
working but not great configs are in the individual files

## Firewall and other hardening

Yeah, this maybe should have been first rather than last

## VRRP floating IP

To simplify routing for the simple ISP router a single IP can be floated between multiple routers allowing all availiable routes locally whatever VyOS routers are online.

On router 1 (VyOS Metis) to be the primary router

```
set high-availability vrrp group Homelan vrid 2
set high-availability vrrp group Homelan interface eth0
set high-availability vrrp group Homelan address 192.168.0.2/
set high-availability vrrp group Homelan priority 200
set high-availability vrrp group Homelan rfc3768-compatibility
set high-availability vrrp sync-group sync member Homelan
```
