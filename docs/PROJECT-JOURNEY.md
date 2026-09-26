# BookNest Project Journey — Complete Implementation Record

This document records the complete BookNest journey from the Flask application through AWS deployment, verification, documentation, and cleanup.

## 1. Application foundation

- Defined BookNest as an MCA academic resource portal.
- Flask backend with HTML/CSS templates.
- Flask-SQLAlchemy with SQLite.
- Session-based user access.
- Institute email-domain validation.
- Semester-wise subject navigation.
- PDF download workflow.
- Admin-only PDF upload/replacement workflow.
- `Document` model stores semester, subject, filename, PDF `LargeBinary`, MIME type, and update time.
- Local application verified at `http://127.0.0.1:5000`.

## 2. Python/project preparation

- Created `.venv`.
- Added `requirements.txt` containing Flask 3.0.2, Flask-SQLAlchemy 3.1.1, and Gunicorn.
- Added `.gitignore` for `.venv`, `.env`, database files, keys, AWS credentials, Terraform state, logs, IDE files, and generated artifacts.
- Kept `database.db` outside Git because it contains application data and PDF content.

## 3. AWS planning

- Region: Mumbai (`ap-south-1`).
- Cost-conscious design.
- Intentionally did not use NAT Gateway, RDS, S3, CloudFront, WAF, or Auto Scaling in this version.

## 4. VPC/networking

Created:

```text
BookNest-VPC       10.0.0.0/16
BookNest-IGW
BookNest-Public-Subnet-A  10.0.1.0/24  ap-south-1a
BookNest-Public-Subnet-B  10.0.2.0/24  ap-south-1b
BookNest-Public-RT
```

Route table:

```text
10.0.0.0/16 -> local
0.0.0.0/0   -> BookNest-IGW
```

Both public subnets were associated with the route table and configured for public IPv4 assignment.

## 5. EC2 security

Created `BookNest-EC2-SG`.

Initial inbound:

```text
SSH TCP 22 -> administrator's IP
```

Later, after the ALB existed:

```text
HTTP TCP 80 -> BookNest-ALB-SG
```

Outbound remained all traffic.

## 6. EC2 server

Created `BookNest-Server` using Ubuntu Server 24.04 LTS in Public Subnet A.

Installed:

```bash
sudo apt update
sudo apt install -y python3-venv python3-pip nginx git
```

Verified Git, Nginx, Python, and pip.

## 7. Application deployment on EC2

Created the application directory, cloned the repository, created the EC2 virtual environment, and installed requirements.

The application database was securely copied to the server with `scp` and verified with:

```bash
ls -lh database.db
file database.db
```

## 8. Gunicorn

First tested Gunicorn directly:

```bash
gunicorn --bind 127.0.0.1:8000 app:app
```

Verified with:

```bash
curl http://127.0.0.1:8000/
```

Then created `/etc/systemd/system/booknest.service`, using the virtual environment, one Gunicorn worker, `127.0.0.1:8000`, and a runtime `SECRET_KEY`.

Enabled and started it with:

```bash
sudo systemctl daemon-reload
sudo systemctl enable booknest
sudo systemctl start booknest
sudo systemctl status booknest
```

## 9. Nginx

Configured Nginx to listen on port 80 and reverse proxy to Gunicorn on `127.0.0.1:8000`.

Validated and restarted:

```bash
sudo nginx -t
sudo systemctl restart nginx
sudo systemctl status nginx
curl http://127.0.0.1
```

## 10. Application Load Balancer

Created `BookNest-ALB-SG` with public HTTP port 80 inbound.

Created `BookNest-TG`:

```text
Target type: Instances
Protocol: HTTP
Port: 80
Health check: /
```

Registered `BookNest-Server` and verified it became healthy.

Created internet-facing `BookNest-ALB` across both public subnets with an HTTP port 80 listener forwarding to `BookNest-TG`.

## 11. End-to-end validation

Opened the AWS-generated ALB DNS name and tested:

1. Login page.
2. Name/institute email validation.
3. Welcome page.
4. Semester navigation.
5. Subject listing.
6. PDF download.
7. Admin-only management access.
8. PDF upload/replacement.
9. PDF download after upload.
10. Target-group health.

## 12. Documentation evidence

Screenshots were captured for the important milestones: VPC, Internet Gateway, subnets, route table, security groups, EC2, SSH/server environment, dependencies, database restoration, Gunicorn, Nginx, target group health, ALB, live login, application pages, PDF download, and admin upload.

The final architecture diagram belongs in:

```text
architecture/booknest-aws-architecture.png
```

## 13. AWS cleanup

After documentation/testing, the cleanup sequence is:

1. Delete ALB.
2. Delete target group.
3. Terminate EC2.
4. Delete ALB security group.
5. Delete EC2 security group.
6. Delete public subnets.
7. Delete public route table.
8. Detach Internet Gateway.
9. Delete Internet Gateway.
10. Delete VPC.
11. Review Billing/Cost Management.

GitHub and the local project remain independent of AWS resource cleanup.

## 14. Final architecture

```text
Internet users
      |
      v
Internet-facing ALB :80
      |
      v
BookNest Target Group
      |
      v
EC2 :80
      |
      v
Nginx
      |
      v
Gunicorn :8000
      |
      v
Flask
      +---- SQLite database.db
      |       +---- PDF binary data
      +---- templates/static
```

The current project does not claim RDS, S3, CloudFront, WAF, Auto Scaling, Docker, Terraform, CI/CD, or CloudWatch as implemented components. They are future improvements only.
