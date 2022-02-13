<h1 align="center">AWS Cloud based project using services such as EC2, VPC and EFS. </h1>
<p>How to setup a two private networks using Debian and RedHat including many features such as: VPN (Site-to-Site and Remote Access), DNS, NAT, IPv4 FORWARDING, Certificate Authority, FTP, HTTP/HTTPS, RAID5, RAID6, NIS, NFS, SMTP, POP3, IMAP, as well as using Wazuh for security monitoring, threat detection, integrity monitoring, incident response and compliance in order to achieve, or try to, the needed requeriments for SOC 2.</p>
<img src="https://user-images.githubusercontent.com/58471643/153767646-5427af82-7169-41d1-a1a4-7d181260e9aa.png" align="center"/>

<h2 align="center">In order to start this project I followed this order:</h2>
1. Create both VPCs in each region;<br>
2. On the US East region I began by creating the "control.enta.pt" machine in order to setup a Certificate Authority(We are using a subordinate authority in the other network to sign requests);<br>
3. Creat the whole "inova.pt" network on the US West region;<br>
4. Set up the remote access client;<br>
5. Finally create the "enta.pt" network on the US East region.<br>
<br>
<h3>VPC Creation - West Region</h3>
1. Head over to the VPC service and click on the orange button that says <b>"Launch VPC Wizard"</b>;<br>
2. Choose <b>"VPC with a Single Public Subnet"</b> and press <b>"Select"</b>;<br>
3. In the <b>"IPv4 CIDR block:"</b> type <b>"10.0.0.0/16"</b>;<br>
4. Edit the <b>"Public subnet's IPv4 CIDR:"</b> to <b>"10.0.100.0/24"</b>;<br>
5. Choose your <b>"Availability Zone:"</b> as <b>"us-west-1a"</b>;<br>
6. Press <b>"Create VPC"</b>.<br>
<br>
<h3>VPC Creation - East Region</h3>
1. Head over to the VPC service and click on the orange button that says <b>"Launch VPC Wizard"</b>;<br>
2. Choose <b>"VPC with a Single Public Subnet"</b> and press <b>"Select"</b>;<br>
3. In the <b>"IPv4 CIDR block:"</b> type <b>"172.31.0.0/16"</b>;<br>
4. Edit the <b>"Public subnet's IPv4 CIDR:"</b> to <b>"172.31.100.0/24"</b>;<br>
5. Choose your <b>"Availability Zone:"</b> as <b>"us-east-1a"</b>;<br>
6. Press <b>"Create VPC"</b>.<br>
