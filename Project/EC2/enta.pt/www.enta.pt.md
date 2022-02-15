## Configuration of the "www.enta.pt" instance.

#### Connect to your control.enta.pt instance and access your www.enta.pt via SSH.

```
ssh -i chave.pem ubuntu@www.enta.pt
```

<br>

Once that you're in, start by changing the hostname of the machine.

```
sudo hostnamectl set-hostname www.enta.pt
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

<br>

#### Since this machine is access by a private IP and therefore it doesn't have internet connection we need to configure our netplan with the gateway and dns settings.
```
nano /etc/sysconfig/network-scripts/ifcfg-eth0
```
Be careful with the IPs and MAC Addresses.
```
DEVICE=eth0
BOOTPROTO=dhcp
ONBOOT=yes
TYPE=Ethernet
USERCTL=yes
PEERDNS=yes
DHCPV6C=yes
DHCPV6C_OPTIONS=-nw
PERSISTENT_DHCLIENT=yes
RES_OPTIONS="timeout:2 attempts:5"
DHCP_ARP_CHECK=no
GATEWAY=172.31.100.101
DNS1=172.31.100.100
DNS2=8.8.8.8
```

<br>

Restart your network to apply changes.
```
systemctl restart network
```

<br>

#### Lets start installing the packages we will need.

```
yum install epel-release
```
```
yum install proftpd proftpd-utils
```
```
amazon-linux-extras install nginx1
```

<br>

### FTP Configuration

<br>

After downloading and installing the packages you just need to start the service.
```
systemctl enable --now proftpd
```

### EFS Mount
Follow this [guide](https://github.com/rodrigopires01/AWS-Cloud/blob/main/Project/EFS/EFS%20East%20Region.md) to launch your EFS.

On the terminal type:
```
cd / && mkdir efs/
```

<br>

This should be provided in the EFS menu, this is just an example of what it looks like.
```
mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport 10.0.100.53:/ efs
```

<br>

Create the folder for Apache.
```
cd /efs/ && mkdir html && mkdir htmls
```

<br>

Type something in each file.
```
nano html/index.html && nano htmls/index.html
```

<br>

Copy the last line.
```
cat /etc/mtab
```

<br>

Paste the line you just copied and add "nofail".
```
nano /etc/fstab
```

<br>

#### NGINX Configuration

Start service.
```
systemctl enable --now nginx
```

<br>

## You're done, your www.inova.pt instance is fully configurated!
