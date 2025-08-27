# 📘 README — Papilio Cron Tasks via Apache + Bash CGI

This document explains how to expose your cron tasks as HTTP-triggerable Bash scripts using Apache CGI.

---

## 📂 Directory Structure

We will place all bash scripts in:

```
/app/cron-scripts/
```

Apache will expose them at URLs like:

```
http://uat-server.papilio.co.in/cron/<scriptname>.sh
```

---

## ✅ Apache Configuration (`/etc/apache2/sites-available/papilio.conf`)

```apache
<VirtualHost *:80>
    ServerName uat-server.papilio.co.in
    ServerAlias *.papilio.co.in

    # Rails App
    DocumentRoot /app/public
    PassengerRuby /usr/local/bin/ruby
    RailsEnv production
    PassengerAppRoot /app
    PassengerEnabled on

    RedirectMatch 302 ^/$ /v2/ui/login

    ErrorLog /app/log/apache-error.log
    CustomLog /app/log/apache-access.log combined

    <Directory /app/public>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    <Directory /app>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    Alias /v2 /app/public
    <Location /v2>
        PassengerBaseURI /v2
        PassengerAppRoot /app
        PassengerRuby /usr/local/bin/ruby
    </Location>

    Alias /v2/assets /app/public/v2/assets
    <Directory /app/public/v2/assets>
        Options FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    Alias /v2/ui /app/public/v2/ui
    <Directory /app/public/v2/ui>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    XSendFile On
    XSendFilePath /app/tmp/download_zip
    XSendFilePath /app/tmp/preview-cache
    XSendFilePath /app/public

    ###################################################################
    # ✅ CGI for Cron Bash Scripts
    ###################################################################
    ScriptAlias /cron/ "/app/cron-scripts/"
    <Directory "/app/cron-scripts">
        Options +ExecCGI
        AddHandler cgi-script .sh
        Require ip 203.0.113.0/24   # 🔒 Restrict to your allowed CIDR
    </Directory>
</VirtualHost>
```

> 💡 Replace `203.0.113.0/24` with your actual IP/CIDR allowed to run the scripts.

---

## 📝 Bash Scripts (`/app/cron-scripts/*.sh`)

Each script:

* Starts with `#!/bin/bash`
* Echoes proper CGI headers
* Sets up environment
* Executes your Rails/Rake tasks
* Must be executable (`chmod +x`)

---

### 1️⃣ `update_all.sh`

```bash
#!/bin/bash
echo "Content-type: text/plain"
echo ""
cd /home/papilio2user/papilio_v2/server || exit 1
export RAILS_ENV=production
/home/papilio2user/.rbenv/shims/bundle exec rails admin:update:all
```

### 2️⃣ `user_weekly.sh`

```bash
#!/bin/bash
echo "Content-type: text/plain"
echo ""
cd /home/papilio2user/papilio_v2/server || exit 1
export RAILS_ENV=production
/home/papilio2user/.rbenv/shims/bundle exec rails metrics:user_weekly
```

### 3️⃣ `client_monthly.sh`

```bash
#!/bin/bash
echo "Content-type: text/plain"
echo ""
cd /home/papilio2user/papilio_v2/server || exit 1
export RAILS_ENV=production
/home/papilio2user/.rbenv/shims/bundle exec rails metrics:client_monthly
```

### 4️⃣ `project_recur.sh`

```bash
#!/bin/bash
echo "Content-type: text/plain"
echo ""
cd /home/papilio2user/papilio_v2/server || exit 1
export RAILS_ENV=production
/home/papilio2user/.rbenv/shims/bundle exec rails admin:project_recur
```

### 5️⃣ `weekly_digest.sh`

```bash
#!/bin/bash
echo "Content-type: text/plain"
echo ""
cd /home/papilio2user/papilio_v2/server || exit 1
export RAILS_ENV=production
/home/papilio2user/.rbenv/shims/bundle exec rake mail:deliver:weekly_digest
```

