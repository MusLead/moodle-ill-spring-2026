# Moodle Installation – Step-by-Step Setup

This document explains how Moodle was installed for the **EU Railways Academy** prototype.

The focus is on the Moodle software setup itself, not on the server provider, AWS, Raspberry Pi, or network configuration.

---

## 1. Choose a Compatible Moodle Version

The first important decision was choosing the Moodle version.

For this project, the version choice was important because some plugins were not yet compatible with newer Moodle or PHP versions.

The stable setup used for this project was based on:

```text
Moodle: Moodle 5.0.x
Web server: Apache2
Database: MariaDB / MySQL
PHP: Version compatible with Moodle 5.0.x
Operating system: Ubuntu 24.04 LTS
````

The main lesson was:

> Do not install the newest Moodle version automatically.
> First check whether the required plugins support that Moodle version.

---

## 2. Install Required Server Packages

Moodle requires a web server, database server, PHP, and several PHP extensions.

Install Apache, MariaDB, PHP, and required PHP modules:

```bash
sudo apt update
sudo apt install apache2 mariadb-server php php-cli php-fpm php-mysql php-xml php-mbstring php-curl php-zip php-gd php-intl php-soap php-xmlrpc php-bcmath php-ldap unzip git -y
```

Check the installed PHP version:

```bash
php -v
```

Check Apache:

```bash
apache2 -v
```

Check MariaDB:

```bash
mysql --version
```

---

## 3. Create the Moodle Database

Log in to MariaDB:

```bash
sudo mysql
```

Create a database for Moodle:

```sql
CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Create a database user:

```sql
CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'strong_password_here';
```

Give the user access to the Moodle database:

```sql
GRANT ALL PRIVILEGES ON moodle.* TO 'moodleuser'@'localhost';
```

Apply the changes:

```sql
FLUSH PRIVILEGES;
EXIT;
```

---

## 4. Download Moodle

Go to the web root directory:

```bash
cd /var/www
```

Download Moodle from Git:

```bash
sudo git clone https://github.com/moodle/moodle.git
```

Enter the Moodle directory:

```bash
cd moodle
```

Check available branches:

```bash
sudo git branch -a
```

Switch to the Moodle 5.0 stable branch:

```bash
sudo git checkout MOODLE_500_STABLE
```

---

## 5. Create the Moodle Data Directory

Moodle needs a separate data directory.

This directory must not be directly accessible from the browser.

Create the directory:

```bash
sudo mkdir /var/www/moodledata
```

Set the correct owner:

```bash
sudo chown -R www-data:www-data /var/www/moodledata
```

Set permissions:

```bash
sudo chmod -R 770 /var/www/moodledata
```

---

## 6. Set Moodle Folder Permissions

Set Apache as the owner of the Moodle folder:

```bash
sudo chown -R www-data:www-data /var/www/moodle
```

Set safe permissions:

```bash
sudo find /var/www/moodle -type d -exec chmod 755 {} \;
sudo find /var/www/moodle -type f -exec chmod 644 {} \;
```

---

## 7. Configure Apache for Moodle

Create an Apache configuration file:

```bash
sudo nano /etc/apache2/sites-available/moodle.conf
```

Example configuration:

```apache
<VirtualHost *:80>
    ServerName your-domain-or-ip
    DocumentRoot /var/www/moodle

    <Directory /var/www/moodle>
        Options FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/moodle_error.log
    CustomLog ${APACHE_LOG_DIR}/moodle_access.log combined
</VirtualHost>
```

Enable the Moodle site:

```bash
sudo a2ensite moodle.conf
```

Disable the default Apache page if needed:

```bash
sudo a2dissite 000-default.conf
```

Enable required Apache modules:

```bash
sudo a2enmod rewrite
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

---

## 8. Open Moodle in the Browser

Open the Moodle website in the browser:

```text
http://your-domain-or-ip
```

The Moodle web installer should appear.

Follow the installation steps in the browser.

---

## 9. Moodle Web Installer Settings

During the Moodle web installation, use the following information:

### Moodle directory

```text
/var/www/moodle
```

### Moodle data directory

```text
/var/www/moodledata
```

### Database type

```text
MariaDB / MySQL
```

### Database name

```text
moodle
```

### Database user

```text
moodleuser
```

### Database password

```text
strong_password_here
```

### Database host

```text
localhost
```

After this, Moodle checks the server environment.

If any PHP extension is missing, install it and restart Apache.

Example:

```bash
sudo apt install php-intl
sudo systemctl restart apache2
```

---

## 10. Complete Moodle Installation

After the environment check is successful, Moodle creates its database tables.

Then create the Moodle administrator account.

Example information:

```text
Username: admin
Password: strong_admin_password
Email: admin@example.com
```

After that, configure the basic site settings.

For this project, the site was created as:

```text
Site name: EU Railways Academy
```

---

## 11. Configure Moodle Cron

Moodle requires a cron job for background tasks.

Open the crontab for the web server user:

```bash
sudo crontab -u www-data -e
```

Add this line:

```bash
* * * * * /usr/bin/php /var/www/moodle/admin/cli/cron.php >/dev/null
```

This runs Moodle background tasks every minute.

---

## 12. Check Moodle Health and Environment

After installation, check Moodle’s environment page:

```text
Site administration
→ Server
→ Environment
```

Also check:

```text
Site administration
→ Reports
→ Security checks
```

These pages help identify:

* missing PHP extensions
* wrong PHP version
* insecure file access
* server configuration problems
* HTTPS problems
* database configuration problems

---

## 13. Important Project Lesson ILL Spring 2026

For this project, the biggest problem was not the server machine.

The bigger issue was compatibility between:

```text
Moodle version
PHP version
Operating system packages
Composer dependencies
Plugin versions
```

Therefore, the setup should always be planned from the plugin requirements first.

The better approach is:

```text
1. Decide which plugins are required.
2. Check which Moodle versions they support.
3. Choose a Moodle version.
4. Choose a compatible PHP version.
5. Choose an operating system version that supports that PHP version.
6. Install Moodle.
7. Install and test the plugins.
```

---

## 14. What Should Be Backed Up

A complete Moodle backup should include:

```text
Moodle source code
Moodle data directory
Moodle database
Apache configuration
Moodle config.php
```

Example backup files:

```text
moodle-code-current.tar.gz
moodledata-current.tar.gz
moodle-db-current.sql
moodle-config.php.backup
apache-moodle.conf.backup
```

Important:

> Moodle cannot be restored from the code folder alone.
> The database and moodledata directory are also required.

---

## 15. Short Summary

The Moodle installation for this project was built around Moodle 5.0.x because plugin compatibility was more important than using the newest Moodle version.

The installation used Apache2, PHP, MariaDB/MySQL, a Moodle code directory, and a separate Moodle data directory.

After the basic installation, Moodle cron, environment checks, and security checks were configured.

The most important technical lesson was that Moodle deployment must be planned around version compatibility, especially when additional plugins are required.

