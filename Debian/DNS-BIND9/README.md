**Installation and configuration of DNS package via SSH.**

-> apt install bind9 bind9-utils bind9-doc -y

-> cd /etc/bind

-> nano named.conf.options
#Uncomment forwarders lines.

-> nano db.forward.enta.pt
#Change the name of the servers to inova.pt.
#Configure the IPs like shown below.
#Configure your zones.
#Should look like this:

$TTL    604800
@       IN      SOA     inova.pt. root.inova.pt. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
                IN      NS      ns
@               IN      A       10.0.100.100
ns              IN      A       10.0.100.100
control         IN      A       10.0.100.100
teleport        IN      A       10.0.100.100
*.teleport      IN      A       10.0.100.100
www             IN      CNAME   inova.pt.
central         IN      A       10.0.100.104
wazuh           IN      A       10.0.100.105
sales           IN      A       10.0.100.106
marketing       IN      A       10.0.100.107



-> nano db.reverse.enta.pt
- Trocar o nome do server e os IPs.

$TTL    604800
@       IN      SOA     enta.pt. root.enta.pt. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      enta.pt.
100.144 IN      PTR     enta.pt.
100.144 IN      PTR     www.enta.pt.
100.144 IN      PTR     central.enta.pt.
100.144 IN      PTR     luxsrv.enta.pt.
100.144 IN      PTR     luxcli.enta.pt.
100.144 IN      PTR     mailsrv.enta.pt.



-> nano named.conf.local
- Colar isto:

zone "enta.pt" {
        type master;
        file "/etc/bind/db.forward.enta.pt";
};

zone "32.172.in-addr.arpa" {
        type master;
        file "/etc/bind/db.reverse.enta.pt";
};



-> systemctl start bind9 && systemctl enable bind9 && systemctl restart bind9 && systemctl status bind9
