<h1 align="center">AWS Cloud based project using services such as EC2, VPC and EFS. </h1>
<img src="https://user-images.githubusercontent.com/58471643/153730563-b3f12df9-c07e-4043-8dff-0409c6dd58f8.png" align="center"/>

<h2 align="center">In order to start this project I followed this order:</h2>
1. Create both VPCs in each region;<br>
2. On the US East region I began by creating the "control.enta.pt" machine in order to setup a Certificate Authority(We are using a subordinate authority in the other network to sign requests);<br>
3. Creat the whole "inova.pt" network on the US West region;<br>
4. Set up the remote access client;<br>
5. Finally create the "enta.pt" network on the US East region.<br>
<br>
<h3>VPC Creation - US West</h3>
1. Head over to the VPC service and click on the orange button that says <b>"Launch VPC Wizard"</b>;<br>
2. Choose <b>"VPC with a Single Public Subnet"</b> and press <b>"Select"</b>;<br>
3. In the <b>"IPv4 CIDR block:"</b> type <b>"10.0.0.0/16"</b>;<br>
4. Edit the <b>"Public subnet's IPv4 CIDR:"</b> to <b>"10.0.100.0/24"</b>;<br>
5. Choose your <b>"Availability Zone:"</b> as <b>"us-west-1a"</b>;<br>
6. Press <b>"Create VPC"</b>.<br>
<br>
<h3>VPC Creation - US East</h3>
1. Head over to the VPC service and click on the orange button that says <b>"Launch VPC Wizard"</b>;<br>
2. Choose <b>"VPC with a Single Public Subnet"</b> and press <b>"Select"</b>;<br>
3. In the <b>"IPv4 CIDR block:"</b> type <b>"172.31.0.0/16"</b>;<br>
4. Edit the <b>"Public subnet's IPv4 CIDR:"</b> to <b>"172.31.100.0/24"</b>;<br>
5. Choose your <b>"Availability Zone:"</b> as <b>"us-east-1a"</b>;<br>
6. Press <b>"Create VPC"</b>.<br>
