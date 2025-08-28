# Kora Manual Installation Tutorial (DigitalOcean Droplet)

This repository contains a step-by-step guide for installing [Kora](https://github.com/matrix-msu/kora) — a digital repository platform — on a [DigitalOcean Droplet](https://www.digitalocean.com/).  
It is written with **students, scholars, and cultural heritage professionals** in mind — people who are experts in their own fields but may not have much technical background.  

The tutorial is designed to do two things at once:
1. Help you successfully install and run Kora.  
2. Teach you the basics of how servers work, how to connect to them, and how software like Kora fits together with web servers, databases, and the command line.  

By the end, you’ll have a working Kora installation and a clearer sense of how server-based applications are managed.  

---

## Table of Contents
1. [What is Kora?](#what-is-kora)
2. [What You’ll Need](#what-youll-need)
3. [Step 1: Create the Droplet](#step-1-create-the-droplet-your-small-cloud-server)
4. [Step 2: Connect to the Server with SSH](#step-2-connect-to-the-server-with-ssh)
5. [Step 3: Create a Safe Admin User](#step-3-create-a-safe-admin-user-recommended)
6. [Step 4: Turn on a Simple Firewall](#step-4-turn-on-a-simple-firewall)
7. [Step 5: Update the System](#step-5-update-the-system)
8. [Step 6: Install Apache (the Web Server)](#step-6-install-apache-the-web-server)
9. [Step 7: Install PHP and Extensions](#step-7-install-php-and-extensions)
10. [Step 8: Install MySQL (the Database)](#step-8-install-mysql-the-database)
11. [Step 9: Install Composer](#step-9-install-composer)
12. [Step 10: Download and Set Up Kora](#step-10-download-and-set-up-kora)
13. [Step 11: Configure Kora Environment File](#step-11-configure-kora-environment-file)
14. [Step 12: Configure Apache for Kora](#step-12-configure-apache-for-kora)
15. [Step 13: Run Database Migrations](#step-13-run-database-migrations)
16. [Step 14: Access Kora in Your Browser](#step-14-access-kora-in-your-browser)
17. [Step 15: Optional Domain and HTTPS](#step-15-optional-domain-and-https)
18. [Troubleshooting](#troubleshooting-cheatsheet)
19. [Quick Command Summary](#quick-command-summary-copy-paste-block)
20. [Official References](#reference-links-official)

---

## What is Kora?

[Kora](https://github.com/matrix-msu/kora) is an open source, web-based digital repository platform designed for cultural heritage institutions, projects, and scholars.  

It allows you to catalog, curate, manage, preserve, share, and display digital objects and metadata.  
Kora supports **documents, images, audio, video, 3D models, and even non-material entities like people or events**.  

Key features:
- Flexible metadata schemas and record structures.  
- Ability to create multiple forms for managing different types of records.  
- Community-driven open-source project developed by [MATRIX at Michigan State University](https://matrix.msu.edu).  

---

## What You’ll Need

- A [DigitalOcean account](https://www.digitalocean.com/).  
- A Droplet running **Ubuntu 24.04 LTS** (2 GB RAM recommended).  
- Optional: a domain name (for easier access and HTTPS).  
- No prior server knowledge is required — this tutorial explains everything step by step.  

Kora requirements:  
- PHP 8.1 or higher with required extensions.  
- MySQL 5.7.20 or newer.  
- Apache (≥2.0) with `mod_rewrite`.  
- Composer (PHP dependency manager).  

---

## Step 1: Create the Droplet (your small cloud server)

1. In the [DigitalOcean control panel](https://docs.digitalocean.com/products/droplets/how-to/create/), click **Create → Droplet**.  
2. Choose **Ubuntu 24.04 LTS**.  
3. Pick the $6/month plan (2 GB RAM is recommended for Kora).  
4. Choose a datacenter region close to you.  
5. Select **Password** login for simplicity.  
6. Name it `kora-server` and click **Create Droplet**.  

**Why.** A Droplet is DigitalOcean’s name for a small server you rent in the cloud. Think of it as a computer that runs 24/7 and can be reached from anywhere.  

---

## Step 2: Connect to the Server with SSH

On your computer, open a terminal (Mac/Linux) or PowerShell (Windows).  
Connect to your server using:  

```bash
ssh root@YOUR_SERVER_IP
```

- `ssh` is the program for connecting securely to another computer.  
- `root` is the default superuser account on the server.  
- `YOUR_SERVER_IP` is the numeric address DigitalOcean gave you.  

**Why.** This step is how you “sit down in front of” your cloud server, even though it lives in a datacenter.  

---

## Step 3: Create a Safe Admin User (recommended)

```bash
adduser sammy
usermod -aG sudo sammy
```

Reconnect as your new user:

```bash
exit
ssh sammy@YOUR_SERVER_IP
```

**Why.** The `root` account has unlimited power, which is dangerous if you make a mistake. By creating a new user with `sudo` privileges, you work more safely day-to-day.  

---

## Step 4: Turn on a Simple Firewall

```bash
sudo ufw allow OpenSSH
sudo ufw allow "Apache Full"
sudo ufw enable
```

**Why.** A firewall is like a security gate for your server. It only allows approved traffic — in this case web traffic (Apache) and remote access (SSH) — and blocks everything else.  

---

## Step 5: Update the System

```bash
sudo apt update
sudo apt upgrade -y
```

**Why.** `apt` is Ubuntu’s package manager, like an app store for your server. Updating makes sure all your software is patched and secure.  

---

## Step 6: Install Apache (the Web Server)

```bash
sudo apt install -y apache2
sudo systemctl enable --now apache2
```

Check in your browser: `http://YOUR_SERVER_IP` should show Apache’s default page.  

**Why.** Apache is the program that listens for requests from web browsers and responds with web pages. Without it, your server cannot “serve” a website.  

---

## Step 7: Install PHP and Extensions

```bash
sudo apt install -y php php-mysql php-xml php-mbstring php-bcmath php-curl php-zip unzip
```

**Why.** Kora is written in PHP. These extensions give it extra abilities, such as talking to the database (`php-mysql`), handling XML (`php-xml`), working with compressed files (`php-zip`), and more.  

---

## Step 8: Install MySQL (the Database)

```bash
sudo apt install -y mysql-server
sudo systemctl enable --now mysql
```

Enter MySQL:

```bash
sudo mysql
```

Create a database and user:

```sql
CREATE DATABASE kora;
CREATE USER 'korauser'@'localhost' IDENTIFIED BY 'STRONG_PASSWORD';
GRANT ALL PRIVILEGES ON kora.* TO 'korauser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**Why.** MySQL is like a structured spreadsheet where Kora stores its information.  
- `CREATE DATABASE` makes a new storage space named `kora`.  
- `CREATE USER` adds a login that Kora will use.  
- `GRANT` gives that user permission to manage the `kora` database.  

---

## Step 9: Install Composer

```bash
sudo apt install -y composer git
composer --version
```

**Why.** Composer is a tool that installs and manages the extra PHP packages that Kora depends on. Think of it as a librarian pulling the books Kora needs before it can start working.  

---

## Step 10: Download and Set Up Kora

Navigate to `/var/www` (the folder where web applications live):

```bash
cd /var/www
sudo git clone https://github.com/matrix-msu/kora.git
sudo chown -R $USER:$USER kora
cd kora
```

Install dependencies:

```bash
composer install
```

**Why.**  
- `git clone` grabs a copy of the Kora code from GitHub.  
- `composer install` pulls in the PHP libraries Kora needs to run.  

---

## Step 11: Configure Kora Environment File

Copy the example configuration:

```bash
cp .env.example .env
nano .env
```

Inside `.env`, edit these values to match your setup:

```
APP_URL=http://YOUR_SERVER_IP
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=kora
DB_USERNAME=korauser
DB_PASSWORD=STRONG_PASSWORD
```

**Why.** The `.env` file is like a settings sheet for Kora. It tells Kora where its database is, what URL it should use, and other important configuration details.  

---

## Step 12: Configure Apache for Kora

Create a new Apache configuration file:

```bash
sudo nano /etc/apache2/sites-available/kora.conf
```

Paste:

```
<VirtualHost *:80>
    ServerName YOUR_DOMAIN_OR_IP
    DocumentRoot /var/www/kora/public

    <Directory /var/www/kora/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/kora_error.log
    CustomLog ${APACHE_LOG_DIR}/kora_access.log combined
</VirtualHost>
```

Enable the site:

```bash
sudo a2ensite kora.conf
sudo a2enmod rewrite
sudo systemctl reload apache2
```

**Why.** This tells Apache to serve Kora from the `public` folder, which is the safe part of the application meant to be exposed on the web. The rest of Kora’s code stays hidden for security.  

---

## Step 13: Run Database Migrations

```bash
php artisan migrate
```

**Why.** Migrations are like blueprints for the database. This command builds the tables and fields Kora needs to function.  

---

## Step 14: Access Kora in Your Browser

Go to:

```
http://YOUR_SERVER_IP
```

You should see Kora’s welcome page. Follow the prompts to create your admin account.  

**Why.** This is the moment everything comes together: Apache serves the site, PHP runs Kora’s code, and MySQL stores the data.  

---

## Step 15: Optional Domain and HTTPS

To use a real domain name and HTTPS encryption:  

1. Point your domain to DigitalOcean DNS ([instructions](https://docs.digitalocean.com/products/networking/dns/getting-started/dns-registrars/)).  
2. Install Let’s Encrypt for free HTTPS certificates:  

```bash
sudo apt install -y certbot python3-certbot-apache
sudo certbot --apache -d yourdomain.org -d www.yourdomain.org
```

**Why.** Without HTTPS, your site still works but isn’t encrypted. HTTPS protects user logins and makes your site look professional.  

---

## Troubleshooting Cheatsheet

- **White screen or error**: Check Apache’s logs: `sudo tail -f /var/log/apache2/error.log`.  
- **Database errors**: Double-check `.env` for typos in the DB name, user, or password.  
- **Permission denied**: Make sure Apache owns the Kora files:  
  ```bash
  sudo chown -R www-data:www-data /var/www/kora
  ```  
- **PHP errors**: Ensure all required extensions are installed.  

---

## Quick Command Summary (Copy-Paste Block)

```bash
# Update and basic setup
sudo apt update && sudo apt upgrade -y
sudo apt install -y apache2 mysql-server php php-mysql php-xml php-mbstring php-bcmath php-curl php-zip unzip git composer

# Database setup
sudo mysql -e "CREATE DATABASE kora; CREATE USER 'korauser'@'localhost' IDENTIFIED BY 'STRONG_PASSWORD'; GRANT ALL PRIVILEGES ON kora.* TO 'korauser'@'localhost'; FLUSH PRIVILEGES;"

# Kora setup
cd /var/www && sudo git clone https://github.com/matrix-msu/kora.git && sudo chown -R $USER:$USER kora && cd kora
composer install
cp .env.example .env
php artisan migrate
```

---

## Reference Links (Official)

- **Kora**
  - [GitHub Repository](https://github.com/matrix-msu/kora)  
  - [Documentation](https://msu-dhi-lab.github.io/kora-documentation/)  
  - [Installing on LAMP](https://msu-dhi-lab.github.io/kora-documentation/getting-started/installing_kora_lamp/)  
  - [Cloud Install](https://msu-dhi-lab.github.io/kora-documentation/getting-started/cloud_install/)  

- **DigitalOcean**
  - [Create a Droplet](https://docs.digitalocean.com/products/droplets/how-to/create/)  
  - [Initial Ubuntu Setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu)  
  - [Apache on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-22-04)  
  - [MySQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04)  
  - [Firewall with UFW](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu)  
  - [Let’s Encrypt SSL](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu)  
