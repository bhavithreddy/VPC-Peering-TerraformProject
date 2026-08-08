# 🌐 Cross-Region AWS VPC Peering with Terraform

> **Production-style AWS networking mini-project** demonstrating multi-region infrastructure provisioning, private cross-VPC communication, routing, security groups, and end-to-end connectivity validation using Terraform.

<p align="center">

![Terraform](https://img.shields.io/badge/Terraform-844FBA?style=for-the-badge\&logo=terraform\&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge\&logo=amazon-aws\&logoColor=white)
![VPC](https://img.shields.io/badge/Amazon%20VPC-FF9900?style=for-the-badge\&logo=amazon-aws\&logoColor=white)
![EC2](https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge\&logo=amazon-aws\&logoColor=white)
![IaC](https://img.shields.io/badge/Infrastructure%20as%20Code-2496ED?style=for-the-badge)

</p>

---

## 📌 Project Overview

This project provisions **two isolated AWS VPCs in different regions** and establishes private communication between them using **cross-region VPC Peering**.

### Regions

| Environment | Region      | VPC CIDR      |
| ----------- | ----------- | ------------- |
| Primary     | `us-east-1` | `10.0.0.0/16` |
| Secondary   | `us-west-2` | `10.1.0.0/16` |

The project demonstrates how EC2 instances in completely separate AWS networks and regions can communicate using **private IP addresses**, without routing application traffic through the public internet.

---

# 🏗️ Architecture

```text
                         AWS CLOUD
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   ┌──────────────────────────┐       ┌────────────────────────┐ │
│   │      us-east-1           │       │       us-west-2        │ │
│   │      PRIMARY VPC         │       │     SECONDARY VPC      │ │
│   │      10.0.0.0/16         │       │      10.1.0.0/16       │ │
│   │                          │       │                        │ │
│   │   ┌──────────────────┐   │       │   ┌─────────────────┐  │ │
│   │   │ Public Subnet    │   │       │   │ Public Subnet   │  │ │
│   │   │ 10.0.1.0/24      │   │       │   │ 10.1.1.0/24     │  │ │
│   │   │                  │   │       │   │                 │  │ │
│   │   │   ┌──────────┐   │   │       │   │  ┌──────────┐   │  │ │
│   │   │   │ EC2-A    │   │   │       │   │  │ EC2-B    │   │  │ │
│   │   │   └────┬─────┘   │   │       │   │  └────┬─────┘   │  │ │
│   │   └────────┼─────────┘   │       │   └───────┼─────────┘  │ │
│   │            │             │       │            │            │ │
│   │      ┌─────▼─────┐       │       │      ┌─────▼─────┐      │ │
│   │      │ Route     │       │       │      │ Route     │      │ │
│   │      │ Table     │       │       │      │ Table     │      │ │
│   │      └─────┬─────┘       │       │      └─────┬─────┘      │ │
│   │            │             │       │            │            │ │
│   │       ┌────▼────┐        │       │       ┌────▼────┐       │ │
│   │       │   IGW   │        │       │       │   IGW   │       │ │
│   │       └─────────┘        │       │       └─────────┘       │ │
│   │                          │       │                          │ │
│   └──────────────┬───────────┘       └───────────┬──────────────┘ │
│                  │                               │                │
│                  │     Cross-Region VPC          │                │
│                  │     Peering Connection        │                │
│                  └───────────────┬───────────────┘                │
│                                  │                                │
│                       Private IP Communication                    │
│                                  │                                │
└──────────────────────────────────┴────────────────────────────────┘
```

---

## 🔑 Key Architectural Decisions

### 1. Separate VPCs per region

Each region has an independent VPC:

```text
Primary VPC    → 10.0.0.0/16
Secondary VPC  → 10.1.0.0/16
```

The CIDR ranges do not overlap, allowing them to be routed through the peering connection.

### 2. Separate Internet Gateways

Each VPC has its own Internet Gateway.

> VPC Peering provides **private connectivity between VPCs**. It does not provide internet connectivity.

Therefore, each VPC maintains its own internet path.

### 3. Explicit routing

Both VPC route tables contain routes for the remote VPC:

```text
10.1.0.0/16 → VPC Peering Connection
10.0.0.0/16 → VPC Peering Connection
```

This creates bidirectional private routing.

### 4. Security-group based access

Cross-VPC traffic is restricted to the peer VPC CIDR rather than allowing unrestricted access.

Example:

```text
ICMP → 10.1.0.0/16
TCP  → 10.1.0.0/16
```

SSH is used for administrative access.

---

# 🎯 What This Project Demonstrates

* ✅ Terraform Infrastructure as Code
* ✅ Multi-region AWS provisioning
* ✅ Terraform provider aliases
* ✅ Cross-region VPC Peering
* ✅ VPC networking
* ✅ Route tables and route propagation concepts
* ✅ Internet Gateways
* ✅ EC2 provisioning
* ✅ Security Group design
* ✅ Private IP communication
* ✅ AWS CLI usage
* ✅ Infrastructure validation
* ✅ Terraform state management
* ✅ Infrastructure teardown

---

# 🧰 Technology Stack

| Technology           | Purpose                              |
| -------------------- | ------------------------------------ |
| **Terraform**        | Infrastructure as Code               |
| **AWS Provider**     | AWS resource provisioning            |
| **Amazon VPC**       | Isolated network environments        |
| **VPC Peering**      | Cross-region private connectivity    |
| **EC2**              | Connectivity test instances          |
| **Internet Gateway** | Internet access for public subnets   |
| **Route Tables**     | Network traffic routing              |
| **Security Groups**  | Network-level access control         |
| **AWS CLI**          | Key-pair and AWS resource operations |

---

# 📁 Project Structure

```text
vpc-peering-cross-region/
│
├── main.tf
│   └── VPCs, subnets, IGWs, route tables,
│       VPC peering and EC2 resources
│
├── variables.tf
│   └── Input variables
│
├── data.tf
│   └── Region-specific AMI lookups
│
├── locals.tf
│   └── Computed values and derived configuration
│
├── outputs.tf
│   └── Public IPs, private IPs, VPC IDs,
│       and peering connection ID
│
├── terraform.tfvars
│   └── Environment-specific values
│
├── .gitignore
│   └── Terraform state, credentials and secrets
│
└── README.md
```

> ⚠️ Never commit private SSH keys, AWS credentials, or sensitive Terraform state to GitHub.

---

# ⚙️ Prerequisites

Before deploying the infrastructure, make sure you have:

* AWS account
* AWS CLI
* Terraform
* AWS credentials configured
* IAM permissions for VPC and EC2 resources
* Access to both `us-east-1` and `us-west-2`

Verify your AWS configuration:

```bash
aws sts get-caller-identity
```

Verify Terraform:

```bash
terraform version
```

---

# 🚀 Deployment

## 1️⃣ Generate SSH Key Pairs

Create a key pair for each region:

```bash
aws ec2 create-key-pair \
  --key-name vpc-peering-demo-east \
  --region us-east-1 \
  --query 'KeyMaterial' \
  --output text > vpc-peering-demo-east.pem
```

```bash
aws ec2 create-key-pair \
  --key-name vpc-peering-demo-west \
  --region us-west-2 \
  --query 'KeyMaterial' \
  --output text > vpc-peering-demo-west.pem
```

Secure the private keys:

```bash
chmod 400 vpc-peering-demo-east.pem
chmod 400 vpc-peering-demo-west.pem
```

---

## 2️⃣ Initialize Terraform

```bash
terraform init
```

Expected:

```text
Terraform has been successfully initialized!
```

---

## 3️⃣ Validate Configuration

```bash
terraform fmt
terraform validate
```

Expected:

```text
Success! The configuration is valid.
```

---

## 4️⃣ Review Infrastructure

```bash
terraform plan
```

Review the resources Terraform intends to create before proceeding.

---

## 5️⃣ Deploy

```bash
terraform apply
```

Confirm with:

```text
yes
```

---

# 🔍 Validation & Testing

Infrastructure deployment succeeding is **not enough**.

The most important part of this project is proving that traffic can actually traverse the cross-region peering connection.

---

## 1️⃣ Get Terraform Outputs

```bash
terraform output
```

You should obtain values such as:

```text
primary_instance_public_ip
primary_instance_private_ip
secondary_instance_public_ip
secondary_instance_private_ip
vpc_peering_connection_id
```

---

## 2️⃣ SSH into Primary EC2

```bash
ssh -i vpc-peering-demo-east.pem \
    ec2-user@<PRIMARY_PUBLIC_IP>
```

---

## 3️⃣ Test Private Connectivity

From the primary EC2 instance:

```bash
ping <SECONDARY_PRIVATE_IP>
```

For example:

```bash
ping 10.1.1.25
```

The important point is that the destination is the **private IP**, not the public IP.

---

## 4️⃣ Test TCP Connectivity

You can also test application-level connectivity:

```bash
nc -zv <SECONDARY_PRIVATE_IP> 80
```

or:

```bash
curl http://<SECONDARY_PRIVATE_IP>
```

depending on the services running on the destination instance.

---

# 🧪 What Successful Testing Proves

A successful private-IP connection demonstrates that the following components are working together:

```text
EC2-A
  │
  ▼
Route Table
  │
  ▼
VPC Peering Connection
  │
  ▼
Secondary VPC Route Table
  │
  ▼
Security Group
  │
  ▼
EC2-B Private IP
```

This validates:

* VPC CIDR configuration
* Peering connection
* Peering acceptance
* Route tables
* Security groups
* EC2 networking
* Cross-region private connectivity

---

# 🧹 Cleanup

When finished with the lab:

```bash
terraform destroy
```

Confirm:

```text
yes
```

This prevents unnecessary AWS charges from running EC2 instances and other resources.

---

# 💼 Real-World Use Cases

Although this is a mini-project, the architecture represents patterns commonly used in enterprise environments.

### 🌎 Multi-Region Disaster Recovery

A primary region may communicate with a secondary region for:

* Database replication
* Backup synchronization
* DR orchestration
* Failover coordination

---

### 🏢 Mergers & Acquisitions

Organizations may need to connect existing AWS environments without immediately redesigning their entire network architecture.

VPC Peering can provide direct connectivity between a small number of VPCs.

---

### 🌍 Data Residency

Applications may need to remain in specific AWS regions while communicating with shared internal services hosted elsewhere.

---

### 💰 Peering vs Transit Gateway

For a small number of VPCs, VPC Peering can be simpler and cheaper.

As the number of VPCs grows, managing many individual peering connections becomes increasingly complex.

A centralized architecture such as **AWS Transit Gateway** can become more appropriate.

---

# 📊 Architecture Trade-offs

| Approach        | Best For                     | Key Consideration                     |
| --------------- | ---------------------------- | ------------------------------------- |
| VPC Peering     | Small number of VPCs         | Non-transitive                        |
| Transit Gateway | Many VPCs                    | Centralized routing + additional cost |
| VPN             | Hybrid connectivity          | Internet-based encrypted tunnel       |
| Direct Connect  | Enterprise hybrid networking | Dedicated connectivity                |

---

# 🔐 Security Considerations

This project intentionally demonstrates several security principles:

### Network segmentation

Each region has an independent VPC.

### Least-privilege network access

Cross-VPC traffic is restricted to the peer CIDR.

### Private communication

Application traffic between EC2 instances uses private IP addresses.

### SSH restriction

For a production environment, avoid:

```text
0.0.0.0/0 → TCP/22
```

Instead restrict SSH to a trusted administrative IP or use AWS Systems Manager Session Manager.

---

# 📸 Project Evidence

Screenshots demonstrating successful deployment and validation:

```text
screenshots/
├── terraform-init.png
├── terraform-plan.png
├── terraform-apply.png
├── aws-vpc-primary.png
├── aws-vpc-secondary.png
├── vpc-peering.png
├── route-tables.png
├── security-groups.png
└── private-connectivity.png
```

These screenshots provide visual evidence of the infrastructure created by Terraform.

---

# 🧠 Key Interview Concepts

Be prepared to explain:

### Networking

* What is a VPC?
* Why must VPC CIDRs not overlap for peering?
* What is VPC Peering?
* Is VPC Peering transitive?
* How does routing work across a peering connection?
* Why are routes required on both sides?
* What is an Internet Gateway?
* Public vs private subnet

### Terraform

* Why use provider aliases?
* How does Terraform manage resources across regions?
* What is Terraform state?
* Why use `terraform plan` before `apply`?
* How do dependencies work between resources?
* Why should Terraform state not be committed to Git?

### AWS Security

* Security Groups vs Network ACLs
* Why restrict cross-VPC traffic to CIDRs?
* Why shouldn't SSH be open to `0.0.0.0/0` in production?
* How would you replace SSH with Systems Manager?

---

# 💼 Resume Highlights

After completing and validating the project, it can be represented on a resume as:

> **Cross-Region AWS VPC Peering with Terraform**
>
> * Automated provisioning of multi-region AWS networking infrastructure across `us-east-1` and `us-west-2` using Terraform provider aliases.
> * Implemented cross-region VPC Peering with bidirectional route-table configuration and CIDR-scoped security groups for private EC2-to-EC2 communication.
> * Validated end-to-end private connectivity using ICMP/TCP traffic over private IPs, demonstrating routing and security controls across isolated AWS networks.
> * Implemented reusable Terraform variables, outputs, provider configurations, and infrastructure teardown workflows following Infrastructure-as-Code practices.

---

# 🚀 Future Enhancements

This project can be extended into a more production-grade architecture by adding:

* [ ] AWS Transit Gateway comparison
* [ ] Terraform modules
* [ ] Remote Terraform state using S3 + DynamoDB/state locking
* [ ] CI/CD using GitHub Actions
* [ ] Terraform security scanning with Checkov
* [ ] Terraform linting with TFLint
* [ ] Secrets management with AWS Secrets Manager
* [ ] AWS Systems Manager Session Manager
* [ ] CloudWatch monitoring
* [ ] VPC Flow Logs
* [ ] Multi-AZ subnets
* [ ] NAT Gateways
* [ ] Private EC2 instances
* [ ] Infrastructure security testing
* [ ] Automated Terraform plan/apply pipeline

---

# ⭐ Skills Demonstrated

```text
AWS
├── VPC
├── EC2
├── Internet Gateway
├── Route Tables
├── Security Groups
└── VPC Peering

Terraform
├── Provider Aliases
├── Variables
├── Locals
├── Data Sources
├── Resources
├── Outputs
└── State Management

Networking
├── CIDR
├── Routing
├── Private IP Communication
├── Cross-Region Networking
└── Network Security

DevOps
├── Infrastructure as Code
├── CLI Automation
├── Validation
└── Reproducible Infrastructure
```

---

## 📌 Final Takeaway

This project demonstrates more than simply creating AWS resources.

It shows the ability to:

> **Design → Provision → Secure → Route → Test → Validate → Destroy**

a multi-region AWS networking environment using Infrastructure as Code.

That makes it a strong foundational project for demonstrating **AWS, Terraform, cloud networking, and DevOps engineering skills**.
