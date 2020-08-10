# Setup correct Permissions

- [Setup correct Permissions](#setup-correct-permissions)
  - [Introduction](#introduction)
  - [`Deployer` user as owner](#deployer-user-as-owner)
    - [Using ACL (recommended)](#using-acl-recommended)
    - [Using chmod;](#using-chmod)
  - [Webserver as owner:](#webserver-as-owner)

## Introduction

There are basically two ways to setup your ownership and permissions.
Either you give deployer ownership or you make the webserver the owner of all files.

## `Deployer` user as owner

If you prefer that deployer own all the directories and files:

```bash
sudo chown -R deployer:www-data /var/www/laravel.com
```

Most folders should be normal "755" and files, "644" :

```bash
sudo find /var/www/laravel.com -type f -exec chmod 664 {} \;
sudo find /var/www/laravel.com -type d -exec chmod 755 {} \;
```

Laravel requires some folders to be writable for the web server user. You can use these command on\*nix based OSs.

### Using ACL (recommended)

`deployer` = deployment user (use to login via ssh)
`www-data` = web server user (nginx)

```bash
sudo setfacl -Rd -m u:deployer:rwx,g:www-data:rwx,o:rx storage bootstrap/cache
sudo setfacl -R -m u:deployer:rwx,g:www-data:rwx,o:rx storage bootstrap/cache
```

### Using chmod;

If you don't have ACL, you can use these but they're not so great

```bash
sudo chgrp -R www-data storage bootstrap/cache
sudo chmod -R ug+rwx storage bootstrap/cache
```

## Webserver as owner:

Assuming `www-data` (it could be something else) is your webserver user.

```bash
sudo chown -R www-data:www-data /var/www/laravel.com
```

if you do that, the webserver owns all the files, and is also the group, and you will have some problems uploading files or working with files via FTP, because your FTP client will be logged in as you, not your webserver, so add your user to the webserver user group:

```bash
sudo usermod -a -G www-data ubuntu
```

Then you set all your directories to 755 and your files to 644... SET file permissions

```bash
sudo find /var/www/laravel.com -type f -exec chmod 644 {} \;
```

SET directory permissions

```bash
sudo find /var/www/laravel.com -type d -exec chmod 755 {} \;
```

Now, you're secure and your website works, AND you can work with the files fairly easily.
