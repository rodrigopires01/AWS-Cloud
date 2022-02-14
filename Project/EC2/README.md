# Instance Settings

## INOVA & ENTA
### Control
* Type: t2.small
* VPC: Default
* Network Interface eth0: 10.0.100.100/24
* Network Interface eth1: 10.0.100.101/24
* Network Interface eth2: 10.0.100.102/24
* Storage: 30GB
* Security Group: Default

### DMZ
* Type: t2.micro
* VPC: Default
* Network Interface eth0: 10.0.100.103/24
* Storage: 30GB
* Security Group: Default

### Central
* Type: t2.micro
* VPC: Default
* Network Interface eth0: 10.0.100.104/24
* Storage: 30GB
* Security Group: Default

### Wazuh
* Type: t2.large
* VPC: Default
* Network Interface eth0: 10.0.100.105/24
* Storage: 30GB
* Security Group: Default

### Sales & Marketing
* Type: t2.small
* VPC: Default
* Network Interface eth0: 10.0.100.106/24 | 10.0.100.107/24
* Storage: 30GB
* Security Group: Default

### Remote Client
* Type: t2.micro
* VPC: Client
* Storage: 30GB
* Security Group: Client

---

# Instance Security

## Security Groups - Inbound Rules

### INOVA

All Traffic -> My IP<br>
All Traffic -> 10.0.0.0/8<br>
All Traffic -> 172.16.0.0/12<br>
All Traffic -> Public IP Client<br>
All Traffic -> Public IP Enta<br>

---

### ENTA

All Traffic -> My IP<br>
All Traffic -> 10.0.0.0/8<br>
All Traffic -> 172.16.0.0/12<br>
All Traffic -> Public IP Client<br>
All Traffic -> Public IP Inova<br>

---

### Remote Client

All Traffic -> My IP

## Security Groups - Outbound Rules

### INOVA, ENTA & Remote Client

All Traffic -> Anywhere IPv4

---

# Follow this order so that you won't have any problems setting up packages.

1. control.enta.pt
2. control.inova.pt
3. www.inova.pt
4. central.inova.pt
5. wazuh.inova.pt
6. sales.inova.pt
7. marketing.inova.pt
8. www.enta.pt
9. central.enta.pt
10. wazuh.enta.pt
11. sales.enta.pt
12. marketing.enta.pt
