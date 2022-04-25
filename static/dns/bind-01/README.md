# Anycast DNS setup

Running bind on a debian 11 container

## Install bind

```
apt update
apt install bind9
```
Then edit `/etc/bind/named.conf.options` full config is in this folder
```
acl goodclients {
 192.168.0.0/16;
 10.0.0.0/8;
 localhost;
};

options {
	directory "/var/cache/bind";

        allow-query { goodclients; };
        forwarders {
                9.9.9.9;
        };

	dnssec-validation auto;
	listen-on-v6 { any; };
};
```
- the goodclients block allows bind to service outside of its subnet
- seems to respond to requests for v4 addresses

```
systemctl restart bind9
systemctl status bind9
~~~~
     Active: active (running) since Sat 2022-04-23 23:28:40 UTC; 5s ago
~~~~
```

## Add network

edit `/etc/network/interfaces`  
add
```
auto lo:1
iface lo:1 inet static
        address 10.53.53.53/32
```
Then to bring up the new interface  
`systemctl restart networking.service`  

Test with a good `dig @10.53.53.53 google.com
```
root@bind-01:~# dig @10.53.53.53 kicker.de

; <<>> DiG 9.16.27-Debian <<>> @10.53.53.53 kicker.de
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34017
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: b7f8a592fd7a6cab01000000626498613972a2010ad247fe (good)
;; QUESTION SECTION:
;kicker.de.			IN	A

;; ANSWER SECTION:
kicker.de.		300	IN	A	86.109.250.101

;; Query time: 96 msec
;; SERVER: 10.53.53.53#53(10.53.53.53)
;; WHEN: Sun Apr 24 00:22:57 UTC 2022
;; MSG SIZE  rcvd: 82
```
Important bits there are `ANSWER: 1` in the flags section and
 `SERVER: 10.53.53.53#53(10.53.53.53)` which means we got a new not 
cached result and asked the correct place

## FRR

#### pre-reqs
`apt update && apt install gnupg lsb-release`  

Installation instructions [https://deb.frrouting.org/](https://deb.frrouting.org/)

Enable BGP in `/etc/fss/daemons` by setting `bgpd=yes` and then reload 
configs with `systemctl status frr.service`

`vytsh`

```
root@bind-01:~# vtysh 

Hello, this is FRRouting (version 8.2.2).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

bind-01# conf terminal 
bind-01(config)# router bgp 64530
bind-01(config-router)# neighbor 10.5.53.0 remote-as 64520
bind-01(config-router)# address-family ipv4 unicast 
bind-01(config-router-af)# neighbor 10.5.53.0 activate 
bind-01(config-router-af)# network 10.53.53.53/32
bind-01(config-router-af)# end
bind-01# wr mem
Note: this version of vtysh never writes vtysh.conf
Building Configuration...
Integrated configuration saved to /etc/frr/frr.conf
[OK]
```

## TODO

Fix vtysh stuff, RFC8212 got us!!  

have to add filters/prefixes to be able to announce/recieve these days

need to cleanup the `/etc/frr/frr.conf` file but its actually working now
```router bgp 64530
 neighbor 10.5.53.0 remote-as 64520
 !
 address-family ipv4 unicast
  network 10.53.53.53/32
  neighbor 10.5.53.0 route-map default-map in
  neighbor 10.5.53.0 route-map default-map out
 exit-address-family
exit
!
ip prefix-list no-default-route seq 10 deny 0.0.0.0/0
ip prefix-list ClassA seq 10 permit 10.0.0.0/8 le 32
ip prefix-list ClassB seq 10 permit 172.16.0.0/12 le 32
ip prefix-list ClassC seq 10 permit 192.168.0.0/16 le 32
!
route-map default-map permit 10
 match ip address prefix-list no-default-route
exit
!
route-map default-map permit 20
 match ip address prefix-list ClassA
exit
!
route-map default-map permit 30
 match ip address prefix-list ClassB
exit
!
route-map default-map permit 40
 match ip address prefix-list ClassC
exit
!
```