### 6️⃣ `send_nth_day.sh`

```bash
#!/bin/bash
echo "Content-type: text/plain"
echo ""
cd /home/papilio2user/papilio_v2/server || exit 1
export RAILS_ENV=production
/home/papilio2user/.rbenv/shims/bundle exec rake mail:deliver:send_nth_day
```

### 7️⃣ `timesheet_reminder.sh`

```bash
#!/bin/bash
echo "Content-type: text/plain"
echo ""
cd /home/papilio2user/papilio_v2/server || exit 1
export RAILS_ENV=production
/home/papilio2user/.rbenv/shims/bundle exec rake mail:deliver:timesheet_reminder
```

### 8️⃣ `send_reminders.sh`

```bash
#!/bin/bash
echo "Content-type: text/plain"
echo ""
cd /home/papilio2user/papilio_v2/mulberry || exit 1
export RAILS_ENV=production
/home/papilio2user/.rbenv/shims/bundle exec rake admin:send_reminders
```

### 9️⃣ `expiry_reminder.sh`

```bash
#!/bin/bash
echo "Content-type: text/plain"
echo ""
cd /home/papilio2user/papilio_v2/server || exit 1
export RAILS_ENV=production
/home/papilio2user/.rbenv/shims/bundle exec rake mail:deliver:expiry_reminder
```

### 🔟 `timesheet_weekly_digest.sh`

```bash
#!/bin/bash
echo "Content-type: text/plain"
echo ""
cd /home/papilio2user/papilio_v2/server || exit 1
export RAILS_ENV=production
/home/papilio2user/.rbenv/shims/bundle exec rails timesheet:weekly_digest
```

### 1️⃣1️⃣ `create_invoices.sh`

```bash
#!/bin/bash
echo "Content-type: text/plain"
echo ""
cd /home/papilio2user/papilio_v2/server || exit 1
export RAILS_ENV=production
/home/papilio2user/.rbenv/shims/bundle exec rails billing:create_invoices
```

### 1️⃣2️⃣ `set_object_tag.sh`

```bash
#!/bin/bash
echo "Content-type: text/plain"
echo ""
/home/papilio2user/scripts/set_object_tag.sh
```

### 1️⃣3️⃣ `del_download_zip.sh`

```bash
#!/bin/bash
echo "Content-type: text/plain"
echo ""
/root/scripts/del-download-zip.sh
```

### 1️⃣4️⃣ `sync_time.sh`

```bash
#!/bin/bash
echo "Content-type: text/plain"
echo ""
/usr/sbin/ntpdate 0.amazon.pool.ntp.org
```

---

## ⚙️ Setup Steps

1️⃣ **Create script directory**:

```bash
mkdir -p /app/cron-scripts
```

2️⃣ **Place each `.sh` file above into this folder**.

3️⃣ **Make them executable**:

```bash
chmod +x /app/cron-scripts/*.sh
```

4️⃣ **Enable CGI module in Apache** (if not already):

```bash
a2enmod cgi
```

5️⃣ **Reload Apache**:

```bash
service apache2 restart
```

---

## 🧪 Testing Steps

To run a task manually via browser or curl:

```
curl http://uat-server.papilio.co.in/cron/update_all.sh
```

You should see console output from the script.

✅ If IP not in CIDR: you’ll get `403 Forbidden`.

✅ If script runs: output will appear on screen.

✅ Apache logs:

* Access log: `/app/log/apache-access.log`
* Error log: `/app/log/apache-error.log`

---

## ✅ Security Notes

* Restrict access using `Require ip` CIDR.
* Scripts are plain bash, avoid exposing sensitive logic.
* Never allow `Require all granted` unless testing only.

---

## 🎯 Summary

This setup allows you to:

* Run any cron task manually over HTTP (e.g., after deployments or debugging)
* Keep the same logic used by `cron`, now available on-demand
* Secure it with IP restrictions
