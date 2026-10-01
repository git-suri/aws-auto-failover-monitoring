# Auto-Failover Website Monitoring System on AWS

A website monitoring setup on AWS that detects when a primary web server goes down, sends an email alert, and is designed to redirect traffic to a standby server using Route 53 failover routing.

[Full project write-up (PDF)](AWS-Auto-Failover-Project.pdf)

## Architecture

![Architecture](images/complete_project_architecture.png)

**Flow:** Route 53 health check monitors the primary EC2 → CloudWatch alarm fires on failure → SNS emails the on-call engineer → Route 53 failover routing sends traffic to the secondary EC2.

## AWS Services Used

| Service | Purpose |
|---|---|
| VPC | Isolated network (10.0.0.0/16) with one public subnet, Internet Gateway and route table |
| EC2 (x2) | Primary web server and standby secondary server (Apache on Amazon Linux 2023) |
| Elastic IP | Fixed public IPs for both servers |
| Route 53 | HTTP health check on the primary; failover routing policy design |
| CloudWatch | Alarm on the `HealthCheckStatus` metric (us-east-1) |
| SNS | Email notification when the alarm triggers |

## What I Built

1. Created a VPC with a public subnet, Internet Gateway and a route (0.0.0.0/0 → IGW).
2. Created a security group allowing HTTP (80) and SSH (22 from my IP).
3. Launched two EC2 instances with Apache via user data, each with an Elastic IP.
4. Created an SNS topic with a confirmed email subscription.
5. Created a Route 53 health check (HTTP, port 80) on the primary server.
6. Created a CloudWatch alarm: `HealthCheckStatus` < 1 → notify SNS.
7. Designed Route 53 failover records: primary (with health check) and secondary.

## Testing and Results

I stopped the primary instance to simulate a failure:

| Test | Result |
|---|---|
| Route 53 health check | Changed to **Unhealthy** |
| CloudWatch alarm | Moved to **In alarm** |
| SNS notification | Alert email received |
| Secondary server | Serving "SECONDARY server" on its own Elastic IP |

![Primary server stopped](images/primary-server-stopped.png)
![Unhealthy health check](images/health-check-unhealthy.png)
![Alarm in alarm state](images/cloudwatch-alarm.png)
![SNS email](images/sns-email.png)
![Secondary server](images/secondary-server.png)

> **Note:** Detection and alerting were tested live. The DNS failover records (Route 53 failover routing policy) were designed and documented but not deployed, as that requires a registered domain.

## What I Learned

- How health checks, alarms and notifications connect across AWS services
- Why Route 53 health check metrics live in us-east-1
- Public subnet networking: Internet Gateway, route tables, security groups
- Cost control: cleaning up Elastic IPs, health checks and instances after testing

## Future Improvements

- Deploy the failover records with a registered domain
- Spread servers across multiple Availability Zones
- Add an Application Load Balancer and Auto Scaling
- Provision everything with Terraform or CloudFormation

## Cleanup

All resources (EC2, Elastic IPs, VPC, health check, alarm, SNS topic) were deleted after testing to avoid charges.
