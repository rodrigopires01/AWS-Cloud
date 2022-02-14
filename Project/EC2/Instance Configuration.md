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
