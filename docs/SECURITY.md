# Security Guidelines

## Secrets that must never be committed

Do not commit:

- AWS access key IDs.
- AWS secret access keys.
- AWS session tokens.
- IAM passwords.
- Application passwords.
- API tokens.
- `SECRET_KEY` values.
- `.env` files containing real secrets.
- SSH private keys such as `.pem`.
- Private certificates and private keys.
- Terraform state files.
- Terraform plan files containing sensitive values.
- Database files containing private application data.

## Repository checks

Before committing:

```bash
git status
git diff --cached
git ls-files
```

Search for common secret files:

```bash
find . -name ".env" -o -name "*.pem" -o -name "*.key" -o -name "database.db"
```

On Windows PowerShell:

```powershell
Get-ChildItem -Recurse -Force -File |
  Where-Object { $_.Name -match '^\.env$|\.pem$|\.key$|^database\.db$' }
```

## AWS credentials

The application does not require AWS access keys to run.

If AWS CLI access is required for future automation, prefer:

- IAM roles for AWS compute resources.
- Short-lived credentials.
- AWS IAM Identity Center where appropriate.
- Least-privilege permissions.

Never hard-code credentials in Python source.

## Application secret

Generate a random Flask secret:

```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

Store it in the runtime environment, not in Git.

## Database

`database.db` contains application data and uploaded PDF content. It is intentionally excluded from Git.

If a sanitized demo database is ever published, verify that it contains no personal, institutional, or confidential information.

## Screenshots

Review every screenshot before publishing. Console screenshots can accidentally reveal:

- Account identifiers.
- Public IP addresses.
- Usernames.
- Resource identifiers.
- Email addresses.
- Secrets.

Only retain information that is useful for explaining the architecture.
