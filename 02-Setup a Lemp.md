# Setup a LEMP stack on Ubuntu 18.04

- [Setup a LEMP stack on Ubuntu 18.04](#setup-a-lemp-stack-on-ubuntu-1804)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
    - [Step 1 : Installing the Nginx Web Server](#step-1--installing-the-nginx-web-server)
    - [Step 2 : Installing MySQL](#step-2--installing-mysql)
    - [Step 3 : Installing PHP](#step-3--installing-php)
    - [Step 4 : Configuring Nginx to Use the PHP Processor](#step-4--configuring-nginx-to-use-the-php-processor)
    - [Step 5 : Creating a PHP File to Test Configuration](#step-5--creating-a-php-file-to-test-configuration)
    - [Step 6 : Secure Nginx with Let's Encrypt on Ubuntu](#step-6--secure-nginx-with-lets-encrypt-on-ubuntu)
      - [Installing Certbot](#installing-certbot)
      - [Obtaining an SSL Certificate](#obtaining-an-ssl-certificate)
      - [Verifying Certbot Auto-Renewal](#verifying-certbot-auto-renewal)
      - [Confirming that Certbot worked](#confirming-that-certbot-worked)

## Introduction

The **LEMP** software stack is a group of software that can be used to serve dynamic web pages and web applications. This is an acronym that describes a **L**inux operating system, with an Nginx (pronounced like “**E**ngine-X”) web server. The backend data is stored in the **M**ySQL database and the dynamic processing is handled by **P**HP.

## Prerequisites

Before you complete this tutorial, you should have a regular, non-root user account on your server with sudo privileges. Set up this account by completing our initial server setup guide.

Once you have your user available, you are ready to begin the steps outlined in this guide.

## Installation

### Step 1 : Installing the Nginx Web Server

In order to display web pages to our site visitors, we are going to employ Nginx, a modern, efficient web server.

All of the software used in this procedure will come from Ubuntu’s default package repositories. This means we can use the apt package management suite to complete the necessary installations.

Since this is our first time using apt for this session, start off by updating your server’s package index. Following that, install the server:

```bash
sudo apt install software-properties-common python-software-properties
sudo apt update
sudo add-apt-repository ppa:nginx/stable
sudo apt install nginx
```

_Note: On Ubuntu 18.04, Nginx is configured to start running upon installation._

If you have the ufw firewall running, as outlined in the initial setup guide, you will need to allow connections to Nginx. Nginx registers itself with ufw upon installation, so the procedure is rather straightforward.

Enable this by typing:

```bash
sudo ufw allow 'Nginx Full'
```

You can verify the change by running:

```bash
sudo ufw status
```

This command’s output will show that HTTP and HTTPS traffic is allowed.

| To              | Action | From          |
| --------------- | ------ | ------------- |
| OpenSSH         | ALLOW  | Anywhere      |
| Nginx Full      | ALLOW  | Anywhere      |
| OpenSSH (v6)    | ALLOW  | Anywhere (v6) |
| Nginx Full (v6) | ALLOW  | Anywhere (v6) |

_Note: Nginx Full profile will allow both http and https traffic._

With the new firewall rule added, you can test if the server is up and running by accessing your server’s domain name or public IP address in your web browser. you should see an nginx welcome pge.

```
http://server_domain_or_IP
```

### Step 2 : Installing MySQL

Now that you have a web server, you need to install MySQL (a database management system) to store and manage the data for your site.

Install MySQL by typing:

```bash
sudo apt install mysql-server
```

The MySQL database software is now installed, but its configuration is not yet complete.

To secure the installation, MySQL comes with a script that will ask whether we want to modify some insecure defaults. Initiate the script by typing:

```bash
sudo mysql_secure_installation
```

### Step 3 : Installing PHP

You now have Nginx installed to serve your pages and MySQL installed to store and manage your data. However, you still don’t have anything that can generate dynamic content. This is where PHP comes into play.

Since Nginx does not contain native PHP processing like some other web servers, you will need to install php-fpm, which stands for “fastCGI process manager”. We will tell Nginx to pass PHP requests to this software for processing.

```bash
sudo apt install php-fpm php-mysql
```

### Step 4 : Configuring Nginx to Use the PHP Processor

You now have all of the required LEMP stack components installed.but you still need to make a few configuration changes in order to tell Nginx to use the PHP processor for dynamic content.

To do this, open a new server block configuration file within the /etc/nginx/sites-available/ directory.

```bash
sudo touch /etc/nginx/sites-available/website.com.conf
```

_Note:In this example, the new server block configuration file is named example.com, although you can name yours whatever you’d like_

By editing a new server block configuration file, rather than editing the default one, you’ll be able to easily restore the default configuration if you ever need to.

Add the following content, which was taken and slightly modified from the default server block configuration file, to your new server block configuration file:

```nginx
server {
	listen 80;
	listen [::]:80;

	root /var/www/website.com;

	# Add index.php to the list if you are using PHP
	index index.php index.html;

	server_name website.com;

	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}

	# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
	#
	location ~ \.php$ {
	    include snippets/fastcgi-php.conf;

	    # With php7.0-cgi alone:
	    # fastcgi_pass 127.0.0.1:9000;
	    # With php7.0-fpm:
	    fastcgi_pass unix:/run/php/php7.2-fpm.sock;
	}

	# deny access to .htaccess files, if Apache's document root
	# concurs with nginx's one
	#
	location ~ /\.ht {
	      deny all;
	}
}
```

After adding this content, save and close the file. Enable your new server block by creating a symbolic link from your new server block configuration file (in the /etc/nginx/sites-available/ directory) to the /etc/nginx/sites-enabled/ directory:

```bash
sudo ln -s /etc/nginx/sites-available/website.com.conf /etc/nginx/sites-enabled/
```

If you want to remove the default bloc,you can unlink the default configuration file from the /sites-enabled/ directory by typing the following command:

```
sudo unlink /etc/nginx/sites-enabled/default
```

Test your new configuration file for syntax errors by typing:

```
sudo nginx -t
```

If any errors are reported, go back and recheck your file before continuing.

When you are ready, reload Nginx to make the necessary changes:

```
sudo systemctl reload nginx
```

### Step 5 : Creating a PHP File to Test Configuration

Your LEMP stack should now be completely set up. You can test it to validate that Nginx can correctly hand .php files off to the PHP processor.

To do this, use your text editor to create a test PHP file called `index.php` in your document root:

```bash
sudo nano /var/www/website.com/index.php
```

Enter the following lines into the new file. This is valid PHP code that will return information about your server:

```php
<?php
phpinfo();

```

When you are finished, save and close the file, and now you can visit this page in your web browser by visiting your server’s domain name or public IP

```
http://your_server_domain_or_IP
```

You should see a web page that has been generated by PHP with information about your server.

### Step 6 : Secure Nginx with Let's Encrypt on Ubuntu

Let’s Encrypt is a Certificate Authority (CA) that provides an easy way to obtain and install free TLS/SSL certificates, thereby enabling encrypted HTTPS on web servers. It simplifies the process by providing a software client, **Certbot**, that attempts to automate most (if not all) of the required steps. Currently, the entire process of obtaining and installing a certificate is fully automated on both Apache and Nginx.

#### Installing Certbot

The first step to using Let’s Encrypt to obtain an SSL certificate is to install the **Certbot** software on your server.

First, add the repository:

```bash
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
```

Install Certbot & Certbot’s Nginx package with apt:

```bash
sudo apt-get install certbot python-certbot-nginx
```

#### Obtaining an SSL Certificate

Certbot provides a variety of ways to obtain SSL certificates through plugins. The Nginx plugin will take care of reconfiguring Nginx and reloading the config whenever necessary.

Run this command to get a certificate and have Certbot edit your Nginx configuration automatically to serve it, turning on HTTPS access in a single step.

```bash
sudo certbot --nginx
```

_Note: If this is your first time running certbot, you will be prompted to enter an email address and agree to the terms of service. After doing so, certbot will communicate with the Let’s Encrypt server, then run a challenge to verify that you control the domain you’re requesting a certificate for._

If that’s successful, certbot will ask how you’d like to configure your HTTPS settings.

```
Output
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):
```

Select the second choice then hit ENTER. The configuration will be updated, and Nginx will reload to pick up the new settings. certbot will wrap up with a message telling you the process was successful and where your certificates are stored.

If you're feeling more conservative and would like to make the changes to your Nginx configuration by hand, run this command.

```bash
sudo certbot certonly --nginx
```

If you want to specify the domain names that you want to get ssl for you can run this commands with the following flags

```bash
sudo certbot --nginx -d website.com -d www.website.com
```

#### Verifying Certbot Auto-Renewal

Let’s Encrypt's certificates are only valid for **90 days**. This is to encourage users to automate their certificate renewal process. The certbot package we installed takes care of this for us by adding a renew script to `/etc/cron.d`. This script runs twice a day and will automatically renew any certificate that’s within thirty days of expiration.

To test the renewal process, you can do a dry run with certbot:

```bash
sudo certbot renew --dry-run
```

If you see no errors, you’re all set.

#### Confirming that Certbot worked

To confirm that your site is set up properly, visit https://website.com/ in your browser and look for the lock icon in the URL bar. If you want to check that you have the top-of-the-line installation, you can head to https://www.ssllabs.com/ssltest/.
