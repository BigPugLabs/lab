# Static configs for Vyos Routers

Below are the commands to manually get to this point

## Install

Download rolling release iso from [vyos.io](https://vyos.io/)
Boot a (1 cpu/1GB ram/2GB disk/2 nic to separate bridges) from iso
Login with user: `vyos` pass: `vyos`
Type `install image` and follow defaults

## vyos-prox
switch to configure mode
`configure`

setup some interfaces
```
set interfaces ethernet eth0 address '192.168.0.6/24'
set interfaces ethernet eth0 description 'outside'
set interfaces ethernet eth1 address '10.0.5.1/24'
set interfaces ethernet eth1 description 'prox lab'
```
Below if we want to set a vlan
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
set nat source rule 10 source address '10.0.53.0/31'
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
