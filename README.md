# Cross-Region VPC Peering on AWS using Terraform

A mini-project that provisions two isolated AWS VPCs in **different regions** (`us-east-1` and `us-west-2`) and connects them via **cross-region VPC Peering**, enabling private, low-latency communication between EC2 instances using internal IP addresses — without traffic ever touching the public internet.


---

## 📌 What This Project Demonstrates

- Multi-region Terraform provisioning using **provider aliases**
- **VPC Peering** across regions (not just across VPCs in the same region — this is the harder, more realistic case)
- Custom route table configuration to route traffic through a peering connection
- Security group design for controlled cross-VPC access (SSH, ICMP, TCP)
- End-to-end validation: proving two EC2 instances in completely separate networks/regions can privately reach each other

---

## 🏗️ Architecture

```
                         ┌───────────────────────────────────────────┐
                         │              AWS Account                  │
                         └───────────────────────────────────────────┘
        ┌──────────────────────────────┐   ┌──────────────────────────────┐
        │   Region: us-east-1 (Primary)│   │  Region: us-west-2 (Secondary)│
        │                              │   │                              │
        │  ┌────────────────────────┐  │   │  ┌────────────────────────┐  │
        │  │   VPC-A (10.0.0.0/16)  │  │   │  │   VPC-B (10.1.0.0/16)  │  │
        │  │                        │  │   │  │                        │  │
        │  │  ┌──────────────────┐  │  │   │  │  ┌──────────────────┐  │  │
        │  │  │ Public Subnet A  │  │  │   │  │  │ Public Subnet B  │  │  │
        │  │  │                  │  │  │   │  │  │                  │  │  │
        │  │  │  ┌────────────┐  │  │  │   │  │  │  ┌────────────┐  │  │  │
        │  │  │  │  EC2 (A)   │  │  │  │   │  │  │  │  EC2 (B)   │  │  │  │
        │  │  │  └─────┬──────┘  │  │  │   │  │  │  └─────┬──────┘  │  │  │
        │  │  └────────┼─────────┘  │  │   │  │  └────────┼─────────┘  │  │
        │  │           │            │  │   │  │           │            │  │
        │  │     Route Table A      │  │   │  │     Route Table B      │  │
        │  │   0.0.0.0/0  → IGW-A   │  │   │  │   0.0.0.0/0  → IGW-B   │  │
        │  │   10.1.0.0/16→ PCX ────┼──┼───┼──┼──→ 10.0.0.0/16 → PCX   │  │
        │  └───────────┬────────────┘  │   │  └───────────┬────────────┘  │
        │              │                │   │              │                │
        │        ┌─────┴─────┐          │   │        ┌─────┴─────┐          │
        │        │  IGW-A    │          │   │        │  IGW-B    │          │
        │        └───────────┘          │   │        └───────────┘          │
        └──────────────┬─────────────────┘   └──────────────┬─────────────────┘
                        │                                    │
                        │        VPC Peering Connection       │
                        └───────────── (PCX, cross-region) ───┘
                                  Private IP traffic only
                                  (no public internet hop)

  Internet ⇄ IGW-A/IGW-B  → SSH admin access (port 22, from anywhere)
  EC2(A) ⇄ EC2(B) via PCX → ICMP (ping) + all TCP, VPC-CIDR to VPC-CIDR only
```

**Key architectural decisions:**
- Each VPC has its **own Internet Gateway** — peering does not provide internet access, only inter-VPC private connectivity, so each side still needs its own path out.
- The **peering connection (PCX) must be accepted on both sides** and routes must be added in **both** route tables — peering is not automatically bidirectional at the routing layer even after the connection is accepted.
- Security groups scope ICMP/TCP access to the **peer VPC's CIDR block specifically**, not `0.0.0.0/0` — SSH is the only rule intentionally left open to the internet, for admin access.

---

## 🧰 Tech Stack

