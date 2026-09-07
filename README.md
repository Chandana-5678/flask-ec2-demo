# Flask AWS DevOps Project

Stage 1 starts a small Flask application locally and prepares it for deployment to Ubuntu EC2.

## Local run

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
py -m pip install -r requirements.txt
py app.py
```

Open http://127.0.0.1:5000/ or http://127.0.0.1:5000/health.
