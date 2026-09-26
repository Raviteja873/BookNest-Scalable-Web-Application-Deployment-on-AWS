# GitHub Publication — Exact Steps

Repository:

```text
https://github.com/Raviteja873/BookNest-Cloud-Deployed-Academic-Resource-Platform-on-AWS
```

## 1. Open the existing local project

PowerShell:

```powershell
cd "E:\BookNest-MCA-Academic-Companion-main"
Get-ChildItem
```

Do not delete the existing Flask application.

## 2. Add the documentation package

Copy/merge these folders/files from the prepared documentation package into the existing project:

```text
architecture/
docs/
infrastructure/
scripts/
src/
configuration/examples/
.gitignore
README.md
requirements.txt
```

Keep your actual application files:

```text
app.py
templates/
static/
LICENSE
```

Add your real screenshots under `screenshots/` using `docs/SCREENSHOTS.md`.

## 3. Add the architecture image

Save the final diagram as:

```text
architecture/booknest-aws-architecture.png
```

## 4. Verify Git

```powershell
git status
```

If the local folder is not already a Git repository:

```powershell
git init
git branch -M main
```

## 5. Verify `.gitignore`

It must exclude at least:

```text
database.db
.venv/
.env
*.pem
*.key
.aws/
*.tfstate
*.tfvars
```

## 6. Verify no sensitive files

```powershell
Get-ChildItem -Recurse -Force -File |
  Where-Object { $_.Name -match '^\.env$|\.pem$|\.key$|database\.db|credentials' } |
  Select-Object FullName
```

Do not commit secrets, private keys, AWS credentials, or the production database.

## 7. Stage

```powershell
git add .
git status
git diff --cached --stat
```

Review the staged list before committing.

## 8. Commit

```powershell
git commit -m "feat: publish BookNest AWS cloud project"
```

## 9. Connect remote

If no remote exists:

```powershell
git remote add origin https://github.com/Raviteja873/BookNest-Cloud-Deployed-Academic-Resource-Platform-on-AWS.git
```

Verify:

```powershell
git remote -v
```

## 10. Push

```powershell
git push -u origin main
```

## 11. Verify the published repository

Open:

```text
https://github.com/Raviteja873/BookNest-Cloud-Deployed-Academic-Resource-Platform-on-AWS
```

Check that:

- README renders.
- Architecture diagram displays.
- Screenshots are present.
- `docs/` is present.
- Source code is present.
- `database.db` is absent.
- `.venv/` is absent.
- `.pem` files are absent.
- `.env` files containing secrets are absent.

## 12. If a secret was staged but not committed

```powershell
git restore --staged <FILE>
```

If a secret was already pushed, revoke/rotate it immediately; deleting it in a later commit does not make the old secret safe.

## 13. Final check

```powershell
git status
git log --oneline --max-count=5
git remote -v
```

Expected final status:

```text
nothing to commit, working tree clean
```
