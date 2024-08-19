#  Configuring VPC and S3 in a Private Subnet to Access the Internet Using NAT Gateway

> This guide will help you configure a VPC with a private subnet that can access the internet via a NAT Gateway.  
> The guide also covers creating an S3 bucket, EC2, VPC subnets, Routes, NAT and more

### Step 1: Create a VPC
1. **Login to AWS Console**.
2. Navigate to **VPC Dashboard**.
3. Click **Create VPC**.
   - **Name Tag:** `MyVPC`.
   - **IPv4 CIDR block:** `10.0.0.0/16`.
   - Leave other settings as default and click **Create**.

### Step 2: Create Public and Private Subnets
1. Go to **Subnets** under the VPC Dashboard.
2. Click **Create Subnet**.
   - **VPC:** Select `MyVPC`.
   - **Subnet name:** `PublicSubnet`.
   - **Availability Zone:** Choose any (e.g., `us-east-1a`).
   - **IPv4 CIDR block:** `10.0.1.0/24`.
   - Click **Create**.
3. Repeat the process to create a private subnet:
   - **Subnet name:** `PrivateSubnet`.
   - **IPv4 CIDR block:** `10.0.2.0/24`.
   - Click **Create**.

### Step 3: Create and Attach an Internet Gateway
1. Navigate to **Internet Gateways** under the VPC Dashboard.
2. Click **Create Internet Gateway**.
   - **Name Tag:** `MyInternetGateway`.
   - Click **Create**.
3. Attach the Internet Gateway to `MyVPC`:
   - Select the created gateway, click **Actions** -> **Attach to VPC** -> select `MyVPC`, and click **Attach**.

### Step 4: Configure Route Tables
#### 4.1. Public Route Table
1. Navigate to **Route Tables** under the VPC Dashboard.
2. Select the route table associated with `MyVPC` (there will be one by default).
3. Rename it to `PublicRouteTable` for clarity.
4. Click **Actions** -> **Edit Routes**.
5. Add a new route:
   - **Destination:** `0.0.0.0/0`.
   - **Target:** Select `MyInternetGateway`.
   - Click **Save Changes**.
6. Associate this route table with the `PublicSubnet`:
   - Go to the **Subnet Associations** tab.
   - Click **Edit subnet associations** -> select `PublicSubnet` -> click **Save**.

#### 4.2. Private Route Table
1. Create a new route table:
   - Click **Create Route Table** -> Name it `PrivateRouteTable` -> Select `MyVPC` -> Click **Create**.
2. Associate this route table with the `PrivateSubnet`:
   - Go to the **Subnet Associations** tab.
   - Click **Edit subnet associations** -> select `PrivateSubnet` -> click **Save**.

### Step 5: Create a NAT Gateway
1. Navigate to **NAT Gateways** under the VPC Dashboard.
2. Click **Create NAT Gateway**.
   - **Subnet:** Select `PublicSubnet`.
   - **Elastic IP Allocation ID:** Allocate a new Elastic IP.
   - Click **Create NAT Gateway**.
3. Update the `PrivateRouteTable` to route traffic through the NAT Gateway:
   - Select `PrivateRouteTable`.
   - Click **Edit Routes**.
   - Add a new route:
     - **Destination:** `0.0.0.0/0`.
     - **Target:** Select the NAT Gateway you just created.
     - Click **Save Changes**.

### Step 6: Create an S3 Bucket
1. **Navigate to the S3 Console**:
   - Go to the AWS Management Console and open the **S3** service.

2. **Create a New S3 Bucket**:
   - Click on **Create bucket**.
   - **Bucket name**: Enter a unique name for your bucket (e.g., `my-private-subnet-bucket`).
   - **Region**: Choose the same region where your VPC and EC2 instances are located.
   - **Block Public Access settings for this bucket**: Leave the default settings to block all public access.
   - Click **Create bucket**.

3. **Upload Objects to the Bucket** (Optional):
   - Click on your new bucket name.
   - Use the **Upload** button to add files or objects to the bucket for testing purposes.

### Step 7: Launch an EC2 Instance in the Private Subnet
1. Go to the **EC2 Dashboard** and click **Launch Instance**.
2. **Name and Tags:** Name it `PrivateInstance`.
3. **AMI:** Choose an Amazon Linux 2 AMI (or any preferred OS).
4. **Instance Type:** Choose `t2.micro` (free tier eligible).
5. **Key Pair:** Select or create a key pair to SSH into the instance.
6. **Network Settings:**
   - **VPC:** Select `MyVPC`.
   - **Subnet:** Select `PrivateSubnet`.
   - **Auto-assign Public IP:** Disabled (default).
7. **Security Group:** 
   - Create a new security group or use an existing one.
   - Ensure outbound rules allow access to the internet.
8. Click **Launch Instance**.

### Step 8: Test Connectivity from the Private Subnet to the Internet
1. **SSH into the Private Instance** using a Bastion Host or via the Public Instance in the `PublicSubnet`:
   - SSH into the public instance: 
     ```bash
     ssh -i "YourKeyPair.pem" ec2-user@<PublicIP of PublicInstance>
     ```
   - From the public instance, SSH into the private instance:
     ```bash
     ssh -i "YourKeyPair.pem" ec2-user@<PrivateIP of PrivateInstance>
     ```
2. **Test Internet Access**:
   - From the private instance, test connectivity to the internet:
     ```bash
     ping google.com
     ```
   - If ping works, it confirms that the NAT Gateway is properly routing traffic from the private subnet to the internet.

### Step 9: Test S3 Access from the Private Subnet
1. **Attach an IAM Role to the Private Instance**:
   - Create an IAM role with S3 full access and attach it to the EC2 instance in the private subnet.

2. **Test S3 Access**:
   - SSH into the private instance and run the following command:
     ```bash
     aws s3 ls
     ```
   - This should list your S3 buckets, confirming that the instance in the private subnet can access S3 via the NAT Gateway.

### Step 10: Clean Up (Optional)
Remember to terminate the EC2 instances, delete the NAT Gateway (to avoid costs), and clean up other resources if not needed.

### Give this repo a :star: Keep Learning!
