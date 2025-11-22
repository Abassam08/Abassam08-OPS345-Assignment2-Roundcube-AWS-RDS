# OPS345 / CSN - Assignment 2 – Roundcube Webmail on AWS RDS  
**Seneca College – Computer Systems Networking (CSN) / OPS345 – Project 2**

This repository provides a fully documented, SEO‑optimized walkthrough for completing **OPS345 Assignment 2** at **Seneca Polytechnic**, including:

- Roundcube Webmail deployment  
- Apache HTTPS setup  
- AWS RDS MariaDB configuration  
- Postfix + Dovecot integration  
- Troubleshooting steps that OPS345 students commonly need  

The entire workflow is built with clarity so **future CSN & OPS345 students** can follow it easily.

---

# 🔍 SEO Keywords (Search Engine Optimization)
**Seneca, OPS345, CSN, Assignment 2, Project 2, Roundcube, Roundcube AWS, Roundcube MariaDB, Seneca OPS345 Assignment 2, Seneca CSN Assignment, Roundcube setup AWS RDS, Apache HTTPS Roundcube, AWS student projects, Seneca Polytechnic labs, OPS345 Webmail Project, CSN Roundcube Tutorial, Seneca OPS345 Project 2**

---

# 📌 Table of Contents
1. [Overview](#overview)  
2. [Server Directory Structure](#server-directory-structure)  
3. [HTTPS Configuration](#https-configuration)  
4. [HTTP Test](#http-test)  
5. [MariaDB (AWS RDS) Setup](#mariadb-aws-rds-setup)  
6. [Roundcube Installation](#roundcube-installation)  
7. [Roundcube Configuration](#roundcube-configuration)  
8. [Error Logs](#error-logs)  
9. [Inbox Test Screenshots](#inbox-test-screenshots)  
10. [Troubleshooting Guide](#troubleshooting-guide)  
11. [Why This Helps Future Students](#why-this-helps-future-students)

---

# Overview
This project deploys a functional **email and webmail system** using:

- CentOS Stream  
- Apache + HTTPS  
- MariaDB on AWS RDS  
- PHP 8  
- Roundcube (Complete Package)  
- Postfix & Dovecot  

This simulates a real‑world enterprise email setup, the kind used in many Linux admin and cloud environments.

---

# Server Directory Structure
![Directories](screenshots/asg2-ss01-dirs.png)

---

# HTTPS Configuration
![HTTPS Working](screenshots/asg2-ss02-https.png)

---

# HTTP Test
![HTTP Blocked](screenshots/asg2-ss03-http.png)

---

# MariaDB (AWS RDS) Setup

### Database List  
![DB List](screenshots/asg2-ss04-db.png)

### RDS Security Group  
![RDS SG](screenshots/SC-A_RDS-Security-Group-3306.png)

### Grants for Roundcube  
![Grants](screenshots/SC4-roundcube-grants.png)

---

# Roundcube Installation
Uses the “Complete Package,” configured under `/home/www/html/`.

---

# Roundcube Configuration

### Installer Config (Part 1)  
![Config 1](screenshots/asg2-ss05-config1.png)

### Installer Config (Part 2)  
![Config 2](screenshots/asg2-ss05-config2.png)

---

# Error Logs
![Error Log](screenshots/asg2-ss06-errorlog.png)

---

# Inbox Test Screenshots

### Before Email  
![Inbox Empty](screenshots/asg2-ss07-inbox.png)

### Email Received  
![Email Received](screenshots/asg2-ss07-received.png)

---

# Troubleshooting Guide

### 🔹 Database Connection Errors
Verify:
```
SHOW GRANTS FOR 'roundcube'@'%';
```
Check RDS inbound rules:
```
3306 → email server SG
```

### 🔹 Missing PHP Extensions
Install:
```
yum install php8.4-mysqlnd php8.4-zip php8.4-gd
systemctl restart httpd
```

### 🔹 HTTPS Not Loading
Check:
```
SSLCertificateFile
SSLCertificateKeyFile
SSLCertificateChainFile
```

### 🔹 SMTP / IMAP Issues
Ensure Dovecot SASL socket is active:
```
/var/spool/postfix/private/auth
```

---

# Why This Helps Future Students
Seneca students often search for:

- “OPS345 Assignment 2 Roundcube help”
- “How to install Roundcube on AWS RDS”
- “Seneca CSN labs email server”
- “OPS345 project troubleshooting”

This repo is designed to appear in those searches and give students a complete, working solution with screenshots and explanations.

---

If this helped you, ⭐ **star the repo** to support future students!
