# **🚀 Deployment Guide: HESK Helpdesk on External Hosting**  
**🌐 Domain:** `help.zimin.dev` (Cloudflare-protected)  
**🛡️ Cloudflare Role:** DDoS protection only (proxy disabled for backend)  

---

## **📦 Server Preparation**  
### **1️⃣ Create MySQL Database**  
```sql
-- Via hosting panel (cPanel/Plesk) or manually:
CREATE DATABASE hesk_db;
CREATE USER 'hesk_user'@'localhost' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON hesk_db.* TO 'hesk_user'@'localhost';
FLUSH PRIVILEGES;
```

### **2️⃣ Upload HESK Files**  
```bash
# Using SFTP/rsync:
rsync -avz ./hesk/ user@server:/var/www/help.zimin.dev/public_html/
# Or via hosting file manager
```

### **3️⃣ Configure Cloudflare DNS**  
| **Type** | **Name**       | **Value**               | **Proxy** |  
|----------|----------------|-------------------------|-----------|  
| `A`      | `help.zimin.dev` | `your.server.ip`        | 🟠 Orange |  
| `CNAME`  | `www`           | `help.zimin.dev`        | 🟠 Orange |  

> ⚠️ **Disable Proxy** (grey cloud) if backend uses non-standard ports.

---

## **⚙️ HESK Installation**  
1. Access `https://help.zimin.dev/install/install.php`  
2. Enter database details:  
   ```ini
   Host: localhost  
   Database: hesk_db  
   Username: hesk_user  
   Password: StrongPassword123!  
   Table Prefix: hesk_  
   ```  
3. Complete setup → **Delete `/install` folder**  

---

## **🔐 Security Hardening**  
### **1️⃣ File Permissions**  
```bash
chmod 644 hesk_settings.inc.php
chmod 755 uploads/
find . -type d -exec chmod 755 {} \;
find . -type f -exec chmod 644 {} \;
```

### **2️⃣ Cloudflare Firewall Rules**  
```text
1. Block SQLi/XSS:  
   `(http.request.uri.path contains "select") or (http.request.uri.query contains "<script>")`  

2. Rate-limiting:  
   `15 requests/minute per IP to /admin/`  
```

---

## **📊 Post-Deployment Checklist**  
| **Task**                      | **Command/Check**                  |  
|-------------------------------|------------------------------------|  
| Database connection test      | `mysql -u hesk_user -p hesk_db`    |  
| Cron jobs setup               | `*/5 * * * * php /path/to/hesk/email/parser.php` |  
| Backup automation             | `mysqldump -u hesk_user -p hesk_db > backup.sql` |  

---

## **❗ Troubleshooting**  
### **Issue: 502 Bad Gateway**  
**Fix:**  
```nginx
# Nginx config snippet:
location / {
    proxy_pass http://localhost:8080;
    proxy_set_header Host $host;
}
```

### **Issue: Mixed Content Warnings**  
```php
// In hesk_settings.inc.php:
$hesk_settings['force_ssl'] = 1;
```

---

## **🌐 Recommended Stack**  
- **Web Server:** Nginx + PHP-FPM 8.2  
- **Database:** MySQL 8.0 (InnoDB)  
- **Caching:** Redis for sessions  

**🔄 Update Strategy:**  
```bash
rsync --exclude='hesk_settings.inc.php' -avz ./hesk-3.4.5/ user@server:/path/to/helpdesk/
```
