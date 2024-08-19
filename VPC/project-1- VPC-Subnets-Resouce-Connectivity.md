# Configuring a VPC with Public and Private Subnets to Test Resource Connectivity
> This project focuses on setting up a Virtual Private Cloud (VPC) with distinct public and private subnets on AWS, aimed at testing connectivity
> between resources located in different subnets. A web server is launched in the public subnet with a public IP, while a database server is placed in the private subnet,
> isolated from direct internet access. The project involves configuring the VPC, subnets, route tables, and security groups to ensure that the web server can securely connect to the database server. This setup is tested by verifying that the web server in the public subnet can successfully interact with the database server in the private subnet, demonstrating effective segmentation and secure communication within the VPC.

### Step 1: Create a VPC (if not already created)
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
3. Repeat the process for the private subnet:
   - **Subnet name:** `PrivateSubnet`.
   - **IPv4 CIDR block:** `10.0.2.0/24`.
   - Click **Create**.

### Step 3: Create an Internet Gateway
1. Navigate to **Internet Gateways** under the VPC Dashboard.
2. Click **Create Internet Gateway**.
   - **Name Tag:** `MyInternetGateway`.
   - Click **Create**.
3. Attach the Internet Gateway to `MyVPC`:
   - Select the created gateway, click **Actions** -> **Attach to VPC** -> select `MyVPC`, and click **Attach**.

### Step 4: Configure Route Tables
1. Navigate to **Route Tables** under the VPC Dashboard.
2. Select the route table associated with `MyVPC` (there will be one by default).
3. Click **Actions** -> **Edit Routes**.
4. Add a new route:
   - **Destination:** `0.0.0.0/0`.
   - **Target:** Select `MyInternetGateway`.
   - Click **Save Changes**.
5. Associate this route table with the `PublicSubnet`:
   - Go to the **Subnet Associations** tab.
   - Click **Edit subnet associations** -> select `PublicSubnet` -> click **Save**.

### Step 5: Launch EC2 Instances
#### 5.1. Launch the Web Server in Public Subnet
1. Go to the **EC2 Dashboard** and click **Launch Instance**.
2. **Name and Tags:** Name it `WebServer`.
3. **AMI:** Choose an Amazon Linux 2 AMI (or any preferred OS).
4. **Instance Type:** Choose `t2.micro` (free tier eligible).
5. **Key Pair:** Select or create a key pair to SSH into the instance.
6. **Network Settings:**
   - **VPC:** Select `MyVPC`.
   - **Subnet:** Select `PublicSubnet`.
   - Ensure **Auto-assign Public IP** is enabled.
7. **Security Group:**
   - Create a new security group.
   - **Inbound Rule:** Allow `HTTP` (port 80) and `SSH` (port 22).
8. Click **Launch Instance**.

#### 5.2. Launch the Database Server in Private Subnet
1. Go back to the **EC2 Dashboard** and click **Launch Instance**.
2. **Name and Tags:** Name it `DatabaseServer`.
3. **AMI:** Choose an Amazon Linux 2 AMI (or any preferred OS).
4. **Instance Type:** Choose `t2.micro` (free tier eligible).
5. **Key Pair:** Select the same key pair.
6. **Network Settings:**
   - **VPC:** Select `MyVPC`.
   - **Subnet:** Select `PrivateSubnet`.
   - **Auto-assign Public IP:** Disabled (default).
7. **Security Group:**
   - Create a new security group.
   - **Inbound Rule:** Allow `MySQL/Aurora` (port 3306).
   - Source: Custom (specify the CIDR of the public subnet, e.g., `10.0.1.0/24`).
8. Click **Launch Instance**.

### Step 6: Configure the Database Server
1. SSH into the `DatabaseServer` instance:
   ```bash
   ssh -i "YourKeyPair.pem" ec2-user@<PrivateIP of DatabaseServer>
   ```
2. Install MySQL (or your preferred database):
   ```bash
   sudo yum update -y
   sudo yum install mysql-server -y
   sudo service mysqld start
   ```
3. Secure MySQL installation:
   ```bash
   sudo mysql_secure_installation
   ```
4. Create a database and a user:
   ```bash
   mysql -u root -p
   CREATE DATABASE mydatabase;
   CREATE USER 'dbuser'@'%' IDENTIFIED BY 'password';
   GRANT ALL PRIVILEGES ON mydatabase.* TO 'dbuser'@'%';
   FLUSH PRIVILEGES;
   EXIT;
   ```

### Step 7: Connect the Web Server to the Database
1. SSH into the `WebServer` instance:
   ```bash
   ssh -i "YourKeyPair.pem" ec2-user@<PublicIP of WebServer>
   ```
2. Install MySQL client:
   ```bash
   sudo yum update -y
   sudo yum install mysql -y
   ```
3. Test connection to the database:
   ```bash
   mysql -h <PrivateIP of DatabaseServer> -u dbuser -p
   ```
   - Enter the password when prompted.
   - Once you can connect from your web instance , you are 99% successfull with this project.

### Step 8: Deploy Your Web Application
1. Install a web server (e.g., Apache or Nginx) on the `WebServer` instance:
   ```bash
   sudo yum install httpd -y
   sudo service httpd start
   sudo chkconfig httpd on
   ```
2. Deploy your web application to `/var/www/html/` or the appropriate web directory.
3. Modify your web application to connect to the database using the private IP of the `DatabaseServer`. This should work just fine if you passed 7 above

### Step 9: Test the Setup
1. Open a browser and navigate to the public IP of the `WebServer`.
2. Ensure that your web application can successfully connect to the database and display data.

