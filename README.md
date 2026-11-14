# AWS High Availability Web Application Deployment (L2)

## 1. Project Overview

This project demonstrates how to deploy a highly available web application on AWS using two EC2 instances distributed across different Availability Zones, an Application Load Balancer (ALB), and a Target Group.

The objective is to eliminate a single point of failure at the compute layer and ensure that incoming HTTP traffic is routed to healthy web servers.

## 2. Architecture

**Traffic flow:**

```text
                  Internet Users
                        |
                        |
               Application Load
                  Balancer
                   (HTTP 80)
                        |
                 Target Group
                        |
             ---------------------
             |                   |
        Web-Server-A        Web-Server-B
        Availability        Availability
          Zone A              Zone B
        Apache Server       Apache Server
```

## 3. AWS Services Used

| AWS Service | Purpose |
|---|---|
| Amazon EC2 | Hosts the web servers |
| Amazon VPC | Provides network infrastructure |
| Availability Zones | Provides infrastructure redundancy |
| Application Load Balancer | Distributes incoming traffic |
| Target Group | Registers EC2 instances and performs health checks |
| Security Groups | Controls inbound and outbound traffic |

## 4. Infrastructure Configuration

| Component | Configuration |
|---|---|
| AWS Region | ap-south-1 (Mumbai) |
| Operating System | Amazon Linux 2023 |
| Web Server | Apache HTTP Server |
| Number of EC2 instances | 2 |
| Web-Server-A | Availability Zone A |
| Web-Server-B | Availability Zone B |
| Application Load Balancer | Internet-facing |
| Listener | HTTP - Port 80 |
| Target Group | HTTP - Port 80 |
| Health Check Path | / |

## 5. Step 1: Launch EC2 Instances

Created two EC2 instances in separate Availability Zones within the same VPC.

### Instance A

- Instance Name: Web-Server-A
- Operating System: Amazon Linux 2023
- Availability Zone: ap-south-1a
- Web Server: Apache

**User Data Script:**

```bash
#!/bin/bash
dnf update -y
dnf install httpd -y
systemctl enable httpd
systemctl start httpd
echo "Hello from Server A" > /var/www/html/index.html
```

### Instance B

- Instance Name: Web-Server-B
- Operating System: Amazon Linux 2023
- Availability Zone: ap-south-1b
- Web Server: Apache

**User Data Script:**

```bash
#!/bin/bash
dnf update -y
dnf install httpd -y
systemctl enable httpd
systemctl start httpd
echo "Hello from Server B" > /var/www/html/index.html
```

## 6. Step 2: Security Group Configuration

Created separate security groups for the Application Load Balancer and EC2 instances.

### ALB Security Group

| Type | Protocol | Port | Source |
|---|---|---|---|
| HTTP | TCP | 80 | 0.0.0.0/0 |

### EC2 Security Group

| Type | Protocol | Port | Source |
|---|---|---|---|
| HTTP | TCP | 80 | ALB Security Group |

This configuration allows internet users to access the ALB while restricting direct HTTP access to the EC2 instances.

## 7. Step 3: Create Target Group

Created a target group with the following configuration:

- Target Type: Instances
- Protocol: HTTP
- Port: 80
- Health Check Protocol: HTTP
- Health Check Path: /

Registered both EC2 instances:

1. Web-Server-A
2. Web-Server-B

Waited until both registered targets showed **Healthy** status.

## 8. Step 4: Create Application Load Balancer

Created an internet-facing Application Load Balancer.

Configuration:

- Load Balancer Type: Application Load Balancer
- Scheme: Internet-facing
- IP Address Type: IPv4
- VPC: Default VPC
- Availability Zones: Two AZs containing the EC2 instances
- Listener: HTTP - Port 80
- Default Action: Forward to the created Target Group

After creation, waited until the ALB status changed to **Active**.

## 9. Step 5: Testing and Validation

Copied the DNS name from the Application Load Balancer details.

Opened the ALB DNS name in a web browser:

```text
http://<ALB-DNS-NAME>
```

Expected responses:

```text
Hello from Server A
```

or

```text
Hello from Server B
```

Repeated requests to verify that traffic was distributed between the registered targets.

**Note:** ALB uses load-balancing algorithms and connection behavior, so consecutive browser refreshes are not guaranteed to alternate between the two servers.

## 10. High Availability Validation

To test the health-check mechanism:

1. Confirm both EC2 targets are Healthy.
2. Stop the Apache service on Web-Server-A.
3. Wait for the ALB health check to mark Web-Server-A Unhealthy.
4. Access the ALB DNS name.
5. Verify that requests are served by Web-Server-B.
6. Restart Apache on Web-Server-A.
7. Wait until it becomes Healthy again.

This demonstrates that the ALB can stop routing new requests to an unhealthy target when another healthy target is available.

## 11. Screenshots

Add your actual deployment screenshots to a folder named `screenshots/`.

| Screenshot | Description |
|---|---|
| 01-ec2-instances.png | Both EC2 instances in different AZs |
| 02-security-groups.png | ALB and EC2 security group rules |
| 03-target-group.png | Both registered targets |
| 04-target-health.png | Both targets showing Healthy |
| 05-alb-details.png | ALB showing Active status |
| 06-alb-web-server-a.png | Browser showing Server A response |
| 07-alb-web-server-b.png | Browser showing Server B response |
| 08-failover-test.png | Target health and response during failover test |

## 12. Project Outcome

Successfully demonstrated the deployment design for a highly available web application using:

- Two EC2 instances in different Availability Zones.
- Apache web servers configured through EC2 User Data.
- Application Load Balancer for incoming HTTP traffic.
- Target Group health checks.
- Security Group-based traffic restrictions.
- Unhealthy-target handling and recovery testing.

## 13. Cleanup

To avoid unnecessary AWS charges after completing the project:

1. Delete the Application Load Balancer.
2. Delete the Target Group.
3. Terminate both EC2 instances.
4. Delete unused security groups after checking dependencies.

## 14. Author

**Project:** AWS High Availability Web Application Deployment  
**Level:** L2  
**Cloud Platform:** Amazon Web Services (AWS)  
**Region:** Mumbai (ap-south-1)
