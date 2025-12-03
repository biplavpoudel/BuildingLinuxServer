# Building Linux Servers: DHCP, DNS and DS
Config files for Building Linux Servers (DHCP, DNS, DS) Course.

Note: These configuration files are designed for learning and testing purposes, and not for use in a production environment.

## Setup ISC-Kea reposiory:
First setup Kea3.0 repository in debain as:
```
curl -1sLf \
  'https://dl.cloudsmith.io/public/isc/kea-3-0/setup.deb.sh' \
  | sudo -E bash

```
 Then install isc-kea-common packages as:
 ```
 apt install isc-kea-common
 ```

## Setup BuildingLinuxServer Repository:
Now, make sure you are logged in as root to a Debian system.
Access the  ~ directory and clone this repository there.

```
su -
cd ~ && git clone https://github.com/daveprowse/bls-ddd.git
cd BuildingLinuxSever
```

Copy the individual files to their respective locations as we proceed.
Backup the original configurations file first:

## Configure Kea-DHCP4-Server
For kea-dhcp4.conf file:

```
apt install isc-kea-dhcp4-server
systemctl enable --now isc-kea-dhcp4-server.service

mv /etc/kea/kea-dhcp4.conf /etc/kea/dhcp4.conf.bak
cp /kea/kea-dhcp4.conf /etc/kea/
```


## Notes: 
1. For this test environment I am using Debian 13 (trixie) as a server for DHCP and DNS (with no GUI) with NAT for inter-VM communication.
2. I have used separate Debian VMs for DHCP and DNS using KVM.
3. The configuration files are based on one of my test labs that runs on the 10.0.2.0/24 NAT network.
4. DHCP Server uses 10.0.2.4/32 and DNS Server uses 10.0.2.5/32
