# Building Linux Servers: DHCP, DNS and DS
Config files for Building Linux Servers (DHCP, DNS, DS) Course.

Note: These configuration files are designed for learning and testing purposes, and not for use in a production environment.

For official documentation on installing kea packages, refer to: https://kb.isc.org/docs/isc-kea-packages

## Setup ISC-Kea reposiory:
First setup Kea3.0 repository in debain as:
```
curl -1sLf \
  'https://dl.cloudsmith.io/public/isc/kea-3-0/setup.deb.sh' \
  | sudo -E bash

```
 Then install isc-kea as:
 ```
 apt install isc-kea
 ```

## Setup BuildingLinuxServer Repository:
Now, make sure you are logged in as root to a Debian system.
Access the  ~ directory and clone this repository there.

```
su -
cd ~ && git clone https://github.com/biplavpoudel/BuildingLinuxServer.git
cd BuildingLinuxSever
```

## Configure Kea-DHCP4-Server
For kea-dhcp4.conf file, install the kea-dhcp4 server and then, copy the individual files to their respective locations. Backup the original configurations file first.

```
apt install isc-kea-dhcp4-server
systemctl disable isc-kea-dhcp4-server.service

mv /etc/kea/kea-dhcp4.conf /etc/kea/dhcp4.conf.bak
cp /kea/kea-dhcp4.conf /etc/kea/
```

Test the configuration now. Optionally, use journalctl for more in-depth system logs.
```
kea-dhcp4 -t /etc/kea/kea-dhcp4.conf

journalctl -u isc-kea-dhcp4-server.service
```

Fix any errors and start up the service:
```
systemctl enable --now isc-kea-dhcp4-server.service
systemctl status isc-kea-dhcp4-server.service
```
If green color is shown, you are good to go!

Ensure the server is listening on port 67 for incoming DHCP requests by checking all the open ports:
```
ss -tulnw
```

## Update KVM's NAT Network Connection
I am using a manually created NAT network called ServersNAT, in my KVM host, for all the VMs.
```
sudo virsh net-edit ServersNAT
```
So locate and remove the <dhcp> section inside <ip> element. Then reactivate the network as:
```
sudo virsh net-destroy ServersNAT
sudo virsh net-start ServersNAT
```

## Renew lease on client VMs
Make sure you have isc-dhcp-client package installed.
```
sudo apt install isc-dhcp-client
```
Then renew the lease using:
```
sudo dhclient -r
sudo dhclient
```

## Use Postgressql as Lease Database
Ensure you build and compile the binary package with switch ```-D postgresql=enabled``` .
For more info, visit: https://kea.readthedocs.io/en/kea-3.0.2/arm/install.html#building-with-postgresql-support

```
git clone https://gitlab.isc.org/isc-projects/kea.git
cd kea
git checkout 3.0.2
meson setup build -D postgres=enabled
meson compile -C build
meson install -C build
```

After setup, update /etc/kea/kea-dhcp4.conf as:
```
"database": {
    "type": "postgresql",
    "host": "192.168.254.26",
    "name": "kea_db",
    "user": "kea_user",
    "password": "****"
}
```

In this example, I have set up PostgreSQL database in my KVM host so I used my host IP `192.168.254.26`.

Visit this link to configure the database: https://kea.readthedocs.io/en/kea-3.0.2/arm/admin.html#pgsql-database-create

In the host, modify `/var/lib/data/pgsql/pg_hba.conf` to include:
```
host  kea_db  kea_user  10.0.2.4/32 md5
```

Add a firewall rule to allow incoming connection to 5432 port as:
```
sudo firewall-cmd --zone=libvirt --add-port=5432/tcp --permanent
```

Then test from dhcp1 as: `psql -h 192.168.254.26 -U kea_user -d kea_db`
Then intialize database as: 
```
kea-admin db-init pgsql \
  -h '192.168.254.26' \
  -n 'kea_db' \
  -u 'kea_user' \
  -p '****'
```

Took me a long time to figure it out!!!

## Notes: 
1. For this test environment I am using Debian 13 (trixie) as a server for DHCP and DNS (with no GUI) with NAT for inter-VM communication.
2. I have used separate Debian VMs for DHCP and DNS using KVM.
3. The configuration files are based on one of my test labs that runs on the `10.0.2.0/24` NAT network.
4. DHCP Server uses `10.0.2.4/32` and DNS Server uses `10.0.2.5/32`
