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
![RDS console](https://github.com/Naveen15github/End-to-End-Cloud-Database-Integration-using-AWS-EC2-and-RDS/blob/5d986faf85493f8d72c0b90a76496ccac216cff5/Screenshot%20(72).png)  
![EC2 instances](https://github.com/Naveen15github/End-to-End-Cloud-Database-Integration-using-AWS-EC2-and-RDS/blob/5d986faf85493f8d72c0b90a76496ccac216cff5/Screenshot%20(73).png)  
![MySQL queries](https://github.com/Naveen15github/End-to-End-Cloud-Database-Integration-using-AWS-EC2-and-RDS/blob/5d986faf85493f8d72c0b90a76496ccac216cff5/Screenshot%20(76).png)

## 🗂 Files Included
- `README.md` (this file)  
- `sql/schema.sql` — DDL to create `myappdb` and core tables  
- `sql/seed.sql` — Seed data to reproduce screenshots and queries  
- `docs/images/*` — architecture + verification screenshots

## ▶️ How to Reproduce
1. Copy SQL to jump-server:  
```bash
scp -i ~/.ssh/<MY_KEYPAIR>.pem sql/schema.sql sql/seed.sql ec2-user@<JUMP_IP>:/home/ec2-user/
# Database Setup, Schema, Seed, Verification & Backups

This document collects the commands, SQL schema, seed data, verification queries, and backup instructions for setting up the `myappdb` MySQL database on RDS. Replace placeholders (like `<MY_KEYPAIR>`, `<JUMP_IP>`, `<RDS_ENDPOINT>`, `<ADMIN_USER>`) with your real values before running any commands.

---

## SSH to Jump Server (or use SSM)

SSH (if using a bastion / jump host):
```bash
ssh -i ~/.ssh/<MY_KEYPAIR>.pem ec2-user@<JUMP_IP>
```

Tip: Prefer SSM Session Manager over public SSH where possible.

---

## Run Schema + Seed

Load schema and seed into your RDS MySQL instance:

Option A: Run schema then seed separately:
```bash
mysql -h <RDS_ENDPOINT> -u <ADMIN_USER> -p < sql/schema.sql
mysql -h <RDS_ENDPOINT> -u <ADMIN_USER> -p myappdb < sql/seed.sql
```

Option B: One-liner to apply both:
```bash
cat sql/schema.sql sql/seed.sql | mysql -h <RDS_ENDPOINT> -u <ADMIN_USER> -p
```

---

## SQL: Schema & Seed

### Schema (sql/schema.sql)
```sql
CREATE DATABASE IF NOT EXISTS myappdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE myappdb;

CREATE TABLE IF NOT EXISTS customers (
  customer_id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(150) NOT NULL,
  email VARCHAR(255) UNIQUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS services (
  service_id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(150) NOT NULL,
  type VARCHAR(100),
  price_per_unit DECIMAL(10,2) DEFAULT 0.00,
  unit VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS subscriptions (
  subscription_id INT AUTO_INCREMENT PRIMARY KEY,
  customer_id INT NOT NULL,
  service_id INT NOT NULL,
  quantity INT DEFAULT 1,
  start_date DATE NOT NULL,
  end_date DATE DEFAULT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ON DELETE CASCADE ON UPDATE CASCADE,
  FOREIGN KEY (service_id) REFERENCES services(service_id) ON DELETE CASCADE ON UPDATE CASCADE,
  INDEX idx_customer (customer_id),
  INDEX idx_service (service_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS projects (
  project_id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(200) NOT NULL,
  customer_id INT NOT NULL,
  budget DECIMAL(12,2) DEFAULT 0.00,
  start_date DATE,
  end_date DATE DEFAULT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ON DELETE RESTRICT ON UPDATE CASCADE,
  INDEX idx_project_customer (customer_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

DROP VIEW IF EXISTS vw_customer_service_summary;
CREATE VIEW vw_customer_service_summary AS
SELECT
  c.customer_id,
  c.name AS customer,
  s.service_id,
  s.name AS service,
  sub.quantity,
  sub.start_date,
  sub.end_date
FROM subscriptions sub
JOIN customers c ON sub.customer_id = c.customer_id
JOIN services s ON sub.service_id = s.service_id;
```

### Seed (sql/seed.sql)
```sql
USE myappdb;

INSERT INTO customers (name, email) VALUES
  ('Alice', 'alice@example.com'),
  ('Bob', 'bob@example.com'),
  ('Charlie', 'charlie@example.com')
ON DUPLICATE KEY UPDATE name=VALUES(name);

INSERT INTO services (name, type, price_per_unit, unit) VALUES
  ('AWS EC2','Compute',0.10,'per hour'),
  ('AWS S3','Storage',0.02,'per GB'),
  ('AWS Lambda','Compute',0.00,'per request'),
  ('Azure VM','Compute',0.12,'per hour'),
  ('Google Cloud Storage','Storage',0.02,'per GB'),
  ('Terraform Setup','DevOps',150.00,'per project')
ON DUPLICATE KEY UPDATE name=VALUES(name), price_per_unit=VALUES(price_per_unit);

INSERT INTO subscriptions (customer_id, service_id, quantity, start_date, end_date) VALUES
  (1, 1, 10, '2025-01-01','2025-12-31'),
  (1, 2, 500, '2025-01-01', NULL),
  (2, 4, 5, '2025-02-01','2025-08-31'),
  (3, 5, 200, '2025-03-01', NULL)
ON DUPLICATE KEY UPDATE quantity=VALUES(quantity);

INSERT INTO projects (name, customer_id, budget, start_date, end_date) VALUES
  ('Website Hosting', 1, 5000.00, '2025-01-10', '2025-03-10'),
  ('Data Analytics', 2, 12000.00, '2025-02-15', '2025-06-15')
ON DUPLICATE KEY UPDATE budget=VALUES(budget);
```

---

## Verification Queries

Run these to verify the database and data were created correctly:

```sql
SHOW DATABASES;
USE myappdb;
SELECT * FROM services;

SELECT c.name AS customer, s.name AS service, sub.quantity, sub.start_date, sub.end_date
FROM subscriptions sub
JOIN customers c ON sub.customer_id = c.customer_id
JOIN services s ON sub.service_id = s.service_id;

SELECT p.name AS project, c.name AS customer, p.budget, p.start_date, p.end_date
FROM projects p
JOIN customers c ON p.customer_id = c.customer_id;
```

---

## Backups & Snapshots

Logical Backup (mysqldump):
```bash
mysqldump -h <RDS_ENDPOINT> -u <ADMIN_USER> -p --databases myappdb > myappdb-backup-$(date +%F).sql
```

Restore Backup:
```bash
mysql -h <RDS_ENDPOINT> -u <ADMIN_USER> -p < myappdb-backup-YYYY-MM-DD.sql
```

Manual RDS Snapshot (AWS CLI):
```bash
aws rds create-db-snapshot --db-instance-identifier my-mysql-db --db-snapshot-identifier my-mysql-db-snapshot-$(date +%F)
```

---

## Security & Best Practices

- Place DB in private subnets only.
- Restrict security groups so only EC2 (or application hosts) can reach RDS.
- Use AWS Secrets Manager for credentials; do not store DB passwords in plain text.
- Enable automated backups and consider Multi-AZ for HA.
- Monitor with CloudWatch metrics & alarms (CPU, freeable memory, replica lag, storage).
- Avoid public SSH; prefer SSM Session Manager for access to jump hosts or runbooks.
- Use least-privilege IAM roles for services that access the database.

