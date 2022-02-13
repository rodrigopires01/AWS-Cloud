<h1 align="center">AWS Cloud based project using services such as EC2, VPC and EFS. </h1>
<img src="https://user-images.githubusercontent.com/58471643/153730563-b3f12df9-c07e-4043-8dff-0409c6dd58f8.png" align="center"/>

In order to start this project I followed this order:
1. Create both VPCs in each region;
2. On the US East region I began by creating the "control.enta.pt" machine in order to setup a Certificate Authority(We are using a subordinate authority in the other network to sign requests);
3. Creat the whole "inova.pt" network on the US West region;
4. Set up the remote access client;
5. Finally create the "enta.pt" network on the US East region.

<h3 align="center">VPC Creation - US East</h3>
1. Head over to the VPC service and click on the orange button that says "Launch VPC Wizard".<br>
2. Choose "VPC with a Single Public Subnet" and press "Select".<br>
3. In the "IPv4 CIDR block:" type "10.0.0.0/16"<br>
4. Edit the "Public subnet's IPv4 CIDR:" to "10.0.100.0/24"<br>
5. Choose your "Availability Zone:" as "us-west-1a"<br>
6. Press "Create VPC"<br>
