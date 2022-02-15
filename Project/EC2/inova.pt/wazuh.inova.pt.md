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

#### Wazuh

This machine only has Wazuh installed and you can follow the instructions how to setup a wazuh [machine here](https://documentation.wazuh.com/current/installation-guide/open-distro/all-in-one-deployment/all-in-one.html).

---

#### Raid6

Go to the EC2 Volume section, create 4 disks and attach them to this instance.
Now lets create the partitions.
```
gdisk /dev/xvdf
```
```
gdisk /dev/xvdg
```
```
gdisk /dev/xvdh
```
```
gdisk /dev/xvdi
```

<br>

Create the Raid6
```
mdadm --create /dev/md0 --level 6 --raid-devices 4 /dev/xvdf1 /dev/xvdg1 /dev/xvdh1 /dev/xvdi1
```

<br>

Create a physical volume using LVM
```
pvcreate /dev/md0
```

<br>

Create a volume group using LVM
```
vgcreate vg0 /dev/md0
```

<br>

Create a logical volume using LVM
```
lvcreate -n lv0 -l +100%FREE vg0
```

<br>

Encrypt your raid with luksCrypt
```
cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/vg0/lv0
```

<br>

Decrypt temporarily your raid
```
cryptsetup luksOpen /dev/vg0/lv0 vg0lv0_crypt
```

<br>

Make filesystem
```
mkfs.xfs /dev/mapper/vg0lv0_crypt
```

<br>

Make a directory 
```
mkdir /mnt/raid6
```

<br>

Mount your filesystem
```
mount /dev/mapper/vg1lv0_crypt /mnt/raid6
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

---

## You're done, your wazuh.inova.pt instance is fully configurated!
