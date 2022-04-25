# netsvr

## What we are

Ubuntu 2204 minimal to act as a network management node for the lab

## How to get to where we are

1. On proxmox give us some extra hdd space and a 2nd nic on the private bridge
2. `sudo apt update` and then `sudo apt install qemu-guest-agent` to get details in proxmox
3. mv the current netplan config somewhere 
4. copy the `99_config.yaml` to `/etc/netplan/99_config.yaml`
5. sudo netplan apply

At this stage we should have basic networking all setup

## FRR

Guide here [https://deb.frrouting.org/](https://deb.frrouting.org/)  

1. at time of writing jammy (22.04) doesn't have a release file, trying with `focal` instead  
`echo deb https://deb.frrouting.org/frr focal $FRRVER | sudo tee -a /etc/apt/sources.list.d/frr.list`

2. had to download an older version of libjson-c4  
```
wget http://security.ubuntu.com/ubuntu/pool/main/j/json-c/libjson-c4_0.13.1+dfsg-7ubuntu0.3_amd64.deb
sudo dpkg -i libjson-c4_0.13.1+dfsg-7ubuntu0.3_amd64.deb
```
3. the apt install should work and then add our users to these groups

```
neal@netsvr:~$ sudo usermod -a -G frr neal
neal@netsvr:~$ sudo usermod -a -G frrvty neal
```
4. relogin and `vtysh` will work without sudo
5. edit `/etc/frr/daemons` to show `bgpd=on` then `sudo systemctl reload frr`

TODO - still need to configure it!
