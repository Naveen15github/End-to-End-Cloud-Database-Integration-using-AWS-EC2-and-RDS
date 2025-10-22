# End-to-End Cloud Database Integration using AWS EC2 and RDS

A complete cloud-based database integration on AWS. A MySQL RDS instance is securely created and connected to an EC2 instance for backend operations. It replicates a real-world IT infrastructure setup where application servers communicate with managed databases over a private and secure network.

🌟 **Project Summary**  
I designed, provisioned, secured, and validated an AWS setup with an EC2 jump-server and an Amazon RDS MySQL instance, then created schema + seed data and verified queries — all reproducible and documented below. ✅

📚 **Table of Contents**  
- ✨ Project Overview  
- 🏗 Architecture Diagram  
- 📸 Proof-of-Work Screenshots  
- 🗂 Files Included  
- ▶️ How to Reproduce  
- 🧾 SQL: Schema & Seed  
- 🔍 Verification Queries  
- 🛟 Backups & Snapshots  
- 🔒 Security & Best Practices  
- 💡 Font / Styling Tips  
- 🧭 Closing Narrative

## 🎯 Project Overview
This repository demonstrates a secure, production-like database integration pattern:  
- EC2 jump-server (bastion) in a public subnet for admin access (SSM recommended over SSH)  
- Amazon RDS MySQL in private subnets (no public DB access)  
- Security groups restricting DB traffic (3306) only from the jump-server  
- Recommended: AWS Secrets Manager for credentials, automated backups, CloudWatch monitoring

## 🏗 Architecture Diagram
![Architecture diagram — EC2 + RDS](https://github.com/Naveen15github/End-to-End-Cloud-Database-Integration-using-AWS-EC2-and-RDS/blob/5d986faf85493f8d72c0b90a76496ccac216cff5/ChatGPT%20Image%20Oct%2022%2C%202025%2C%2001_51_38%20PM.png)  
*Caption:* Users/Admin → Internet → EC2 Jump Server (public subnet) → Amazon RDS MySQL (private subnet). Security groups restrict DB access to the jump-server. 🔐

## 📸 Proof-of-Work Screenshots
![RDS console](docs/images/6-rds-console.png)  
![EC2 instances](docs/images/7-ec2-instances.png)  
![MySQL queries](docs/images/8-mysql-queries.png)

## 🗂 Files Included
- `README.md` (this file)  
- `sql/schema.sql` — DDL to create `myappdb` and core tables  
- `sql/seed.sql` — Seed data to reproduce screenshots and queries  
- `docs/images/*` — architecture + verification screenshots

## ▶️ How to Reproduce
1. Copy SQL to jump-server:  
```bash
scp -i ~/.ssh/<MY_KEYPAIR>.pem sql/schema.sql sql/seed.sql ec2-user@<JUMP_IP>:/home/ec2-user/
