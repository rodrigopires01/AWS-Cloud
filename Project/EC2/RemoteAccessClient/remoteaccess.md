## Remote Access Client installations.

### Once the machne is created, start by changing the hostname of the machine.

```
sudo hostnamectl set-hostname client
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
yum install openvpn -y
```

<br>

## OpenVPN
### You can get this certificates in control.inova.pt. They are generated there.
```
cd /etc/openvpn/
```

<br>

Copy the same ta.key that you created in "control.inova.pt" and paste it here.
```
nano ta.key
```

<br>

Configurate the Remote Access file
```
nano server_ra_inova.conf
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

Configurate the Remote Access file
```
nano server_ra_inova.conf
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

Disable the default openvpn service
```
systemctl disable openvpn && systemctl stop openvpn
```

<br>

Enable both site to site and remote access services
```
systemctl enable --now openvpn@server_ra_enta
```
```
systemctl enable --now openvpn@server_ra_enta
```

<br>

## You're done, your control.inova.pt instance is fully configurated!
