# AWS Scalable & Highly Available Web Architecture

<p align="center">
  <img src="https://img.shields.io/badge/AWS-EC2_%7C_ALB_%7C_ASG_%7C_VPC-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white" alt="AWS" />
  <img src="https://img.shields.io/badge/High_Availability-Multi--AZ-0073BB?style=for-the-badge&logo=amazonaws&logoColor=white" alt="Multi-AZ" />
  <img src="https://img.shields.io/badge/Elastic_Scaling-Dynamic_Policies-43B02A?style=for-the-badge&logo=grafana&logoColor=white" alt="Auto Scaling" />
  <img src="https://img.shields.io/badge/Monitoring-CloudWatch-FF4F00?style=for-the-badge&logo=amazoncloudwatch&logoColor=white" alt="CloudWatch" />
</p>

---

## 📌 Project Overview

This project implements a production-grade, fault-tolerant, and dynamically scalable cloud infrastructure on **Amazon Web Services (AWS)**. It demonstrates how to decouple incoming user traffic from backend compute instances using an **Application Load Balancer (ALB)** and maintain seamless availability across multiple availability zones using **Auto Scaling Groups (ASG)**.

The architecture is designed to withstand traffic spikes, eliminate single points of failure (SPOF), and automatically replace unhealthy compute instances without service interruption.

---

## 🏛️ High-Availability Architecture

```mermaid
flowchart TD
    Users((Internet Users)) -->|HTTP Traffic Port 80| ALB[Application Load Balancer<br/>Multi-AZ Public Subnets]
    
    subgraph VPC ["AWS Virtual Private Cloud (VPC)"]
        subgraph Target_Group ["ALB Target Group (Health Check: HTTP /)"]
            ALB -->|Route Healthy Traffic| AZ1
            ALB -->|Route Healthy Traffic| AZ2
        end

        subgraph ASG ["Auto Scaling Group (Min: 1, Desired: 2, Max: 4)"]
            subgraph AZ1 ["Availability Zone 1 (ap-south-1a)"]
                EC2_1["EC2 Instance 1<br/>t3.micro (Web Tier)"]
            end
            
            subgraph AZ2 ["Availability Zone 2 (ap-south-1b)"]
                EC2_2["EC2 Instance 2<br/>t3.micro (Web Tier)"]
            end
        end

        CW["Amazon CloudWatch<br/>CPU Utilization Metric"] -->|Trigger Alarm (>70%)| ScalingPolicy["ASG Scaling Policy<br/>Scale-Out / Scale-In"]
        ScalingPolicy -.->|Adjust Desired Capacity| ASG
    end
```

---

## ⚙️ Core Infrastructure Components

### 1. Application Load Balancer (ALB)
* **Traffic Ingress:** Distributes incoming HTTP requests uniformly across compute targets in separate availability zones.
* **Target Health Checks:** Continuously polls target endpoints on port 80; automatically drains and detaches failing instances within 30 seconds.
* **Security Group:** Ingress open on port 80/443; egress constrained to target EC2 security groups.

### 2. Auto Scaling Group (ASG) & Launch Templates
* **Launch Template:** Standardized AMI, instance type (`t3.micro`), IAM role, user-data bootstrap script, and security group.
* **Capacity Management:**
  * **Minimum Capacity:** `1` (ensures minimum baseline presence)
  * **Desired Capacity:** `2` (guarantees cross-AZ redundancy during standard load)
  * **Maximum Capacity:** `4` (absorbs sudden high-volume bursts)
* **Scaling Policies:** Dynamic target tracking based on average CPU utilization threshold (`> 70%`).

### 3. Monitoring & Auto-Healing
* **CloudWatch Telemetry:** Real-time metrics tracking CPU utilization, network I/O, and HTTP 5xx error rates.
* **Automated Replacement:** If an EC2 instance fails an ALB health check or encounters hardware degradation, the ASG automatically terminates the faulty instance and provisions a fresh replacement from the launch template.

---

## 📸 Implementation & Verification Evidence

### 1. Load Balancer Configuration
![Application Load Balancer](images/Screenshot%202026-03-27%20162535.png)

### 2. Target Group & Health Probes
![Target Group](images/Screenshot%202026-03-27%20162550.png)

### 3. Auto Scaling Group Specification
![Auto Scaling Group](images/Screenshot%202026-03-27%20162603.png)

### 4. Dynamic Scaling Activity History
![Scaling Activity](images/Screenshot%202026-03-27%20162624.png)

### 5. CloudWatch Metrics & Instance Telemetry
![CloudWatch Monitoring](images/Screenshot%202026-03-27%20162636.png)

### 6. Security Group & Subnet Networking
![Configurations](images/Screenshot%202026-03-27%20162649.png)

---

## 💡 Key Engineering Takeaways

* **Decoupled Architecture:** Using an Application Load Balancer shields compute instances from direct internet exposure and facilitates seamless rolling deployments.
* **Resilience Testing:** Simulating high CPU workloads triggered CloudWatch alarms and validated that scale-out activities executed without packet drops.
* **Cost vs Availability Balance:** Utilizing `t3.micro` instances with aggressive scale-in policies provides high availability within free-tier cost boundaries.

---

## 🔮 Future Enhancements

- [ ] Codify the entire infrastructure into reusable Terraform modules.
- [ ] Implement HTTPS / TLS certificate termination using AWS Certificate Manager (ACM).
- [ ] Integrate AWS WAF (Web Application Firewall) to protect against common OWASP vulnerabilities.
- [ ] Add an automated stress-testing workflow using Locust or Apache JMeter.
