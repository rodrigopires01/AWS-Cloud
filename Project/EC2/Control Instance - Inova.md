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

Now that you're back and have copied the signed SubCA just paste it as your ca.crt
```
nano pki/ca.crt
```
#### Now you have a Subordinate CA ready to sign certificates.



