## Lets start by launching the "control.inova.pt" accordingly to the [Instance Configuration](https://github.com/rodrigopires01/AWS-Cloud/blob/main/Project/EC2/Instance%20Configuration.md).

#### Now that the machine has fully launched and is ready to be connected, lets do that via SSH.

Once that you're in, start by changing the hostname of the machine.

```
sudo hostnamectl set-hostname control.inova.pt
```

Now lets change user to root.
```
sudo su -
```

Lets configure our hosts so that we can easily SSH into other machines.
```
nano /etc/hosts
```
Should look something like this: 
```
172.31.100.100  control.enta.pt
172.31.100.103  www.enta.pt
172.31.100.104  central.enta.pt
172.31.100.105  wazuh.enta.pt
172.31.100.106  sales.enta.pt
172.31.100.107  marketing.enta.pt

10.0.100.100  control.inova.pt
10.0.100.103  www.inova.pt
10.0.100.104  central.inova.pt
10.0.100.105  wazuh.inova.pt
10.0.100.106  sales.inova.pt
10.0.100.107  marketing.inova.pt
```

#### Now lets start installing the packages we will need.
Update the Operating System
```
apt update && apt upgrade -y
```
Install the packages that we will be using in our instance.
```
apt install openvpn bind9 bind9-utils netfilter-persistent iptables-persistent easy-rsa -y
```
#### EasyRSA Configuration
Copy the EasyRSA package files into a more accessible area
```
cp -R /usr/share/easy-rsa/ /etc/easy-rsa
```
```
cd /etc/easy-rsa/
```
EasyRSA vars
```
nano vars
```
Paste this:
```
if [ -z "$EASYRSA_CALLER" ]; then
        echo "You appear to be sourcing an Easy-RSA 'vars' file." >&2
        echo "This is no longer necessary and is disallowed. See the section called" >&2
        echo "'How to use this file' near the top comments for more details." >&2
        return 1
fi

set_var EASYRSA_DN     "org"

set_var EASYRSA_REQ_COUNTRY    "PT"
set_var EASYRSA_REQ_PROVINCE   "Azores"
set_var EASYRSA_REQ_CITY       "Ponta Delgada"
set_var EASYRSA_REQ_ORG        "INOVA Certificate Authority"
set_var EASYRSA_REQ_EMAIL      "inovaca@inova.pt"
set_var EASYRSA_REQ_OU         "SubCA"

set_var EASYRSA_CA_EXPIRE      3650
set_var EASYRSA_CERT_EXPIRE    1825
```
EasyRSA init-pki
```
./easyrsa init-pki
```
Build RootCA
```
./easyrsa build-ca subca
```
Copy the request
```
cat /etc/easy-rsa/pki/reqs/ca.req
```
#### Our SubCA request is now created. 


Head over to **control.enta.pt** in order to sign the SubCA.

---

Now that you're back and have copied the signed SubCA just paste it as your ca.crt
```
nano pki/ca.crt
```
#### Now you have a Subordinate CA ready to sign certificates.
Since we have a running SubCA, lets generate all the certificates we are going to need the configure the whole network.
```
./easyrsa build-client-full vpn.client.pt nopass
./easyrsa build-server-full vpn.inova.pt nopass
./easyrsa build-client-full vpn.ss.enta.pt nopass
./easyrsa build-server-full vpn.ra.enta.pt nopass
./easyrsa build-server-full dmz.inova.pt nopass
./easyrsa build-server-full central.inova.pt nopass
```

##### DNS Configuration
Change directory
```
cd /etc/bind
```
Uncomment the forwarders
```
nano named.conf.options
```
Create your forward zone like this
```
nano forward.inova.pt 
```
```
$TTL    604800
@       IN      SOA     inova.pt. root.inova.pt. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
                IN      NS      ns
@               IN      A       10.0.100.100
ns              IN      A       10.0.100.100
control         IN      A       10.0.100.100
teleport        IN      A       10.0.100.100
*.teleport      IN      A       10.0.100.100
www             IN      CNAME   inova.pt.
central         IN      A       10.0.100.104
wazuh           IN      A       10.0.100.105
sales           IN      A       10.0.100.106
marketing       IN      A       10.0.100.107
```
Create your reverse zone like this
```
nano reverse.inova.pt 
```
```
$TTL    604800
@       IN      SOA     inova.pt. root.inova.pt. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      inova.pt.
100.100 IN      PTR     inova.pt.
100.100 IN      PTR     control.inova.pt.
100.100 IN      PTR     teleport.inova.pt.
103.100 IN      PTR     www.inova.pt.
104.100 IN      PTR     central.inova.pt.
105.100 IN      PTR     wazuh.inova.pt.
106.100 IN      PTR     sales.inova.pt.
107.100 IN      PTR     marketing.inova.pt.
```
Define your zones
```
nano named.conf.local
```
```
zone "inova.pt" {
        type master;
        file "/etc/bind/forward.inova.pt";
};

zone "0.10.in-addr.arpa" {
        type master;
        file "/etc/bind/reverse.inova.pt";
};

################################################################################################
#This part is necessary in order to have your server re-direct querys to the "enta.pt" network.#
################################################################################################
zone "enta.pt" {
        type forward;
        forwarders {172.31.100.100;};
};
```
Apply your changes
```
systemctl enable --now bind9.service
```
##### Now that we have DNS Configured and running properly lets re-configure our netplan so that we use our DNS server and add our third network interface. Be careful with the IPs and MAC Addresses.
```
nano /etc/netplan/50-cloud-init.yaml
```
```
network:
    ethernets:
        eth0:
            dhcp4: true
            dhcp4-overrides:
                route-metric: 100
                use-dns: false
            nameservers:
                search: [inova.pt, enta.pt]
                addresses: [10.0.100.100, 172.31.100.100]
            dhcp6: false
            match:
                macaddress: 06:d5:6b:87:6a:e3
            set-name: eth0
        eth1:
            dhcp4: true
            dhcp4-overrides:
                route-metric: 200
            dhcp6: false
            match:
                macaddress: 06:be:09:28:5b:91
            set-name: eth1
        eth2:
            dhcp4: true
            dhcp4-overrides:
                route-metric: 200
            dhcp6: false
            match:
                macaddress: 06:70:45:06:68:2d
            set-name: eth2

    version: 2
```

##### IPTables Configuration

This is the easiest configuration, here we're going to add routing destinations so that when we receive a client in our Public IP it will redirect to the right instances

Flush IP Tables
```
iptables -F
```
Postrouting
```
iptables -F && iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
Prerouting
```
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 10.0.100.103
```
```
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j DNAT --to-destination 10.0.100.103
```
```
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 444 -j DNAT --to-destination 10.0.100.105:443
```
```
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 20 -j DNAT --to-destination 10.0.100.103
```
```
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 21 -j DNAT --to-destination 10.0.100.103
```
```
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 990 -j DNAT --to-destination 10.0.100.103
```
```
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 10000:101000 -j DNAT --to-destination 10.0.100.103
```
Save rules to IP Tables
```
netfilter-persistent save
```









































