# End-to-End-Cloud-Database-Integration-using-AWS-EC2-and-RDS
A complete cloud-based database integration on AWS. A MySQL RDS instance is securely created and connected to an EC2 instance for backend operations. It replicates a real-world IT infrastructure setup where application servers communicate with managed databases over a private and secure network.
```markdown

🌟 Project summary — in one line
I designed, provisioned, secured, and validated an AWS setup with an EC2 jump-server and an Amazon RDS MySQL instance, then created schema + seed data and verified queries — all reproducible and documented below. ✅

---

📚 Table of contents
- ✨ Project overview
- 🏗 Architecture diagram
- 📸 Proof-of-work screenshots
- 🗂 Files included
- ▶️ How to reproduce (commands)
- 🧾 SQL: schema & seed (full scripts)
- 🔍 Verification queries
- 🛟 Backups & snapshots
- 🔒 Security & best-practices
- 💡 Font / styling tips for GitHub Pages
- 🧭 Closing narrative

---

🎯 Project overview
This repo demonstrates a secure, production-like database integration pattern:
- EC2 jump-server (bastion) in a public subnet for admin access (recommend SSM instead of SSH)  
- Amazon RDS MySQL in private subnets (no public DB access)  
- Security groups that only allow DB traffic (3306) from the jump-server  
- Secrets Manager recommended for credentials, automated backups enabled, CloudWatch monitoring suggested

---

🏗 Architecture diagram
![Architecture diagram — EC2 + RDS](docs/images/5-architecture.png)  
*Caption:* Users (admin) → Internet → EC2 Jump Server (public subnet) → Amazon RDS MySQL (private subnet). Security groups restrict DB access to the jump-server. 🔐

---

📸 Proof-of-work screenshots
Add these images to docs/images/ for the README to show:
- docs/images/5-architecture.png — Architecture diagram
- docs/images/6-rds-console.png — RDS console: my-mysql-db (Available)
- docs/images/7-ec2-instances.png — EC2 console: db-jump-server (Running)
- docs/images/8-mysql-queries.png — Terminal: queries & results run from jump-server

Example display (they will render after you add the files):
![RDS console](docs/images/6-rds-console.png)  
![EC2 instances](docs/images/7-ec2-instances.png)  
![MySQL queries](docs/images/8-mysql-queries.png)

---

🗂 Files included
- README.md (this file)
- sql/schema.sql — DDL to create `myappdb` and core tables
- sql/seed.sql — Seed data to reproduce screenshots and queries
- docs/images/* — architecture + verification screenshots (add the PNGs)

---

▶️ How to reproduce (quick)
1. Copy SQL to jump-server:
   scp -i ~/.ssh/<MY_KEYPAIR>.pem sql/schema.sql sql/seed.sql ec2-user@<JUMP_IP>:/home/ec2-user/

2. SSH to jump-server (or use SSM):
   ssh -i ~/.ssh/<MY_KEYPAIR>.pem ec2-user@<JUMP_IP>

3. Run schema + seed:
   mysql -h <RDS_ENDPOINT> -u <ADMIN_USER> -p < sql/schema.sql  
   mysql -h <RDS_ENDPOINT> -u <ADMIN_USER> -p myappdb < sql/seed.sql

Tip: one-liner to apply both:
```bash
cat sql/schema.sql sql/seed.sql | mysql -h <RDS_ENDPOINT> -u <ADMIN_USER> -p
```

---

🧾 SQL: Full schema (also in sql/schema.sql)
```sql
-- schema.sql
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

---

🪴 SQL: Full seed data (also in sql/seed.sql)
```sql
-- seed.sql
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

🔍 Verification queries (copy/paste)
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

🛟 Backups & snapshots
- Logical backup:
  mysqldump -h <RDS_ENDPOINT> -u <ADMIN_USER> -p --databases myappdb > myappdb-backup-$(date +%F).sql
- Restore:
  mysql -h <RDS_ENDPOINT> -u <ADMIN_USER> -p < myappdb-backup-YYYY-MM-DD.sql
- Manual RDS snapshot (CLI):
  aws rds create-db-snapshot --db-instance-identifier my-mysql-db --db-snapshot-identifier my-mysql-db-snapshot-$(date +%F)