| Component | Purpose |
|---|---|
| Terraform | Infrastructure as Code — provisions both regions from one codebase |
| AWS Provider (aliased) | Two provider blocks (`aws.east`, `aws.west`) targeting different regions in a single `apply` |
| Amazon VPC | Two isolated networks (10.0.0.0/16 and 10.1.0.0/16) |
| VPC Peering Connection | Cross-region private connectivity |
| Internet Gateway | Public internet access per VPC (for SSH admin access) |
| Route Tables | Direct traffic — internet-bound via IGW, peer-bound via PCX |
| EC2 | Compute instances used to prove connectivity |
| Security Groups | SSH (0.0.0.0/0), ICMP + TCP (peer VPC CIDR only) |
| AWS CLI | Key pair generation per region |

---

## 📁 Project Structure

```
vpc-peering-cross-region/
├── main.tf              # Providers (aliased per region), VPCs, subnets, IGWs, peering connection, EC2 instances
├── variables.tf         # CIDR blocks, instance types, region names, key pair names
├── data.tf       # AMI lookups (region-specific, via data "aws_ami")
├── locals.tf            # Computed/derived values (e.g., peer CIDR references)
├── outputs.tf           # Public IPs, private IPs, VPC IDs, peering connection ID
├── terraform.tfvars     # Actual values (gitignored if it contains anything sensitive)
└── README.md
```

---

## ⚙️ Prerequisites

- AWS CLI installed and configured (`aws configure`)
- Terraform installed (`terraform -version`)
- An AWS account with permissions for VPC, EC2, and IAM resource creation in **two regions**

---

## 🚀 How to Run

**1. Generate SSH key pairs in each region**
```bash
aws ec2 create-key-pair \
  --key-name vpc-peering-demo-east \
  --region us-east-1 \
  --query 'KeyMaterial' \
  --output text > vpc-peering-demo-east.pem

aws ec2 create-key-pair \
  --key-name vpc-peering-demo-west \
  --region us-west-2 \
  --query 'KeyMaterial' \
  --output text > vpc-peering-demo-west.pem

chmod 400 vpc-peering-demo-east.pem vpc-peering-demo-west.pem
```

**2. Initialize and apply Terraform**
```bash
terraform init
terraform plan
terraform apply
```

**3. Validate connectivity**
```bash
# SSH into instance A (us-east-1)
ssh -i vpc-peering-demo-east.pem ec2-user@<EC2-A-public-ip>

# From inside instance A, ping instance B's PRIVATE IP (not public)
ping <EC2-B-private-ip>
```
A successful ping confirms the peering connection, both route tables, and both security groups are configured correctly — this is the actual proof-of-work for the project, not just `terraform apply` succeeding.

**4. Tear down when done**
```bash
terraform destroy
```

---

## 💼 Corporate / Business Perspective

This is a small demo, but it directly mirrors a **very common real-world enterprise networking pattern**. A few reasons this exact setup shows up constantly in production environments:

- **Multi-region disaster recovery and data replication** — companies running a primary region and a DR/standby region (see: any serious HA/DR architecture) need private connectivity between the two for database replication, backup sync, and failover coordination, without exposing that traffic to the public internet.
- **Mergers, acquisitions, and multi-team AWS accounts** — when two teams or two acquired companies each already have their own VPC (sometimes with overlapping architecture decisions made independently), peering is the standard way to connect them without a costly re-architecture.
- **Regulatory/data residency requirements** — some workloads must run in a specific region for compliance (e.g., data residency laws), while still needing to talk to a shared central service (like an internal API or auth service) hosted in another region.
- **Cost control vs. Transit Gateway** — VPC Peering is the cheaper, simpler option for connecting a small, fixed number of VPCs (no hourly gateway charge, only data transfer cost). Enterprises graduate to **AWS Transit Gateway** once they have many-to-many VPC connectivity needs (peering doesn't scale well past a handful of VPCs because it's non-transitive — this is a key business/architecture tradeoff worth knowing).

**What a hiring manager sees in this project:** it demonstrates you understand that "the cloud" isn't one flat network — that networking boundaries, routing, and security are deliberate design decisions, and that you can reason about traffic paths across account/region boundaries. That's a materially more senior signal than a single-VPC demo.

---
