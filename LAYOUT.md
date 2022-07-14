# Network layout


## Home `192.168.0.0/24`

```
Home router 		    192.168.0.1
│
├── prox 		        192.168.0.5
│    └── vyos-prox	    192.168.0.6	AS 64520
│         ├── bind-01   10.5.53.1	AS 64530
│         ├── netsvr    10.5.0.7        AS 64700
│         └── k3s-control    10.5.10.10        AS 64510(metallb)
│
├── netsvr              192.168.0.7	AS 64700
├── proxy               192.168.0.8
│
├── metis		        192.168.0.10
│    └── metis-prox     192.168.0.16	AS 64520
│         └── fedora-01 10.15.0.100	AS 64531
│
├── other physical	    192.168.0.20-29
├── virtual		        192.168.0.30-50
└── dhcp range		    192.168.0.100-200
```

## Service IPs

Anycast dns 	10.53.53.53  
pihole dns	    192.168.0.53  

## vyos-prox `10.5.0.0/16` AS 64520

```
eth0                    192.168.0.6	    wan
│
├── eth1                10.5.0.1/16	    local
│    │                  10.5.0.8/24     proxy
│    │
│    ├── eth1.10        10.5.10.1/24    k3s
│    └── eth1.53        10.5.53.0/31	dns vlan
└── wg0                 10.21.21.0/31	ptp with metis
```

## vyos-metis `10.15.0.0/16` AS 64520

```
eth0                    192.168.0.16    wan
│
├── eth1                10.15.0.1/16    local
│    └── eth1.53        10.15.53.0/31   dns vlan
└── wg0                 10.21.21.1/31   ptp with prox
```
