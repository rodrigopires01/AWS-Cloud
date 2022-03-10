## Ubuntu Server 20.04 - Firewall using iptables package.

-> apt update && apt upgrade -y

-> apt install netfilter-persistent iptables-persistent -y


-> systemctl enable --now iptables


-> iptables -F


-> netfilter-persitent save


-> nano /etc/iptables/rules.v4



### Nao ficar trancado lado de fora
-A INPUT -i lo -j ACCEPT
<br>
-A INPUT -i eth0 -p tcp -m tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>
-A INPUT -i eth0 -p tcp -m tcp --sport 22 -m state --state NEW,ESTABLISHED -j ACCEPT

### WEST ###
#Permitir SSH para a WEST
<br>
-A INPUT -s 192.168.0.32/27 -i eth1 -p tcp -m tcp --sport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir Porta 80 WEST
<br>
-A INPUT -s 192.168.0.32/27 -i eth1 -p tcp -m tcp --sport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir Porta 8080 WEST
<br>
-A INPUT -s 192.168.0.32/27 -i eth1 -p tcp -m tcp --sport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir Porta 1124 WEST
<br>
-A INPUT -s 192.168.0.32/27 -i eth1 -p udp -m udp --sport 1124 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir PINGs WEST
<br>
-A INPUT -s 192.168.0.32/27 -i eth1 -p icmp -m icmp --icmp-type 8 -m state --state NEW,ESTABLISHED -j ACCEPT

### EAST 
#Permitir SSH para a EAST
<br>
-A INPUT -s 192.168.0.64/27 -i eth2 -p tcp -m tcp --sport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir Porta 443 EAST
<br>
-A INPUT -s 192.168.0.64/27 -i eth2 -p tcp -m tcp --sport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir Porta 8080 EAST
<br>
-A INPUT -s 192.168.0.64/27 -i eth2 -p tcp -m tcp --sport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir Porta 1124 EAST
<br>
-A INPUT -s 192.168.0.64/27 -i eth2 -p udp -m udp --sport 1123 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir PINGs EAST
<br>
-A INPUT -s 192.168.0.64/27 -i eth2 -p icmp -m icmp --icmp-type 8 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#LOGS
<br>
-A INPUT -j LOG --log-prefix "IPTABLES:INPUT:Denied: " --log-level 7
<br>
-A INPUT -j REJECT --reject-with icmp-port-unreachable



### Nao ficar trancado lado de fora

-A OUTPUT -o lo -j ACCEPT
<br>
-A OUTPUT -o eth0 -p tcp -m tcp --sport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>
-A OUTPUT -o eth0 -p tcp -m tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

### WEST ###
#Permitir SSH para a WEST
<br>
-A OUTPUT -d 192.168.0.32/27 -o eth1 -p tcp -m tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir Porta 80 WEST
<br>
-A OUTPUT -d 192.168.0.32/27 -o eth1 -p tcp -m tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir Porta 8080 WEST
<br>
-A OUTPUT -d 192.168.0.32/27 -o eth1 -p tcp -m tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir Porta 8080 WEST
<br>
-A OUTPUT -d 192.168.0.32/27 -o eth1 -p udp -m udp --dport 1124 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir PINGs WEST
<br>
-A OUTPUT -d 192.168.0.32/27 -o eth1 -p icmp -m icmp --icmp-type 0 -m state --state NEW,ESTABLISHED -j ACCEPT

### EAST ###

#Permitir SSH para a EAST
<br>
-A OUTPUT -d 192.168.0.64/27 -o eth2 -p tcp -m tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir Porta 80 EAST
<br>
-A OUTPUT -d 192.168.0.64/27 -o eth2 -p tcp -m tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir Porta 8080 EAST
<br>
-A OUTPUT -d 192.168.0.64/27 -o eth2 -p tcp -m tcp --dport 8080 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir Porta 8080 EAST
<br>
-A OUTPUT -d 192.168.0.64/27 -o eth2 -p udp -m udp --dport 1123 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#Permitir PINGs EAST
<br>
-A OUTPUT -d 192.168.0.64/27 -o eth2 -p icmp -m icmp --icmp-type 0 -m state --state NEW,ESTABLISHED -j ACCEPT
<br>

#LOGS
<br>
-A OUTPUT -j LOG --log-prefix "IPTABLES:OUTPUT:Denied: " --log-level 7
<br>
-A OUTPUT -j REJECT --reject-with icmp-port-unreachable
