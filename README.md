# OPS345 Assignment 2
## Roundcube Webmail + AWS RDS (Walkthrough)

![Status](https://img.shields.io/badge/status-completed-success)
![AWS](https://img.shields.io/badge/AWS-RDS-orange)
![Apache](https://img.shields.io/badge/Apache-2.4-red)

---

## Quick Start (TL;DR)

This project demonstrates deploying Roundcube webmail on an Amazon Linux 2023 email server, connecting to AWS RDS MariaDB. Key accomplishments:

- ✅ Apache + PHP + SSL configuration
- ✅ AWS RDS database setup with proper security groups
- ✅ Roundcube installation and configuration
- ✅ End-to-end email testing through web interface

**Tech Stack**: Apache 2.4, PHP 8.x, Roundcube 1.6.x, AWS RDS MariaDB, Let's Encrypt SSL

---

## Table of Contents

1. [Topology](#1-topology-high-level)
2. [Repository Layout](#2-repository-layout)
3. [Prerequisites](#3-prerequisites)
4. [Step-by-Step Walkthrough](#4-step-by-step-walkthrough)
5. [Lessons Learned](#5-lessons-learned)
6. [Troubleshooting Notes](#6-troubleshooting-notes)
7. [How to Use This Repo](#7-how-to-use-this-repo)
8. [Future Improvements](#8-future-improvements)
9. [License & Contact](#9-license--contact)

---

## 1. Topology (High-level)

```
┌─────────────────────────────────────────────────────────────┐
│                        Internet                              │
└────────────────────────────┬────────────────────────────────┘
                             │
                    DNS: email.asabra1.ops345.ca
                             │
                             ▼
                    ┌─────────────────┐
                    │   email VM      │
                    │  (Kali Linux)   │
                    ├─────────────────┤
                    │ Apache (HTTPS)  │
                    │ PHP 8.x         │
                    │ Roundcube       │
                    │ Postfix (SMTP)  │
                    │ Dovecot (IMAP)  │
                    └────────┬────────┘
                             │ Port 3306
                             │
                    ┌────────▼────────┐
                    │   AWS RDS       │
                    │   MariaDB       │
                    │                 │
                    │ DB: roundcube   │
                    │ mail            │
                    └─────────────────┘
```

**Architecture Components:**

- **email VM**: Kali Linux hosting Apache, PHP, Dovecot (IMAP), Postfix (SMTP), and Roundcube
- **Apache + PHP**: Web server serving Roundcube application in `/home/www/html/roundcube`
- **AWS RDS MariaDB**: Database hosting `roundcubemail` database
- **DNS**: `email.asabra1.ops345.ca` → email VM

---

## 2. Repository Layout

Recommended structure for this repo:

```
OPS345-A2-Roundcube/
├─ README.md
├─ submission/
│  └─ asg2.tar.gz                      # Final submission archive
└─ screenshots/
   ├─ asg2-ss01-dirs.png
   ├─ asg2-ss02-https.png
   ├─ asg2-ss03-http.png
   ├─ SC-A_RDS-Security-Group-3306.png
   ├─ SC-B_Databases-List.png
   ├─ SC4-roundcube_grants.png
   ├─ asg2-ss04-db.png
   ├─ asg2-ss05-config1.png
   ├─ asg2-ss05-config2.png
   ├─ asg2-ss06-errorlog.png
   ├─ asg2-ss07-inbox.png
   └─ asg2-ss07-received.png
```

---

## 3. Prerequisites

### Software Versions Used

- **Operating System**: Amazon Linux 2023 (email server VM)
- **Apache**: 2.4.x
- **PHP**: 8.4.14
- **Roundcube**: 1.6.x (Complete package)
- **MariaDB**: 10.x (AWS RDS)

### Required Setup

Working email VM from previous OPS345 labs:

- Apache, Postfix, Dovecot already configured
- HTTPS virtual host for `email.asabra1.ops345.ca`
- Let's Encrypt SSL certificates installed

AWS Academy account with:

- Existing MariaDB RDS instance (`ops345db`)
- Security group you can edit
- VPC and subnet configurations from previous labs

SSH access to the email VM as `asabra1` (and root).

---

## 4. Step-by-Step Walkthrough

### Step 1 – Verify /home/www and document root

**Time Estimate: 5-10 minutes**

On the email server I confirmed `/home` was the XFS LV, and that `/var/www` is symlinked to `/home/www` with `html` under it.

**Commands:**

```bash
mount | grep home
ls -lad /var/www
ls -la /var/www /home/www/
```

**Screenshot:**

![asg2-ss01-dirs](screenshots/asg2-ss01-dirs.png)

*Screenshot shows: mount output and directory listing showing `/var/www` -> `/home/www` and the `html` directory owned by `apache:apache`.*

---

### Step 2 – Confirm HTTPS and HTTP on email.asabra1.ops345.ca

**Time Estimate: 10-15 minutes**

#### Testing HTTPS

First I tested HTTPS and saw the Roundcube installer page, which proves:

- DNS for `email.asabra1.ops345.ca` works
- Apache SSL vhost is serving content from `/home/www/html`

**Screenshot:**

![asg2-ss02-https](screenshots/asg2-ss02-https.png)

*Screenshot shows: Roundcube installer over `https://email.asabra1.ops345.ca/installer/`.*

#### Testing HTTP (Before Restriction)

Then I tested HTTP (plain `http://email.asabra1.ops345.ca/`) and saw the default directory listing (`Index of /`). This is before Roundcube was moved into the document root.

**Screenshot:**

![asg2-ss03-http](screenshots/asg2-ss03-http.png)

*Screenshot shows: HTTP access showing `Index of /` on `email.asabra1.ops345.ca`.*

**Note**: Later in the configuration, HTTP access was restricted to HTTPS only.

---

### Step 3 – Open RDS security group for the email VM

**Time Estimate: 10-15 minutes**

In AWS I edited the RDS DB security group to allow MySQL (TCP 3306) from the two lab security groups (web + email). That lets the email VM talk to the MariaDB instance.

**Screenshot:**

![SC-A_RDS-Security-Group-3306](screenshots/SC-A_RDS-Security-Group-3306.png)

*Screenshot shows: Security group `ops345dbsg` inbound rules – MYSQL/Aurora, TCP 3306 from the web and email SGs.*

---

### Step 4 – Create roundcubemail DB and the roundcube user

**Time Estimate: 15-20 minutes**

Connected to the RDS instance from the email VM using `mysql` and created the database:

```sql
CREATE DATABASE roundcubemail;
SHOW DATABASES;
```

**Screenshot:**

![SC-B_Databases-List](screenshots/SC-B_Databases-List.png)

*Screenshot shows: `SHOW DATABASES;` output showing the new `roundcubemail` database.*

Then I created the `roundcube` user and granted full access to that DB:

```sql
CREATE USER 'roundcube'@'%' IDENTIFIED BY 'SomeStrongPass123!';
GRANT USAGE ON *.* TO 'roundcube'@'%';
GRANT ALL PRIVILEGES ON `roundcubemail`.* TO 'roundcube'@'%';
FLUSH PRIVILEGES;
SHOW GRANTS FOR 'roundcube'@'%';
```

**Screenshot:**

![SC4-roundcube_grants](screenshots/SC4-roundcube_grants.png)

*Screenshot shows: `SHOW GRANTS FOR 'roundcube'@'%';` confirming USAGE on all DBs and ALL PRIVILEGES on `roundcubemail.*`*

The Roundcube installer's DB test later confirmed connectivity and schema creation:

**Screenshot:**

![asg2-ss04-db](screenshots/asg2-ss04-db.png)

*Screenshot shows: Roundcube "Test config" page showing DSN (write) / DB Schema / DB Write / DB Time: OK.*

---

### Step 5 – Configure Roundcube via config.inc.php

**Time Estimate: 30-45 minutes**

I used the installer **Step 2 – Create config** to generate the PHP config, then copied the output text into `/home/www/html/roundcube/config/config.inc.php` (overwriting the existing template).

#### Key Settings:

```php
$config['db_dsnw'] = 'mysql://roundcube:SomeStrongPass123%21@ops345db.cl2eqicgq1i1.us-east-1.rds.amazonaws.com/roundcubemail';
$config['imap_host'] = 'localhost:143';
$config['smtp_host'] = 'localhost:25';
$config['auto_create_user'] = true;
$config['des_key'] = 'DquWSVzJUEXdni5hClZksukc';
$config['plugins'] = [];
$config['prefer_html'] = false;
```

**Important Notes:**

- URL-encode special characters in passwords (e.g., `!` becomes `%21`)
- `auto_create_user = true` allows automatic user creation on first login
- IMAP/SMTP use localhost without encryption (internal communication)

#### Installer Screenshots:

**Screenshot:**

![asg2-ss05-config1](screenshots/asg2-ss05-config1.png)

*Screenshot shows: Roundcube "Create config" page filled in with MySQL DSN, IMAP/SMTP host, and `auto_create_user` checked.*

**Screenshot:**

![asg2-ss05-config2](screenshots/asg2-ss05-config2.png)

*Screenshot shows: "Test config" screen – both `defaults.inc.php` and `config.inc.php` OK, directories writable, DB tests all OK.*

---

### Step 6 – Verify Apache SSL error log

**Time Estimate: 5 minutes**

The assignment asked for a snapshot of any SSL errors. In this case, there were no recent errors, which is still valid.

**Command:**

```bash
sudo su -
date && tail /var/log/httpd/ssl_error_log
```

**Screenshot:**

![asg2-ss06-errorlog](screenshots/asg2-ss06-errorlog.png)

*Screenshot shows: `date && tail /var/log/httpd/ssl_error_log` on the email server, showing an empty/clean SSL error log.*

---

### Step 7 – Log in to Roundcube

**Time Estimate: 10-15 minutes**

Opened:

```
https://email.asabra1.ops345.ca/
```

Logged in with my local mail account:

- **Username**: `asabra1`
- **Password**: `Bassam08` (set earlier with `passwd` on the email server)

**Screenshot:**

![asg2-ss07-inbox](screenshots/asg2-ss07-inbox.png)

*Screenshot shows: Roundcube inbox for `asabra1@localhost` after successful login, showing forwarded test messages from previous labs.*

---

### Step 8 – Send a test message (bonus)

**Time Estimate: 10 minutes**

From Roundcube I composed a new email and sent it from `asabra1@localhost` to `asabra1@localhost` with the subject "TEST" and a short body, then verified it arrived.

**Screenshot:**

![asg2-ss07-received](screenshots/asg2-ss07-received.png)

*Screenshot shows: Message view of the "TEST" email in Roundcube (From: `asabra1@localhost` to `asabra1@localhost`), confirming end-to-end delivery through Postfix/Dovecot/Roundcube.*

---

## 5. Lessons Learned

### What Worked Well

- **Symbolic linking** `/var/www` to `/home/www` allows flexible storage management without breaking Apache's default paths
- **AWS RDS** handles database backups automatically, removing that operational burden from the system administrator
- **Roundcube's installer** makes initial configuration much easier than manual setup - the web-based interface catches configuration errors early
- **Modular approach** to the assignment (breaking it into distinct steps) made troubleshooting much more manageable

### Challenges Overcome

- **SQL syntax errors** with GRANT statements - learned the difference between backticks (for database/table names) and single quotes (for strings)
- **Understanding Apache document root** and how it relates to Roundcube installation path - initially confused about whether to access `/roundcube/` subdirectory or configure document root directly
- **AWS Security Groups** configuration - required allowing MySQL traffic (port 3306) from multiple security groups to enable communication between VMs and RDS
- **Password encoding in DSN strings** - special characters in passwords must be URL-encoded (e.g., `!` → `%21`)

### Skills Developed

- AWS RDS database administration and security group configuration
- Apache virtual host configuration with SSL/TLS
- PHP application deployment and troubleshooting
- DNS configuration and verification for mail services
- Understanding email flow: SMTP (Postfix) → IMAP (Dovecot) → Webmail (Roundcube)
- Systematic troubleshooting methodology across multiple infrastructure layers

---

## 6. Troubleshooting Notes

This section is to help anyone who hits the same issues I encountered.

### 6.1. DB Schema: NOT OK (Database not initialized)

**Symptom:**

In the installer "Test config" page: `DB Schema: NOT OK`.

**Cause:**

- Database existed, but Roundcube tables were never created
- Or grants on `roundcubemail.*` were missing/incorrect

**Fix:**

1. Make sure user + grants are correct:

   ```sql
   SHOW GRANTS FOR 'roundcube'@'%';
   ```

2. In the installer, click "Initialize database"
3. Re-run the test and confirm all DB checks show OK

---

### 6.2. SQL ERROR 1064 (42000) on GRANT statements

**Symptom:**

```
ERROR 1064 (42000): You have an error in your SQL syntax...
```

when running:

```sql
GRANT ALL PRIVILEGES ON 'roundcubemail'.* TO 'roundcube'@'%';
```

**Cause:**

Using single quotes around the database name instead of backticks, or stray quotes.

**Fix:**

Use this exact syntax:

```sql
GRANT ALL PRIVILEGES ON `roundcubemail`.* TO 'roundcube'@'%';
```

**Note:** Use backticks (\`) around database/table names, and single quotes (') around strings.

---

### 6.3. Roundcube login fails but Dovecot works from CLI

**Symptom:**

Login failed in the Roundcube UI, even though IMAP works with `telnet` or `openssl s_client`.

**Possible causes & fixes:**

1. **`auto_create_user` set to false**
   
   → Set it to `true` in `config.inc.php`:

   ```php
   $config['auto_create_user'] = true;
   ```

2. **Wrong IMAP host/port**
   
   → Use `localhost:143` (or `ssl://localhost:993` if you configured IMAPS)

3. **Configuration not reloaded**

   Restart Apache after changes:

   ```bash
   sudo systemctl restart httpd
   ```

---

### 6.4. https://email.asabra1.ops345.ca shows "Index of /" instead of Roundcube

**Symptom:**

You browse to the HTTPS URL and only see an Apache directory listing.

**Fix:**

Make sure Roundcube is extracted under the document root of the SSL vhost (for this lab: `/home/www/html/roundcube`).

Either:

- Access it via `/roundcube/` (easiest), or
- Update the vhost `DocumentRoot` to point at the Roundcube directory and restart Apache

**Example Apache vhost configuration:**

```apache
<VirtualHost *:443>
    ServerName email.asabra1.ops345.ca
    DocumentRoot /home/www/html/roundcube
    
    SSLEngine on
    SSLCertificateFile /path/to/cert.pem
    SSLCertificateKeyFile /path/to/privkey.pem
    
    <Directory /home/www/html/roundcube>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

---

### 6.5. SSL error log is empty

**Symptom:**

`tail /var/log/httpd/ssl_error_log` shows nothing, but Apache is working fine.

**Explanation:**

That just means there were no SSL errors yet - which is actually a good thing!

For the assignment screenshot, it's enough to show the command output and current date/time to prove Apache SSL is operational.

---

### 6.6. Connection timeout to AWS RDS

**Symptom:**

Cannot connect to RDS from email VM - connection times out.

**Possible causes & fixes:**

1. **Security group not configured**
   
   → Check RDS security group allows inbound MySQL (port 3306) from your email VM's security group

2. **Wrong RDS endpoint**
   
   → Verify you're using the correct RDS endpoint from AWS console

3. **VPC/Subnet mismatch**
   
   → Ensure your email VM and RDS instance are in the same VPC or have proper routing configured

**Test connection:**

```bash
mysql -h ops345db.cl2eqicgq1i1.us-east-1.rds.amazonaws.com -u admin -p
```

---

## 7. How to Use This Repo

### Clone the Repository

```bash
git clone https://github.com/Abassam08/OPS345-A2-Roundcube.git
cd OPS345-A2-Roundcube
```

### Follow the Walkthrough

1. Open `README.md` and follow the steps in Section 4
2. Reference screenshots in the `screenshots/` directory
3. Adapt configurations to your own environment (domain names, IP addresses, etc.)

### Review Troubleshooting

- Check Section 6 if you encounter any issues
- Each problem includes symptoms, causes, and tested solutions

---

## 8. Future Improvements

Potential enhancements for this setup:

- [ ] Configure IMAPS (port 993) with SSL/TLS encryption for secure IMAP connections
- [ ] Enable Roundcube plugins:
  - Password change plugin
  - Calendar integration
  - Contact management
- [ ] Set up automated backups of Roundcube configuration and RDS database
- [ ] Implement fail2ban for brute-force protection on webmail login
- [ ] Configure SPF, DKIM, and DMARC records for better email deliverability and anti-spoofing
- [ ] Add monitoring and alerting:
  - Apache/PHP performance metrics
  - Email queue monitoring
  - Database connection health checks
- [ ] Implement rate limiting on SMTP to prevent abuse
- [ ] Create custom Roundcube theme matching organizational branding
- [ ] Set up email archiving for compliance

---

## 9. License & Contact

This project was completed as part of **OPS345 (Systems Administration)** at **Seneca College**.

**Author**: Ahmed Sabra  
**Course**: OPS345 - Assignment 2  
**Date**: November 2024  
**Institution**: Seneca College  
**GitHub**: [@Abassam08](https://github.com/Abassam08)

### Acknowledgments

- Thanks to the OPS345 instructors for providing comprehensive lab materials
- AWS Academy for cloud infrastructure resources
- Roundcube community for excellent documentation

### Feedback

Feel free to reach out if you have questions about this implementation or suggestions for improvements!

---

*This documentation is provided for educational purposes. All configurations should be reviewed and adapted for production environments with proper security hardening.*

---

**Total Project Time**: ~4-5 hours  
**Difficulty Level**: Intermediate  
**Key Takeaway**: Integrating web applications with cloud database services requires careful attention to networking, security, and configuration details across multiple layers of infrastructure.
