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

Configurate the Remote Access file to Inova Server
```
nano server_ra_inova.conf
```
```
#####################################################
#   Example of OpenVPN Remote Access config file    #
#   Pay attention to the certificates and routes    #
#####################################################
client
dev tun
proto udp
remote 52.39.105.130 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert vpn.client.pt.crt
key vpn.client.pt.key
remote-cert-tls server
tls-auth ta.key 1
cipher AES-256-CBC
verb 3
#####################################################
```

<br>

Configurate the Remote Access file to Enta Server
```
nano server_ra_inova.conf
```
```
#####################################################
#   Example of OpenVPN Remote Access config file    #
#   Pay attention to the certificates and routes    #
#####################################################
client
dev tun
proto udp
remote 54.147.125.61 11940
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert vpn.client.pt.crt
key vpn.client.pt.key
remote-cert-tls server
tls-auth ta.key 1
cipher AES-256-CBC
verb 3
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
systemctl enable --now openvpn@server_ra_inova
```
```
systemctl enable --now openvpn@server_ra_enta
```

<br>

## You're done, your control.inova.pt instance is fully configurated!
