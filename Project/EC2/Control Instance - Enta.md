## Lets start by launching the "control.enta.pt" accordingly to the [Instance Configuration](https://github.com/rodrigopires01/AWS-Cloud/blob/main/Project/EC2/Instance%20Configuration.md).
### We have to start here because this instance will be your RootCA

#### Now that the machine has fully launched and is ready to be connected, lets do that via SSH.

Once that you're in, start by changing the hostname of the machine.

```
sudo hostnamectl set-hostname control.enta.pt
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
We need to install a pack of tools.
```
amazon-linux-extras install epel
```
And now the packages that we will be using in our instance.
```
yum install openvpn named iptables-services easy-rsa -y
```
#### EasyRSA Configuration
Copy the EasyRSA package files into a more accessible area
```
cp -r /usr/share/easy-rsa/3.0.8/ /etc/
```
```
cd /etc/easy-rsa/3/
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
set_var EASYRSA_REQ_ORG        "ENTA Certificate Authority"
set_var EASYRSA_REQ_EMAIL      "entaca@enta.pt"
set_var EASYRSA_REQ_OU         "RootCA"

set_var EASYRSA_CA_EXPIRE      3650
set_var EASYRSA_CERT_EXPIRE    1825
```
EasyRSA init-pki
```
./easyrsa init-pki
```
Build RootCA
```
./easyrsa build-ca
```
#### Our RootCA is now created and ready to sign certificates.
Head over to **control.inova.pt** in order to build the SubCA.

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

#### EasyRSA is done for now. Lets move into the next step: OpenVPN
```
cd /etc/openvpn/
```
