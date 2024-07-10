

# VPC Setup in AWS

## Objective
Design and set up a Virtual Private Cloud (VPC) with both public and private subnets. Implement routing, security groups, and network access control lists (NACLs) to ensure proper communication and security within the VPC. Work in the AWS EU-West-1 (Ireland) region.

## Requirements
- VPC: `KCVPC`, IPv4 CIDR block: `10.0.0.0/16`
- Public Subnet: `PublicSubnet`, IPv4 CIDR block: `10.0.1.0/24`
- Private Subnet: `PrivateSubnet`, IPv4 CIDR block: `10.0.2.0/24`
- Internet Gateway: `KCVPC-IGW`
- NAT Gateway: `KCVPC-NAT-Gateway`
- Security Groups: `PublicSecurityGroup`, `PrivateSecurityGroup`
- Network ACLs: `PublicSubnetNACL`, `PrivateSubnetNACL`
- Instances: `PublicInstance`, `PrivateInstance`


## Steps to Complete the Project

### 1. Create a VPC
- **Name:** KCVPC
- **IPv4 CIDR block:** 10.0.0.0/16
1. Open the VPC Dashboard in the AWS Management Console.
2. Click on "Create VPC".
3. Enter "KCVPC" for the name.
4. Set the IPv4 CIDR block to "10.0.0.0/16".
5. Click "Create VPC".

- [Link to VPC architecture diagram](./KCVPC%20-%20src1.PNG))

### 2. Create Subnets
- **Public Subnet:**
  - **Name:** PublicSubnet
  - **IPv4 CIDR block:** 10.0.1.0/24
  - **Availability Zone:** Select any one from your region ( I am using Ireland - EU-West-1)
1. In the VPC Dashboard, click on "Subnets" and then "Create Subnet".
2. Select "KCVPC" as the VPC.
3. Enter "PublicSubnet" for the name.
4. Set the IPv4 CIDR block to "10.0.1.0/24".
5. Choose an Availability Zone.
6. Click "Create Subnet".


- [Link to VPC architecture diagram](./KCVPC%20-%20src_2.PNG))

- **Private Subnet:**
  - **Name:** PrivateSubnet
  - **IPv4 CIDR block:** 10.0.2.0/24
  - **Availability Zone:** Select any one from your region (preferably the same as the Public Subnet for simplicity)
1. Repeat the steps above to create the PrivateSubnet.
2. Use "PrivateSubnet" for the name.
3. Set the IPv4 CIDR block to "10.0.2.0/24".
4. Choose an Availability Zone (preferably the same as the Public Subnet).

- [Link to VPC architecture diagram](./KCVPC%20-%20scr3.PNG))

### 3. Configure an Internet Gateway (IGW)
1. In the VPC Dashboard, click on "Internet Gateways" and then "Create Internet Gateway".
2. Name it "KCVPC-IGW".
3. Click "Create Internet Gateway".
4. Once created, select the IGW and click "Actions" -> "Attach to VPC".
5. Select "KCVPC" and click "Attach".

- [Link to VPC architecture diagram](./KCVPC-src5.PNG))
- [Link to VPC architecture diagram](./KCVPC-src6.PNG))

### 4. Configure Route Tables
- **Public Route Table:**
  - **Name:** PublicRouteTable
  - Associate PublicSubnet with this route table.
  - Add a route to the IGW (0.0.0.0/0 -> IGW).
1. In the VPC Dashboard, click on "Route Tables" and then "Create Route Table".
2. Name it "PublicRouteTable" and select "KCVPC" as the VPC.
3. Click "Create".
4. Select the newly created PublicRouteTable and click on the "Routes" tab.
5. Click "Edit routes" and add a route: 
   - Destination: "0.0.0.0/0"
   - Target: The ID of the IGW created earlier (e.g., igw-xxxxxx).
6. Click "Save routes".
7. Click on the "Subnet Associations" tab, "Edit subnet associations", and associate the "PublicSubnet".

- [Link to VPC architecture diagram](./KCVPC-src7.PNG))
- [Link to VPC architecture diagram](./KCVPC-src8.PNG))

- **Private Route Table:**
  - **Name:** PrivateRouteTable
  - Associate PrivateSubnet with this route table.
  - Ensure no direct route to the internet.
1. Repeat the steps above to create the PrivateRouteTable.
2. Use "PrivateRouteTable" for the name.
3. Do not add any routes to the internet.
4. Associate the "PrivateSubnet" with this route table.

- [Link to VPC architecture diagram](./KCVPC-src9.PNG))

### 5. Configure NAT Gateway
1. In the VPC Dashboard, click on "NAT Gateways" and then "Create NAT Gateway".
2. Select "PublicSubnet" for the subnet.
3. Allocate an Elastic IP for the NAT Gateway by clicking "Allocate Elastic IP".
4. Click "Create a NAT Gateway".
5. Go back to the PrivateRouteTable, click on "Routes" -> "Edit routes", and add a route:
   - Destination: "0.0.0.0/0"
   - Target: The ID of the NAT Gateway created earlier (e.g., nat-xxxxxx).
6. Click "Save routes".

- [Link to VPC architecture diagram](./KCVPC-src10.PNG))

