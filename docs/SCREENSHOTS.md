# Screenshot Documentation Plan

Store project screenshots under:

```text
screenshots/
```

Use lowercase descriptive names with a two-digit sequence.

## AWS Infrastructure

### 01 — VPC

- **Filename:** `01-aws-vpc.png`
- **Capture:** VPC details page.
- **Visible:** `BookNest-VPC`, CIDR `10.0.0.0/16`, region if shown.
- **README:** Architecture / AWS Infrastructure section.

### 02 — Internet Gateway

- **Filename:** `02-internet-gateway.png`
- **Capture:** Internet Gateway details.
- **Visible:** `BookNest-IGW` and attached VPC.
- **README:** Architecture / Networking.

### 03 — Public Subnets

- **Filename:** `03-public-subnets.png`
- **Capture:** VPC subnet list/details.
- **Visible:** both public subnets, CIDRs, and Availability Zones.
- **README:** Architecture / Networking.

### 04 — Route Table

- **Filename:** `04-public-route-table.png`
- **Capture:** `BookNest-Public-RT`.
- **Visible:** `0.0.0.0/0 -> BookNest-IGW` and subnet associations.
- **README:** Architecture / Networking.

### 05 — EC2 Security Group

- **Filename:** `05-ec2-security-group.png`
- **Capture:** `BookNest-EC2-SG`.
- **Visible:** SSH restricted to administrator IP and HTTP from ALB SG.
- **README:** Security section.

### 06 — EC2 Instance

- **Filename:** `06-ec2-instance.png`
- **Capture:** EC2 instance details.
- **Visible:** `BookNest-Server`, Ubuntu, VPC/subnet, running state during deployment.
- **README:** AWS Infrastructure.

### 07 — SSH / Server Environment

- **Filename:** `07-ssh-connected.png`
- **Capture:** terminal connected to EC2.
- **Visible:** hostname/user and non-sensitive version information.
- **Do not show:** private key contents, credentials, secrets.
- **README:** Deployment.

### 08 — Server Tools

- **Filename:** `08-server-tools.png`
- **Capture:** terminal showing installed Git, Nginx, Python.
- **README:** Deployment.

### 09 — Python Dependencies

- **Filename:** `09-python-dependencies.png`
- **Capture:** successful `pip install -r requirements.txt`.
- **README:** Deployment.

### 10 — Database

- **Filename:** `10-database-restored.png`
- **Capture:** `ls -lh database.db` and `file database.db`.
- **Do not expose:** sensitive application records.
- **README:** Deployment/Data layer.

### 11 — Gunicorn

- **Filename:** `11-gunicorn-service.png`
- **Capture:** `sudo systemctl status booknest`.
- **Visible:** active service.
- **README:** Application deployment.

### 12 — Nginx

- **Filename:** `12-nginx-service.png`
- **Capture:** `sudo nginx -t` and/or service status.
- **Visible:** successful configuration.
- **README:** Application deployment.

### 13 — ALB Security Group

- **Filename:** `13-alb-security-group.png`
- **Capture:** `BookNest-ALB-SG`.
- **Visible:** HTTP 80 inbound and outbound rule.
- **README:** Security.

### 14 — Target Group

- **Filename:** `14-target-group-healthy.png`
- **Capture:** `BookNest-TG` targets.
- **Visible:** EC2 target and `healthy` status.
- **README:** Architecture / Verification.

### 15 — ALB

- **Filename:** `15-alb-active.png`
- **Capture:** ALB details.
- **Visible:** internet-facing ALB, listener, subnets, active state.
- **README:** Architecture.

## Application Screenshots

### 16 — Live Login

- **Filename:** `16-live-login.png`
- **Capture:** BookNest login page through the ALB DNS name.
- **Visible:** browser address bar with ALB DNS name and application.
- **README:** Screenshots / Live deployment.

### 17 — Welcome Page

- **Filename:** `17-welcome-page.png`
- **Capture:** authenticated welcome page.
- **README:** Application features.

### 18 — Semester Page

- **Filename:** `18-semester-page.png`
- **Capture:** semester/subject listing.
- **README:** Application features.

### 19 — PDF Download

- **Filename:** `19-pdf-download.png`
- **Capture:** successful PDF resource access/download.
- **README:** Application features.

### 20 — Admin Upload

- **Filename:** `20-admin-upload.png`
- **Capture:** admin management/upload page.
- **README:** Application features.

## Screenshot Safety

Before publishing:

- Blur or crop AWS account IDs if not needed.
- Do not show access keys.
- Do not show secret keys.
- Do not show passwords.
- Do not show private SSH keys.
- Do not show `.env` contents.
- Do not expose sensitive personal information.
- Avoid publishing unnecessary public IP information if it is no longer relevant.

## README Image Convention

Use relative Markdown paths:

```markdown
![VPC](screenshots/01-aws-vpc.png)
```

For a large screenshot collection, keep the README concise and link to this document rather than embedding every screenshot at full size.
