# Source

The Flask application source is currently maintained at the repository root as:

```text
app.py
```

This folder is reserved for future application modularization.

A future refactor could use:

```text
src/
├── __init__.py
├── routes/
├── models/
├── services/
└── templates/
```

Do not change the application module name in deployment documentation unless the Gunicorn command is updated accordingly.