### 6. Set Up Security Groups
- **Security Group for Public Instances (e.g., web servers):**
  - Allow inbound HTTP (port 80) and HTTPS (port 443) traffic from anywhere (0.0.0.0/0).
  - Allow inbound SSH (port 22) traffic from a specific IP (e.g., your local IP). ([What is my IP?](https://www.whatismyip.com/))
  - Allow all outbound traffic.
1. In the VPC Dashboard, click on "Security Groups" and then "Create security group".
2. Name it "PublicInstanceSG" and select "KCVPC" as the VPC.
3. Add the following inbound rules:
   - Type: HTTP, Protocol: TCP, Port Range: 80, Source: 0.0.0.0/0
   - Type: HTTPS, Protocol: TCP, Port Range: 443, Source: 0.0.0.0/0
   - Type: SSH, Protocol: TCP, Port Range: 22, Source: Your IP (e.g., 203.0.113.0/32)
4. Allow all outbound traffic by default.

- **Security Group for Private Instances (e.g., database servers):**
  - Allow inbound traffic from the PublicSubnet on required ports (e.g., MySQL port 3306).
  - Allow all outbound traffic.
1. Repeat the steps above to create the "PrivateInstanceSG".
2. Add the following inbound rules:
   - Type: MySQL/Aurora, Protocol: TCP, Port Range: 3306, Source: The CIDR of the PublicSubnet (e.g., 10.0.1.0/24)
3. Allow all outbound traffic by default.

### 7. Network ACLs
- **Public Subnet NACL:** Allow inbound HTTP, HTTPS, and SSH traffic. Allow outbound traffic.
1. In the VPC Dashboard, click on "Network ACLs" and then "Create Network ACL".
2. Name it "PublicSubnetNACL" and select "KCVPC" as the VPC.
3. Add the following inbound rules:
   - Rule #: 100, Type: HTTP, Protocol: TCP, Port Range: 80, Source: 0.0.0.0/0, Allow
   - Rule #: 110, Type: HTTPS, Protocol: TCP, Port Range: 443, Source: 0.0.0.0/0, Allow
   - Rule #: 120, Type: SSH, Protocol: TCP, Port Range: 22, Source: 0.0.0.0/0, Allow
4. Add an outbound rule:
   - Rule #: 100, Type: ALL Traffic, Protocol: ALL, Port Range: ALL, Destination: 0.0.0.0/0, Allow

- **Private Subnet NACL:** Allow inbound traffic from the public subnet. Allow outbound traffic to the public subnet and internet.
1. Repeat the steps above to create the "PrivateSubnetNACL".
2. Add the following inbound rules:
   - Rule #: 100, Type: ALL Traffic, Protocol: ALL, Port Range: ALL, Source: The CIDR of the PublicSubnet (e.g., 10.0.1.0/24), Allow
3. Add an outbound rule:
   - Rule #: 100, Type: ALL Traffic, Protocol: ALL, Port Range: ALL, Destination: The CIDR of the PublicSubnet (e.g., 10.0.1.0/24), Allow

### 8. Deploy Instances
- **Launch an EC2 instance in the PublicSubnet:**
  - Use the public security group.
  - Verify that the instance can be accessed via the internet.
1. In the EC2 Dashboard, click on "Launch Instance".
2. Choose an AMI and instance type.
3. Configure instance details:
   - Network: KCVPC
   - Subnet: PublicSubnet
4. Add storage, tags, and configure the security group (use "PublicInstanceSG").
5. Launch the instance and verify internet access.

- **Launch an EC2 instance in the PrivateSubnet:**
  - Use the private security group.
  - Verify that the instance can access the internet through the NAT Gateway and can communicate with the public instance.
1. Repeat the steps above to launch an instance in the PrivateSubnet.
2. Configure instance details:
   - Network: KCVPC
   - Subnet: PrivateSubnet
3. Configure the security group (use "PrivateInstanceSG").
4. Launch the instance.
5. Verify that the instance can access the internet through the NAT Gateway by SSH into the Public Instance and from there SSH into the Private Instance and check internet connectivity.

- [Link to VPC architecture diagram](./KCVPC-src13-%20Two%20instance%20running%20-%20Private%20and%20Public%20instance.PNG))
- [Link to VPC architecture diagram](./KCVPC-src14.PNG))
- [Link to VPC architecture diagram](./KCVPC-src16%20-%20Now%20able%20to%20login%20to%20the%20private%20instance%20from%20the%20public%20instance%20IP%20now..PNG))

## Deliverables

1. **Detailed Report with Screenshots:**
   - [Link to detailed report](.//)

2. **VPC Architecture Diagram:**
   - [Link to VPC architecture diagram](./KCVPC%20-%20VPC%20Architecture.PNG)

3. **Component Explanations:**
   - **VPC:** A logically isolated section of the AWS cloud where you can launch AWS resources in a virtual network that you define.
   - **Subnets:** Subdivisions within the VPC, where you can group instances based on security and operational needs.
   - **Internet Gateway (IGW):** Allows instances in the VPC to access the internet.
   - **NAT Gateway:** A managed NAT service that enables instances in a private subnet to connect to the internet or other AWS services but prevents the internet from initiating a connection with those instances.
   - **Route Tables:** Used to determine where network traffic is directed.
   - **Security Groups:** Virtual firewalls that control inbound and outbound traffic to instances.
   - **Network ACLs (NACLs):** Provide an additional layer of security at the subnet level.

## Conclusion
This was the process I used to complete this task. 

By following these steps, you will have successfully designed and set up a VPC with both public and private subnets, implemented routing, security groups, and NACLs to ensure proper communication and security within the VPC in the AWS EU-West-1 (Ireland) region.










