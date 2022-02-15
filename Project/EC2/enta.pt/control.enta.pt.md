## Lets start by launching the "control.enta.pt" accordingly to the [Instance Configuration](https://github.com/rodrigopires01/AWS-Cloud/blob/main/Project/EC2/Instance%20Configuration.md).
### We have to start here because this instance will be your RootCA

#### Now that the machine has fully launched and is ready to be connected, lets do that via SSH.

Once that you're in, start by changing the hostname of the machine.

```
sudo hostnamectl set-hostname control.enta.pt
```

<br>

Now lets change user to root.
```
sudo su -
```

<br>

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
We need to install a pack of tools.
```
amazon-linux-extras install epel
```

<br>

And now the packages that we will be using in our instance.
```
yum install openvpn bind bind-utils iptables-services easy-rsa -y
```

<br>

#### EasyRSA Configuration
Copy the EasyRSA package files into a more accessible area
```
cp -r /usr/share/easy-rsa/3.0.8/ /etc/
```
```
cd /etc/easy-rsa/3/
```

<br>

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
set_var EASYRSA_REQ_ORG        "ENTA Certificate Authority"
set_var EASYRSA_REQ_EMAIL      "entaca@enta.pt"
set_var EASYRSA_REQ_OU         "RootCA"

set_var EASYRSA_CA_EXPIRE      3650
set_var EASYRSA_CERT_EXPIRE    1825
```

<br>

EasyRSA init-pki
```
./easyrsa init-pki
```

<br>

Build RootCA
```
./easyrsa build-ca
```

<br>

#### Our RootCA is now created and ready to sign certificates.
Head over to **control.inova.pt** in order to build the SubCA.
---
Now that you have a SubCA request, ready to be signed, do the following commands.
Paste your SubCA request into here
```
nano pki/reqs/subca.req
```
Sign it
```
./easyrsa sign-req ca subca
```
Copy the Signed SubCA
```
cat /etc/easy-rsa/3/pki/issued/ca.crt
```
Head over to **control.inova.pt** to complete the SubCA.

#### EasyRSA is done for now. Lets move into the next step.

<br>

### OpenVPN
#### You will need two difference certificates for this instance. A client type for the Site-to-Site and a server type for the Remote Access.
#### You can get this certificates in control.inova.pt. They are generated there.
```
cd /etc/openvpn/
```

<br>

Copy the same ta.key that you created in "control.inova.pt" and paste it here.
```
nano ta.key
```

<br>

Generate a Diffie Helman key
```
openssl dhparam -out dh2048.pem 2048
```

<br>

Configurate the Remote Access file
```
nano server_ra.conf
```
```
#####################################################
#   Example of OpenVPN Remote Access config file    #
#   Pay attention to the certificates and routes    #
#####################################################
port 11940
proto udp
dev tun
ca ca.crt
cert vpn.ra.enta.pt.crt
key vpn.ra.enta.pt.key
dh dh2048.pem
server 10.15.0.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt
push "route 172.31.100.0 255.255.255.0"
keepalive 10 120
tls-auth ta.key 0
cipher AES-256-CBC
persist-key
persist-tun
verb 3
explicit-exit-notify 1
#####################################################
```

<br>

Configurate the Site to Site file
```
nano server_ss.conf
```
```
#############################################################
#  Example of OpenVPN Site to Site main server config file  #
#  Pay attention to the IPs, ports, certificates and routes #
#############################################################
lport 11194
remote 52.39.105.130
rport 11194
ifconfig 10.1.100.101 10.1.100.100
route 172.31.100.0 255.255.255.0
proto udp
dev tun
ca ca.crt
cert vpn.ss.enta.pt.crt
key vpn.ss.enta.pt.key
dh dh2048.pem
push "route 10.0.100.0 255.255.255.0"
keepalive 10 120
tls-auth ta.key 0
cipher AES-256-CBC
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 3
explicit-exit-notify 1
tls-client
#############################################################
```

<br>

Disable the default openvpn service
```
systemctl disable openvpn && systemctl stop openvpn
```

<br>

Enable both site to site and remote access services
```
systemctl enable --now openvpn@server_ra
```
```
systemctl enable --now openvpn@server_ss
```

### DNS

<br>

Edit this file.
```
nano /etc/named.conf
```
In "listen-on port 53" add "any;"
In "allow-query" add "any;".
At the end of the file before "include" add our zones. Example:
```
zone "enta.pt" IN {
        type master;
        file "forward.enta.pt";
        allow-update { none; };
};

zone "31.172.in-addr.arpa" IN {
        type master;
        file "reverse.enta.pt";
        allow-update { none; };
};
```

<br>

Create the forward zone.

nano /etc/named/forward.enta.pt
```
$TTL    604800
@       IN      SOA     enta.pt. root.enta.pt. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
                IN      NS      ns
@               IN      A       172.31.100.100
ns              IN      A       172.31.100.100
control         IN      A       172.31.100.100
www             IN      CNAME   enta.pt.
central         IN      A       172.31.100.104
wazuh           IN      A       172.31.100.105
sales           IN      A       172.31.100.106
marketing       IN      A       172.31.100.107
```

<br>

Create the reverse zone.

nano /etc/named/reverse.enta.pt
```
$TTL    604800
@       IN      SOA     enta.pt. root.enta.pt. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      enta.pt.
100.100 IN      PTR     enta.pt.
100.100 IN      PTR     control.enta.pt.
103.100 IN      PTR     www.enta.pt.
104.100 IN      PTR     central.enta.pt.
105.100 IN      PTR     wazuh.enta.pt.
106.100 IN      PTR     sales.enta.pt.
107.100 IN      PTR     marketing.enta.pt.
```

<br>

Change the files permission
```
chown root:named forward.enta.pt
```
```
chown root:named reverse.enta.pt
```

<br>

Restart the service
```
systemctl enable --now named
```

<br>

### IPTables Configuration

This is the easiest configuration, here we're going to add routing destinations so that when we receive a client in our Public IP it will redirect to the right instances

<br>

Flush IP Tables
```
iptables -F
```

<br>

Postrouting
```
iptables -F && iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

<br>

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

<br>

Save rules to IP Tables
```
netfilter-persistent save
```

<br>

## You're done, your control.inova.pt instance is fully configurated!
