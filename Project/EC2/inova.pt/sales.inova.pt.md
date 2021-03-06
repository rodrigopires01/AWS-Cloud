## Configuration of the "sales.inova.pt" instance.

#### Connect to your control.inova.pt instance and access your sales.inova.pt via SSH.

```
ssh -i chave.pem ubuntu@sales.inova.pt
```

<br>

Once that you're in, start by changing the hostname of the machine.

```
sudo hostnamectl set-hostname sales.inova.pt
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

<br>

Lets try and apply our changes
```
netplan try
```
```
netplan apply
```

<br>

#### Now lets start installing the packages we will need.
Update the Operating System.
```
apt update && apt upgrade -y
```

<br>

Install the packages that we will be using in our instance.
```
apt install -y xfce4 xfce4-goodies xrdp chromium-browser filezilla nfs-common nis
```

---

#### Graphic Interface

By executing the previous command, you get a graphic interface installed on your instance in which you can use a RDP Client to connect to it.

Exit the root so that you can execute this comand.
Note: This is important for you connection, otherwise it won't work.
```
exit
```
```
echo xfce4-session > ~/.xsession
```
```
sudo su -
```

<br>

There you go, you graphic interface is ready to be used.

#### NFS Client

Since we're already installed the NFS on the client we just need to do some simple configurations.
Make a directory to share the files.
```
mkdir -p /mnt/raid5/user_homes
```

<br>

Mount the file system
```
mount -t nfs 10.0.100.104:/mnt/raid5/user_homes /mnt/raid5/user_homes
```

Copy the last line.
```
cat /etc/mtab
```

<br>

Paste the line you just copied and add "nofail".
```
nano /etc/fstab
```

#### NIS Client

For nis you have to add this line: ypserver central.inova.pt
```
nano /etc/yp.conf
```

<br>

Type "nis" in front of "passwd", "group", "shadow" and "hosts"
```
nano /etc/nsswitch.conf
```
```
passwd:         files systemd nis
group:          files systemd nis
shadow:         files nis
gshadow:        files

hosts:          files mdns4_minimal [NOTFOUND=return] dns nis
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis
```

<br> 

Add this line: session optional        pam_mkhomedir.so skel=/etc/skel umask=077
```
nano /etc/pam.d/common-session 
```

---

## You're done, your sales.inova.pt instance is fully configurated!
