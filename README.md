# AWS VPC Setup: Public and Private Subnets Across Two AZs

This guide provides step-by-step instructions to set up an AWS VPC with the following architecture:

![image](https://github.com/user-attachments/assets/3069fa13-6db5-4215-9a9a-c054d42871e5)


- A single **VPC**.
- Two **Availability Zones (AZs)**: **Zone A** and **Zone B**.
- Each AZ contains one **public subnet** and one **private subnet**:
  - **Zone A**: Public Subnet 1 and Private Subnet 1.
  - **Zone B**: Public Subnet 2 and Private Subnet 2.
- Public subnets are connected to the internet using an **Internet Gateway (IGW)**.
- Private subnets communicate securely via a **Virtual Private Gateway (VPG)**.

---

## Key Terminology

### Virtual Private Cloud (VPC)
A Virtual Private Cloud is a logically isolated portion of the AWS Cloud where resources can operate in a defined virtual network. It provides control over IP ranges, subnets, route tables, and gateways.

### Subnets
Subnets divide a VPC into smaller segments to group resources by IP range within an AZ. Public subnets enable external access, while private subnets are restricted from internet access.

### Internet Gateway (IGW)
An IGW enables resources in public subnets to communicate with the internet. It serves as the target for internet-bound traffic in public route tables.

### Virtual Private Gateway (VPG)
A VPG acts as a secure VPN concentrator, facilitating communication between private subnets and external private networks such as on-premises data centers or other VPCs.

### Route Table
A route table is a set of rules, or "routes," that determines the flow of network traffic within a VPC.

### Availability Zone (AZ)
An AZ is a distinct geographical location within an AWS Region, designed for fault isolation and high availability.

---

## Architecture Summary

### VPC Configuration
- **CIDR Block**: `10.0.0.0/24`

### Subnet Configuration

| Zone        | Public Subnet       | Private Subnet       |
|-------------|---------------------|----------------------|
| **Zone A**  | `10.0.0.0/26`       | `10.0.0.128/26`      |
| **Zone B**  | `10.0.0.64/26`      | `10.0.0.192/26`      |

### Gateways
- **Internet Gateway (IGW)**: Connects public subnets to the internet.
- **Virtual Private Gateway (VPG)**: Enables private subnets to communicate securely with each other.

---

## Step-by-Step Implementation

### Step 1: VPC Creation
1. Log in to the **AWS Management Console**.
2. Go to the **VPC Dashboard** and click **Create VPC**.
3. Configure the VPC:
   - **Name**: `My-VPC`
   - **CIDR Block**: `10.0.0.0/24`
   - Leave other settings as default.
4. Click **Create VPC**.

![image](https://github.com/user-attachments/assets/621040e9-2bcd-4eb7-bff0-6173104e6807)


### Step 2: Subnet Creation

#### Zone A
1. Navigate to **Subnets** in the VPC Dashboard.
2. Create `Public-Subnet-1`:
   - **VPC**: Select `My-VPC`.
   - **AZ**: Choose `us-east-1a`.
   - **CIDR Block**: `10.0.0.0/26`.
3. Create `Private-Subnet-1`:
   - **VPC**: `My-VPC`.
   - **AZ**: `us-east-1a`.
   - **CIDR Block**: `10.0.0.128/26`.

#### Zone B
1. Create `Public-Subnet-2`:
   - **VPC**: `My-VPC`.
   - **AZ**: `us-east-1b`.
   - **CIDR Block**: `10.0.0.64/26`.
2. Create `Private-Subnet-2`:
   - **VPC**: `My-VPC`.
   - **AZ**: `us-east-1b`.
   - **CIDR Block**: `10.0.0.192/26`.

![image](https://github.com/user-attachments/assets/53a88960-17bd-49df-a545-3f55e263b817)


### Step 3: Internet Gateway (IGW)
1. In the **VPC Dashboard**, go to **Internet Gateways** and click **Create Internet Gateway**.
2. Configure the IGW:
   - **Name**: `My-IGW`
3. Attach the IGW to `My-VPC`:
   - Select the IGW, then click **Actions** → **Attach to VPC** → Choose `My-VPC`.

![image](https://github.com/user-attachments/assets/cb025cef-79c3-4de5-a172-39bdc11350af)


### Step 4: Virtual Private Gateway (VPG)
1. Navigate to **Virtual Private Gateways** and click **Create Virtual Private Gateway**.
2. Configure the VPG:
   - **Name**: `My-VPG`.
3. Attach the VPG to `My-VPC`:
   - Select the VPG → **Actions** → **Attach to VPC** → Choose `My-VPC`.

![image](https://github.com/user-attachments/assets/e8ee93a5-692d-4879-881c-4b0d753df22c)


### Step 5: Configure Route Tables

#### Public Route Table
1. Create a public route table:
   - Go to **Route Tables** and click **Create Route Table**.
   - **Name**: `Public-Route-Table`.
   - **VPC**: `My-VPC`.
2. Add a route for internet access:
   - Go to the **Routes** tab → **Edit Routes** → Add:
     - **Destination**: `0.0.0.0/0`.
     - **Target**: `My-IGW`.
3. Associate public subnets:
   - In the **Subnet Associations** tab → **Edit Subnet Associations** → Select `Public-Subnet-1` and `Public-Subnet-2`.

![image](https://github.com/user-attachments/assets/1f35dd61-fdf5-40af-ad60-7865d31c6b30)


#### Private Route Table
1. Create a private route table:
   - **Name**: `Private-Route-Table`.
   - **VPC**: `My-VPC`.
2. Enable VPG route propagation:
   - In the **Route Propagation** tab → **Edit Route Propagation** → Select `My-VPG`.
3. Associate private subnets:
   - In **Subnet Associations** → **Edit Subnet Associations** → Select `Private-Subnet-1` and `Private-Subnet-2`.

![image](https://github.com/user-attachments/assets/046c0896-bd3b-4fb7-a98f-ba948bf9a1cd)


### Step 6: Launch EC2 Instances

#### Public Subnets
1. Launch an EC2 instance in `Public-Subnet-1`:
   - Assign a **Public IP Address**.
   - Use a security group allowing SSH (port 22).
2. Repeat for `Public-Subnet-2`.

![image](https://github.com/user-attachments/assets/3654fbec-c9c2-4723-98ed-5b6bc9398e5e)


#### Private Subnets
1. Launch an EC2 instance in `Private-Subnet-1`:
   - Disable **Public IP Address**.
2. Repeat for `Private-Subnet-2`.

---

## Configure SSH Access for Instances
1. Navigate to **Security Groups** → **Create Security Group**:
   - **Name**: `Private-SSH-Access`.
   - Add **Inbound Rule**:
     - **Type**: `SSH`.
     - **Port**: `22`.
     - **Source**: Use your IP (`My IP`) or a specific CIDR.

![image](https://github.com/user-attachments/assets/2bcc5065-e3d3-4c0a-b412-7f0ee80757e8)

       
2. Attach this security group to private instances.

---

## Validation
- Confirm public instances can access the internet.

![image](https://github.com/user-attachments/assets/58c00e82-b94d-4a5e-9709-2fb181ee8e60)

  
- Verify private instances are isolated and connected through the VPG.

---

## References
- [AWS VPC User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/latest/userguide/)
- [AWS Internet Gateway Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
- [AWS Virtual Private Gateway Documentation](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html)
