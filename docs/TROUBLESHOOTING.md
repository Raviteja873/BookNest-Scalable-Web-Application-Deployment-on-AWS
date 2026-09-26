# Troubleshooting

## 1. `database.db` is missing

Check:

```bash
ls -lh database.db
file database.db
```

If it is absent, transfer the correct local database file securely with `scp`.

Do not download or copy it from an untrusted source.

## 2. Gunicorn service is not running

```bash
sudo systemctl status booknest
sudo journalctl -u booknest -n 100 --no-pager
```

Check:

- Working directory.
- Virtual environment path.
- `app:app` module reference.
- Python dependencies.
- `SECRET_KEY`.

## 3. Gunicorn works but Nginx returns 502

Test Gunicorn:

```bash
curl http://127.0.0.1:8000/
```

Test Nginx:

```bash
curl http://127.0.0.1/
```

If Gunicorn works but Nginx fails:

```bash
sudo nginx -t
sudo systemctl status nginx
sudo tail -n 100 /var/log/nginx/error.log
```

## 4. ALB target is unhealthy

Verify:

```text
ALB -> Target Group -> EC2:80
```

EC2 security group must allow:

```text
HTTP 80 from BookNest-ALB-SG
```

The target group health check must use:

```text
HTTP
Port 80
Path /
```

Also verify Nginx:

```bash
sudo systemctl status nginx
curl http://127.0.0.1/
```

## 5. Application loads but PDF is missing

Check the database:

```bash
ls -lh database.db
```

Confirm the application is using the expected database path.

The current Flask code builds the path relative to the application directory.

## 6. Admin page redirects

The current application checks the session email against the configured administrator email.

Verify that the login email matches the configured administrator address exactly after normalization.

## 7. SSH stops working

If SSH was previously restricted to the administrator's public IP, verify that the current public IP has not changed.

Do not solve the problem by permanently opening SSH to `0.0.0.0/0`.

## 8. High resource usage

Check:

```bash
free -h
df -h
top
```

Check Gunicorn:

```bash
sudo systemctl status booknest
```

The documented service intentionally uses one Gunicorn worker to keep the small EC2 workload lightweight.
