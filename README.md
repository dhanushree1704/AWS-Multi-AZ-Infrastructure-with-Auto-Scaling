# AWS Multi-AZ Scalable Infrastructure with Auto Scaling

## Project Overview

This project demonstrates the deployment of a highly available and scalable web infrastructure on AWS using a Virtual Private Cloud (VPC), public and private subnets, an Application Load Balancer, and an Auto Scaling Group.

The infrastructure is distributed across multiple Availability Zones to improve availability. EC2 instances host a static website using Nginx, while the Application Load Balancer distributes incoming HTTP requests across healthy instances. The Auto Scaling Group adjusts the number of instances based on CPU utilization.

## AWS Services Used

- **Amazon VPC** – Provides an isolated virtual network.
- **Amazon EC2** – Hosts the website.
- **Amazon Machine Image (AMI)** – Provides the configured instance image.
- **Launch Template** – Defines the configuration for launching EC2 instances.
- **Application Load Balancer (ALB)** – Distributes incoming traffic.
- **Auto Scaling Group (ASG)** – Automatically adjusts instance capacity.
- **Amazon NAT Gateway** – Enables outbound internet access from private subnets.
- **Internet Gateway (IGW)** – Enables internet connectivity for public subnets.
- **Security Groups** – Control inbound and outbound traffic.
- **Nginx** – Serves the static website.

## Architecture

```text
                     Internet Users
                           |
                           v
                Application Load Balancer
                    (Public Subnets)
                           |
                           v
                     Target Group
                           |
                           v
                   Auto Scaling Group
                      /          \
                     v            v
                EC2 Instance   EC2 Instance
                (Private AZ-A) (Private AZ-B)
                     \            /
                      \          /
                       NAT Gateway
                           |
                     Internet Gateway
                           |
                        Internet
```

## Implementation Steps

### 1. Create the VPC and Subnets

- Create a VPC with the CIDR block `10.0.0.0/16`.
- Create two public subnets and two private subnets across two Availability Zones.
- Create public and private route tables and associate them with the appropriate subnets.

### 2. Configure Internet and NAT Gateways

- Attach an Internet Gateway to the VPC.
- Configure the public route table to route internet traffic through the Internet Gateway.
- Create a NAT Gateway in a public subnet and associate an Elastic IP.
- Configure the private route table to route outbound internet traffic through the NAT Gateway.

### 3. Deploy the Website on EC2

- Launch an Ubuntu EC2 instance.
- Install Nginx and deploy the static website files.
- Verify website accessibility through the instance's public IP during initial testing.

### 4. Create an AMI and Launch Template

- Create an AMI from the configured website instance.
- Create a Launch Template using the AMI.
- Configure the instance type, key pair, and appropriate security group.

### 5. Configure the Application Load Balancer

- Create a target group using HTTP on port `80`.
- Create an internet-facing Application Load Balancer.
- Select the public subnets across the Availability Zones.
- Configure an HTTP listener to forward requests to the target group.

### 6. Configure the Auto Scaling Group

- Create an Auto Scaling Group using the Launch Template.
- Select the private subnets across the Availability Zones.
- Attach the target group to the Auto Scaling Group.
- Configure minimum capacity `1`, desired capacity `1`, and maximum capacity `3`.
- Configure a target-tracking scaling policy based on average CPU utilization.

### 7. Test Auto Scaling

Install the stress utility on an EC2 instance managed by the Auto Scaling Group:

```bash
sudo apt update
sudo apt install stress -y
```

Generate CPU load:

```bash
stress --cpu 2 --timeout 300
```

Monitor CPU utilization, Auto Scaling activity history, and the number of running EC2 instances in the AWS Console.

## Project Results

- Created a VPC with public and private subnets across multiple Availability Zones.
- Configured Internet Gateway and NAT Gateway routing.
- Hosted a static website using Nginx on EC2.
- Configured an Application Load Balancer and target group.
- Set up an Auto Scaling Group with a capacity range of 1–3 instances.
- Observed successful scale-out activity, with the group reaching three instances during testing.

## Screenshots

Add project screenshots to the `screenshots/` directory.

Recommended screenshots:

- VPC resource map
- Website accessed through the Load Balancer DNS
- Auto Scaling Group configuration
- Auto Scaling activity history
- EC2 instances after scaling
- Target group showing healthy instances

Example:

```markdown
![VPC Architecture](screenshots/vpc-resource-map.png)

![Auto Scaling Group](screenshots/auto-scaling-group.png)

![Scaling Activity](screenshots/scaling-activity.png)

![Healthy Targets](screenshots/target-group-health.png)
```

Ensure the screenshot filenames match the actual files uploaded to the repository.

## Security Considerations

- Restrict SSH access to trusted IP addresses.
- Allow HTTP traffic to application instances from the Load Balancer security group.
- Keep application instances in private subnets where possible.
- Never upload AWS credentials or private SSH keys to GitHub.
- Review AWS resource usage to avoid unnecessary charges.

## Cleanup

After completing the project:

- Delete the Auto Scaling Group and its managed instances.
- Delete the Application Load Balancer and target group.
- Delete the NAT Gateway and release its Elastic IP if no longer required.
- Terminate standalone EC2 instances.
- Deregister unused AMIs and delete associated snapshots if no longer needed.

## Disclaimer

This project is intended for learning and demonstration purposes. Verify resource configurations, security settings, and AWS costs before deployment.