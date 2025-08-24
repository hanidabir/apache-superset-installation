## Installing Apache Superset on Ubuntu 24.04 (Practically Tested Guide)

### Prerequisites
- Ubuntu 24.04 with sudo access
- Python 3.10 installed  
- Open firewall ports for Superset web and SSH access (default: 8088, custom: 2180/21443, SSH)
- Reliable internet connection (or alternative PyPI mirror access)

***

### 1. Update & Install System Dependencies

**Main way:**
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y python3.10 python3.10-venv python3.10-dev build-essential libssl-dev libffi-dev libsasl2-dev libldap2-dev
```

***

### 2. Create a Python Virtual Environment

```bash
python3.10 -m venv superset-env
source superset-env/bin/activate
```

***

### 3. Upgrade pip

**Main way:**
```bash
pip install --upgrade pip
```
**If you encounter SSL or connection errors (commonly due to sanctions, firewall, or local filtering):**
```bash
pip install --upgrade pip --trusted-host pypi.org --trusted-host files.pythonhosted.org
# OR use a PyPI mirror (example: TUNA mirror, China)
pip install --upgrade pip --index-url https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
```

***

### 4. Install Apache Superset

**Main way:**
```bash
pip install apache-superset
```
**If you see SSL or download errors:**
```bash
pip install apache-superset --trusted-host pypi.org --trusted-host files.pythonhosted.org
# OR with a mirror
pip install apache-superset --index-url https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
```

***

### 5. Fix marshmallow Compatibility (If You Get an Error)

If you see errors about marshmallow version compatibility, run:

```bash
pip uninstall marshmallow
pip install marshmallow==3.19.0 --trusted-host pypi.org --trusted-host files.pythonhosted.org
# OR with a mirror
pip install marshmallow==3.19.0 --index-url https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
```

***

### 6. Generate and Add a SECRET_KEY

Generate a strong secret key:
```bash
openssl rand -base64 42
```

Create a `superset_config.py` file in your home directory (or project root) and add:
```python
SECRET_KEY = "YOUR_GENERATED_SECRET_KEY"
```

Set the config path so Superset can find it:
```bash
export SUPERSET_CONFIG_PATH=/home/USER/superset_config.py
```
(Change the path if necessary.)

***

### 7. Set the FLASK_APP Variable  
(Do this in any active shell before running Superset commands)

```bash
export FLASK_APP=superset
```

***

### 8. Initialize the Superset Database

```bash
superset db upgrade
```

***

### 9. Create an Admin User

```bash
superset fab create-admin
```
Enter the username, email, and password as prompted.

***

### 10. Final Superset Initialization

```bash
superset init
```

***

### 11. Run Superset for Testing

**Default (test) mode:**
```bash
superset run -p 8088 --host 0.0.0.0
```
**Or on a custom port, per your admin/network policy:**
```bash
superset run -p 2180 --host 0.0.0.0
```

Open the desired port on your firewall if needed.

***

### 12. Firewall and Network

Ask your server admin to open the default Superset web (or any custom port you want) TCP port for access: 8088

***

### 13. (Optional) Running as a systemd Service

Create `/etc/systemd/system/superset.service` with the following content (adjust paths and user as necessary):

```ini
[Unit]
Description=Apache Superset
After=network.target

[Service]
User=YOUR_USER
Group=YOUR_USER
WorkingDirectory=/home/YOUR_USER
Environment="PATH=/home/YOUR_USER/superset-env/bin"
Environment="FLASK_APP=superset"
Environment="SUPERSET_CONFIG_PATH=/home/YOUR_USER/superset_config.py"
ExecStart=/home/YOUR_USER/superset-env/bin/superset run -p 8088 --host 0.0.0.0

[Install]
WantedBy=multi-user.target
```

Then run:

```bash
sudo systemctl daemon-reload
sudo systemctl enable superset
sudo systemctl start superset
sudo systemctl status superset
```

***

**Note:**  
For production use, run Superset behind a proper WSGI server like gunicorn and (optionally) a reverse proxy such as Nginx/Apache, especially if you require HTTPS.  
If you need a sample for SSL or production deployment, just ask!

***

**This document is based on real-world troubleshooting and will help you go from zero to a working Superset install -- even under PyPI connectivity limitations or networking restrictions.**
