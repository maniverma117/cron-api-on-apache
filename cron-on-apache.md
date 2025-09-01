# Apache CGI with Bash Script

This guide explains how to run a Bash script as a CGI program on Apache and access it over HTTP.

---

## 1. Enable CGI module
```bash
sudo a2enmod cgi
sudo systemctl restart apache2
````

Verify:

```bash
apache2ctl -M | grep cgi
```

---

## 2. Configure Apache VirtualHost

Edit your Apache site config (e.g. `/etc/apache2/sites-available/000-default.conf`) and add:

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    # Enable CGI for /cgi-bin/
    ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/

    <Directory "/usr/lib/cgi-bin">
        AllowOverride None
        Options +ExecCGI
        AddHandler cgi-script .cgi .sh
        Require all granted
    </Directory>
</VirtualHost>
```

Reload Apache:

```bash
sudo systemctl reload apache2
```

---

## 3. Create a Bash CGI Script

Create `/usr/lib/cgi-bin/test.sh`:

```bash
#!/bin/bash
echo "Content-type: text/html"
echo ""
echo "<html><body>"
echo "<h2>Hello from Bash CGI!</h2>"
echo "<pre>"
date
echo "You hit this script via Apache CGI"
echo "</pre>"
echo "</body></html>"
```

Make it executable:

```bash
sudo chmod 755 /usr/lib/cgi-bin/test.sh
```

---

## 4. Access in Browser

Open:

```
http://<server-ip>/cgi-bin/test.sh
```

You should see your HTML output.

---

## 5. Passing Parameters (GET & POST)

* **GET example:**

  ```
  http://<server-ip>/cgi-bin/test.sh?name=Mani
  ```

  Inside the script, access it via:

  ```bash
  echo "Query string: $QUERY_STRING"
  ```

* **POST example:**

  ```bash
  curl -X POST -d "key=value" http://<server-ip>/cgi-bin/test.sh
  ```

  Inside the script, read body:

  ```bash
  read POST_DATA
  echo "Post data: $POST_DATA"
  ```

---

## 6. Logs

Check Apache error logs if something goes wrong:

```bash
sudo tail -f /var/log/apache2/error.log
```

---

✅ Now you have a working Bash CGI setup on Apache!
