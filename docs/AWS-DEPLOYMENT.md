# AWS Deployment Guide

This document describes the deployment procedure for the BookNest Flask application using the architecture documented in the repository README.

> Replace every placeholder such as `<AWS_REGION>`, `<EC2_PUBLIC_IP>`, `<SSH_KEY_PATH>`, and `<ALB_DNS_NAME>` with values from your own environment. Never place credentials or private key contents in this document.

## 1. AWS Region

The documented deployment used:

```text
ap-south-1 (Mumbai)
```

If reproducing in another region, use two Availability Zones available in that region and update the documentation.

## 2. Create the VPC

AWS Console:

```text
VPC -> Your VPCs -> Create VPC
```

Configuration:

```text
Name: BookNest-VPC
IPv4 CIDR: 10.0.0.0/16
```

## 3. Create Internet Gateway

Create:

```text
BookNest-IGW
```

Attach it to:

```text
BookNest-VPC
```

## 4. Create Public Subnet A

```text
Name: BookNest-Public-Subnet-A
CIDR: 10.0.1.0/24
Availability Zone: ap-south-1a
```

Enable automatic public IPv4 assignment.

## 5. Create Public Subnet B

```text
Name: BookNest-Public-Subnet-B
CIDR: 10.0.2.0/24
Availability Zone: ap-south-1b
```

Enable automatic public IPv4 assignment.

## 6. Create Public Route Table

Create:

```text
BookNest-Public-RT
```

Associate it with both public subnets.

Add:

```text
Destination: 0.0.0.0/0
Target: BookNest-IGW
```

## 7. Create EC2 Security Group

Create:

```text
BookNest-EC2-SG
```

Initial inbound rule:

```text
SSH TCP 22 -> My IP
```

Do not expose HTTP to the entire Internet.

After the ALB security group exists, add:

```text
HTTP TCP 80 -> BookNest-ALB-SG
```

## 8. Launch EC2

Launch an Ubuntu Server 24.04 LTS instance.

Use:

```text
Name: BookNest-Server
VPC: BookNest-VPC
Subnet: BookNest-Public-Subnet-A
Public IPv4: Enabled
Security Group: BookNest-EC2-SG
```

Use an appropriate free-tier/credit-eligible instance type for your account and verify the current AWS terms before launch.

## 9. Connect with SSH

From your local machine:

```bash
ssh -i "<SSH_KEY_PATH>" ubuntu@<EC2_PUBLIC_IP>
```

Never commit `<SSH_KEY_PATH>` or the private key to GitHub.

## 10. Install Server Packages

On EC2:

```bash
sudo apt update
sudo apt install -y python3-venv python3-pip nginx git
```

Verify:

```bash
git --version
nginx -v
python3 --version
python3 -m pip --version
```

## 11. Clone the Repository

On EC2:

```bash
mkdir -p ~/BookNest
cd ~/BookNest
git clone https://github.com/<YOUR_GITHUB_USERNAME>/<YOUR_REPOSITORY>.git
```

The commands below assume the repository directory is:

```text
~/BookNest/<YOUR_REPOSITORY>
```

## 12. Create the Virtual Environment

```bash
python3 -m venv ~/BookNest/.venv
source ~/BookNest/.venv/bin/activate
```

Install requirements:

```bash
cd ~/BookNest/<YOUR_REPOSITORY>
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## 13. Restore the Application Database

The application stores PDF data in SQLite. The production database should therefore be transferred securely to the server rather than committed to Git.

From the local machine, copy the database:

```powershell
scp -i "<SSH_KEY_PATH>" ".\database.db" ubuntu@<EC2_PUBLIC_IP>:/home/ubuntu/BookNest/<YOUR_REPOSITORY>/database.db
```

On EC2:

```bash
cd ~/BookNest/<YOUR_REPOSITORY>
ls -lh database.db
file database.db
```

Expected type:

```text
SQLite 3.x database
```

## 14. Generate a Production Secret

On EC2:

```bash
python3 -c "import secrets; print(secrets.token_hex(32))"
```

Save the generated value securely. Do not publish it.

## 15. Create Gunicorn systemd Service

Create:

```bash
sudo nano /etc/systemd/system/booknest.service
```

Use:

```ini
[Unit]
Description=BookNest Flask Application
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/BookNest/<YOUR_REPOSITORY>
Environment="SECRET_KEY=<GENERATED_SECRET>"
ExecStart=/home/ubuntu/BookNest/.venv/bin/gunicorn --workers 1 --bind 127.0.0.1:8000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl enable booknest
sudo systemctl start booknest
sudo systemctl status booknest
```

Test:

```bash
curl http://127.0.0.1:8000/
```

## 16. Configure Nginx

Create:

```bash
sudo nano /etc/nginx/sites-available/booknest
```

Use:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:8000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable it:

```bash
sudo ln -s /etc/nginx/sites-available/booknest /etc/nginx/sites-enabled/booknest
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl restart nginx
sudo systemctl status nginx
```

Test locally:

```bash
curl http://127.0.0.1
```

## 17. Create ALB Security Group

Create:

```text
BookNest-ALB-SG
```

Inbound:

```text
HTTP TCP 80 -> 0.0.0.0/0
```

Outbound:

```text
All traffic
```

## 18. Update EC2 Security Group

Update `BookNest-EC2-SG`:

```text
HTTP TCP 80 -> BookNest-ALB-SG
SSH TCP 22 -> administrator's IP
```

Do not use `0.0.0.0/0` for the EC2 HTTP rule.

## 19. Create Target Group

Create:

```text
Name: BookNest-TG
Target type: Instances
Protocol: HTTP
Port: 80
VPC: BookNest-VPC
Health check protocol: HTTP
Health check path: /
```

Register:

```text
BookNest-Server
Port 80
```

Wait for the target to become:

```text
healthy
```

## 20. Create Application Load Balancer

Create an:

```text
Application Load Balancer
```

Configuration:

```text
Name: BookNest-ALB
Scheme: Internet-facing
IP address type: IPv4
VPC: BookNest-VPC
Subnets:
  BookNest-Public-Subnet-A
  BookNest-Public-Subnet-B
Security Group:
  BookNest-ALB-SG
Listener:
  HTTP :80
Default action:
  Forward to BookNest-TG
```

## 21. Verify

Open:

```text
http://<ALB_DNS_NAME>
```

Verify:

- Login page loads.
- User login/session works.
- Semester pages load.
- PDF download works.
- Admin workflow works.

## 22. Operational Verification

EC2:

```bash
sudo systemctl status booknest
sudo systemctl status nginx
```

Gunicorn logs:

```bash
sudo journalctl -u booknest --no-pager
```

Nginx logs:

```bash
sudo ls -lah /var/log/nginx/
```

Target group:

```text
Target -> healthy
```

## 23. Cleanup

When the deployment is no longer required:

1. Delete the ALB.
2. Delete the target group.
3. Terminate the EC2 instance.
4. Delete the ALB security group.
5. Delete the EC2 security group.
6. Delete both public subnets.
7. Delete the public route table.
8. Detach the Internet Gateway.
9. Delete the Internet Gateway.
10. Delete the VPC.
11. Review AWS Billing/Cost Management.

The exact AWS console labels can change; verify the current console before deletion.
