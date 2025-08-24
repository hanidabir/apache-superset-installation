## راهنمای نصب Apache Superset روی Ubuntu 24.04 (عملی و تست‌شده)

### پیش‌نیازها
- نسخه Ubuntu 24.04 با دسترسی sudo  
- نصب Python 3.10  
- باز بودن پورتهای لازم روی فایروال (پیشفرض: 8088)
- اینترنت پایدار یا دسترسی به آیینه‌های pip

***

### 1. به‌روزرسانی سیستم و نصب وابستگی‌ها

**راه اصلی:**
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y python3.10 python3.10-venv python3.10-dev build-essential libssl-dev libffi-dev libsasl2-dev libldap2-dev
```

***

### 2. ساخت محیط مجازی (virtual environment)

```bash
python3.10 -m venv superset-env
source superset-env/bin/activate
```

***

### 3. به‌روزرسانی pip

**راه اصلی:**
```bash
pip install --upgrade pip
```
**اگر خطای SSL یا تحریم داشتید:**
```bash
pip install --upgrade pip --trusted-host pypi.org --trusted-host files.pythonhosted.org
# یا با آیینه TUNA چین
pip install --upgrade pip --index-url https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
```

***

### 4. نصب Apache Superset

**راه اصلی:**
```bash
pip install apache-superset
```
**اگر مشکل SSL یا دانلود داشتید:**
```bash
pip install apache-superset --trusted-host pypi.org --trusted-host files.pythonhosted.org
# یا با آیینه TUNA چین
pip install apache-superset --index-url https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
```

***

### 5. رفع ناسازگاری marshmallow (در صورت وجود خطا)

اگر در این مرحله marshmallow خطا داد، دستور زیر را اجرا کنید:

```bash
pip uninstall marshmallow
pip install marshmallow==3.19.0 --trusted-host pypi.org --trusted-host files.pythonhosted.org
# یا با آیینه TUNA چین
pip install marshmallow==3.19.0 --index-url https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
```

***

### 6. ساخت کلید مخفی (SECRET_KEY) و پیکربندی

یک کلید قوی بسازید:
```bash
openssl rand -base64 42
```

یک فایل به نام `superset_config.py` در شاخه home یا پروژه بسازید و مقدار زیر را قرار دهید (کلید تولید شده را جایگزین کنید):

```python
SECRET_KEY = "کلید_تولیدی_شده"
```

سپس فایل پیکربندی را به Superset معرفی کنید:
```bash
export SUPERSET_CONFIG_PATH=/home/USER/superset_config.py
```
(مسیر دقیق فایل خود را قرار دهید.)

***

### 7. ست کردن متغیر محیطی FLASK_APP  
(در هر ترمینال قبل از اجرای دستورات Superset)

```bash
export FLASK_APP=superset
```

***

### 8. راهاندازی دیتابیس Superset

```bash
superset db upgrade
```

***

### 9. ساخت حساب کاربری ادمین

```bash
superset fab create-admin
```
نام کاربری، ایمیل و رمز را وارد کنید.

***

### 10. راه‌اندازی اولیه Superset

```bash
superset init
```

***

### 11. اجرای Superset برای تست

**روش پیش‌فرض (تست):**
```bash
superset run -p 8088 --host 0.0.0.0
```
**یا روی پورتی که ادمین در اختیارتان گذاشته:**
```bash
superset run -p 2180 --host 0.0.0.0
```

حتماً پورت موردنظر را در فایروال باز کنید.

***

### 12. فایروال و شبکه

از ادمین بخواهید پورت پیش‌فرض آپاچی سوپرست (یا هر پورت دلخواهی که می‌خواهید روی آن راه‌اندازی کنید) را باز کند: 8088

***

### 13. (اختیاری) راهاندازی Superset به عنوان سرویس systemd

فایل `/etc/systemd/system/superset.service` را با محتوای زیر (آدرس‌ها و یوزر را صحیح بگذارید) بسازید:

```ini
[Unit]
Description=Apache Superset
After=network.target

[Service]
User=USER
Group=USER
WorkingDirectory=/home/USER
Environment="PATH=/home/USER/superset-env/bin"
Environment="FLASK_APP=superset"
Environment="SUPERSET_CONFIG_PATH=/home/USER/superset_config.py"
ExecStart=/home/USER/superset-env/bin/superset run -p 8088 --host 0.0.0.0

[Install]
WantedBy=multi-user.target
```

سپس:
```bash
sudo systemctl daemon-reload
sudo systemctl enable superset
sudo systemctl start superset
sudo systemctl status superset
```

***

**توجه:**  
برای محیط عملیاتی (production)، توصیه می‌شود Superset را پشت یک reverse-proxy مثل nginx یا apache و با WSGI سرور (مثل gunicorn) اجرا کنید و برای SSL از reverse-proxy استفاده شود.
