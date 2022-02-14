## Configuration of the "wazuh.inova.pt" instance.

#### Connect to your control.inova.pt instance and access your wazuh.inova.pt via SSH.

```
ssh -i chave.pem ubuntu@wazuh.inova.pt
```

<br>

Once that you're in, start by changing the hostname of the machine.

```
sudo hostnamectl set-hostname wazuh.inova.pt
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
#### Since this machine is access by a private IP and therefore it doesn't have internet connection we need to configure our netplan with the gateway and dns settings.
```
nano /etc/netplan/50-cloud-init.yaml
```
Be careful with the IPs and MAC Addresses.
```
network:
    ethernets:
        eth0:
            dhcp4: true
            dhcp4-overrides:
                use-dns: false
                use-routes: false
            routes:
              - to: 0.0.0.0/0
                via: 10.0.100.102
                on-link: true
            nameservers:
                search: [inova.pt, enta.pt]
                addresses: [10.0.100.100, 172.31.100.100]
            dhcp6: false
            match:
                macaddress: 06:4c:7d:76:a4:3b
            set-name: eth0
    version: 2
```
#### Now lets start installing the packages we will need.
Update the Operating System.
```
apt update && apt upgrade -y
```

<br>

Install the packages that we will be using in our instance.
```
apt install nis nfs-kernel-server postfix postfix-doc dovecot-imapd dovecot-pop3d sasl2-bin -y
```

This machine only has Wazuh installed and you can follow the instructions how to setup a wazuh [machine here](https://documentation.wazuh.com/current/installation-guide/open-distro/all-in-one-deployment/all-in-one.html).

<br>

## You're done, your www.inova.pt instance is fully configurated!
