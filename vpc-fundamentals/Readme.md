## **VPC**

* VPC: private network to deploy your resources (regional)
* Subnets allow you to partition your network inside your VPC
  * Public Subnets usually contains:
    * Load Balancers
    * Static Websites
    * Files
    * Public Authentication Layers
  * Private Subnets usually contains:
    * Web application servers
    * Databases
  * Public and private subnets can communicate if they're in the same VPC
* To define access to the internet and between subnets, we need to use **Route Tables**

#### **Internet Gateway**

* Helps our VPC instances connect with the internet
* Public Subnets have a route to the internet gateway

#### **NAT Gateway**

* NAT Gateways (AWS-managed) & NAT Instances (self-managed) allow instances in **Private Subnets** to access the internet while remaining private

#### **NACL (Network ACL)**

* A firewall which controls traffic from and to subnet
* Can have ALLOW or DENY rules
* Are attached at the **Subnet** level
* Rules only include IP addresses

#### **Security Groups**

* A firewall that controls traffic to and from an **ENI / an EC2 instance**
* Can only have ALLOW rules
* Rules include IP Adresses and other security groups

#### **VPC Flow Logs**

* Capture information about IP traffic going into your interfaces:
  * VPC Flow Logs
  * Subnet Flow Logs
  * ENI Flow Logs
* Helps to monitor & troubleshoot connectivity issues. Example:
  * Subnets to internet
  * Subnets to subnets
  * Internet to subnets
* Captures network information from AWS Managed interfaces too: ELB, ElastiCache, RDS, Aurora, etc..
* VPC Flow Logs data can go to S3 / CloudWatch Logs

#### **VPC Peering**

* Connects two VPC privately using AWS's network
* Make them behave as if they were in the same network
* Must not have overlapping CIDR
* VPC Peering connection **is not transitive**, each connection needs to be configured individually

#### **VPC Endpoints**

* Endpoints allow you to connect to AWS Services **using a private network** instead of the public network (the default method)
* This gives you enhanced security and lower latency to access AWS services
* **VPC Endpoint Gateway**: S3 & DynamoDB
* **VPC Endpoint Interface**: all the other services
* **Only used within your VPC**

#### Site to Site VPN

* Connect an on-premises VPN to AWS
* The connection is automatically encrypted
* Goes over the public internet

#### Direct Connect (DX)

* Establish a physical connection betweeen on-premises and AWS
* The connection is private, secure and frast
* Goes over a private network
* Takes at least a month to establish


###  **Site-to-site VPN and Direct Connect cannot access VPC Endpoints**