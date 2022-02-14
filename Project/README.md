# AWS Cloud based project using services such as EC2, VPC and EFS.
### Two private networks including many features such as: VPN, DNS, NAT, IPv4 FORWARDING, CA, FTP, HTTP/HTTPS, RAID5, RAID6, NIS, NFS, SMTP, POP3, IMAP and Wazuh 
<img src="https://user-images.githubusercontent.com/58471643/153781830-0236103d-3f8c-4b93-8a16-261f7d7c32ee.png"/>

## Since both networks are the exact same as the other I will only be describing one.
---
### Control Instance

**Network:**

The only instance with direct access to the internet is this one, the other instances will be using our control machine as a "router".

This instance will have three network interfaces with each one having different purpose.

1. The IP 10.0.100.100/24 is only going to be used by our control instance;
2. The IP 10.0.100.101/24 is only going to allow data to our DMZ instance;
3. The IP 10.0.100.102/24 is going to be the IP of broadcast to the remaining instances on the network.

**Services:**

Certificate Authority, VPN, DNS, NAT, Firewall and Teleport.

---

### DMZ Instance

**Network:**

By using a specific route, such as the one described before, the network will be "safe" from attacks to specific services, such as the FTP and HTTP/HTTPs.

**Services:**

FTP and HTTP/HTTPS

In the Inova network I will be installing vsftpd and Apache2.<br>
In the Enta network I will be installing ProFTPD and Nginx.

---

### Central Instance

**Services**

NFS = Network File System<br>
NIS = Network Information Service

Mail Servers:<br>
In the Inova network I will be installing Postfix and Dovecot.<br>
In the Enta network I will be installing Postfix and Courier.

---

### Wazuh Instance

**Services**

The only service that is going to be installed here will be Wazuh.<br>
Wazuh is a enterprise-ready security monitoring solution for threat detection, integrity monitoring, incident response and compliance.<br>
With this service I will try to achieve, the needed requeriments for SOC2.

---

### Sales and Marketing Instances

**Services**

This last two instances will serve as an example of the following machines that connect to our instance and therefore I will be installing a graphic user interface.
