# lizferLinux
Provisioning of a Wordpress website implementation on cloud based on a server with LAMP (Linux 🐧, Apache2 🖌️, MySQL 🐬 and PHP 🐘). Check it at http://188.166.90.196 / http://lizfer.com.

I decided to create a personal space where I could demonstrate my Linux system administration skills. The prerrogative was:
- This space needs to be operational anytime, anywhere (availability); and
- it must meet minimal performance standards (reliability).


## 1. Linux machine

### 1.a: Provisioning the cloud machine

The first step was provisioning an instance (that is, a _droplet_) at Digital Ocean. My pick was Ubuntu 20.04 LTS, hosted at Amsterdam (default-ams3).

![image](https://user-images.githubusercontent.com/22382891/132209088-88bf6633-9c73-4460-9c9e-ec7e6225c9a3.png)

After having the instance ready (with my SSH public and private keys properly installed), the next step was configuring the server to have the LAMP setup installed. 

First things first: `ssh root@188.166.90.196` into it, then `apt-get update && apt-get upgrade`. Now, good practices: create a regular user and stop using the root account.

### 1.b: User creation and authentication

This was pretty straightforward: 

1. Ran `adduser lizfer` and created the PID 1000 user; 
2. Added it to the sudoers group using `usermod -aG sudo lizfer`;
3. Allowed it to use the SSH public key by copying it from root (and granted ownership of this copy) using rsync: `rsync --archive --chown=lizfer:lizfer ~/.ssh /home/lizfer`. 

At this point, my newly-created user had administrative privileges whenever it used `sudo` and I could use this account to access from any remote client that had the SSH private key installed.

### 1.c: Firewall

Although this was a simple demonstration, it's always good practice to have a firewall set, to avoid undesired connections. Since that this is an Ubuntu machine, I used UFW. To enable it, I just had to enter `ufw enable`. As other software are installed, description on firewall settings will be added.


## 2. Apache2

From this moment onwards, I decided to authenticate using the PID 1000 account (lizfer) for testing reasons - hence why lots of `sudo` will appear.

Configuring the "A" in LAMP was a handful of work - my first attempt was configuring a Nginx web server, but conflicts with php-fpm and cacheing prevented me from doing so. 

### 2.a: Downloading

The first step was getting Apache2 from Ubuntu's repos using `sudo apt install apache2`. Just had to accept the installation terms.

### 2.b: Firewall settings

By downloading Apache, UFW automatically creates profiles for Apache based on ports available for access. By entering `sudo ufw app list`, the list of available applications was prompted on the stdout (tl;dr: **Apache** profile opens only port 80, **Apache Full** opens both port 80 and port 443 and **Apache Secure** opens only port 443):

    Available applications:
      Apache
      Apache Full
      Apache Secure
      OpenSSH

To configure the TLS/SSL certificate for Wordpress, I allowed communication in ports 80 and 443 by enabling the **Apache Full** profile: `sudo ufw allow in "Apache Full"`.


## 3. MySQL

It might look scary at first, but setting up a database was not that difficult due to Ubuntu's configuration.

Just had to use `sudo apt install mysql-server` to start downloading. Once completed, I ran `sudo mysql_secure_installation` to configure it using MySQL's security script:
- For the `VALIDATE PASSWORD PLUGIN`, I set a MySQL root password with `STRONG` length (security should always be strong, IMHO);
- Disabled root account access from a remote client (outside of the local host);
- Removed anonymous user pre-set accounts;
- Removed the test database, and privileges that allow access to databases with names starting with "test_". 

Setting up a MySQL root password was extremely cautious, since that the default authentication method for root is using a unix_socket. 


## 4. PHP

PHP installation by itself is not enough: I also installed `php-mysql` (module that allows PHP to communicate with MySQL) and `libapache2-mod-php` (to enable Apache to handle PHP files). Just entered `sudo apt install php php-mysql libapache2-mod-php`.

At this point, the LAMP stack was already installed and I had a production instance working.


## 5. Pre-Wordpress Configuration

This chapter is reserved for configuration that must be applied before installing Wordpress.

### 5.a: Apache's Virtual Host configuration

*I decided to use this machine for hosting more than one domain. Therefore, the configuration of an Apache Virtual Host was necessary. Data is displayed as `lizfer` but this is a placeholder - upon provisioning a new domain for this machine, the proper data must be used.*

The default `/var/www/html` directory remained as backup for the original files; I created a new folder under `www` (`sudo mkdir /var/www/lizfer`) and assigned its ownership to the PID 1000 user (`sudo chown -R $USER:$USER /var/www/lizfer`).

Then, I created a new configuration file under Apache structure using Nano: `sudo nano /etc/apache2/sites-available/lizfer.conf` and included the following content in it:
```
<VirtualHost *:80>
    ServerName lizfer
    ServerAlias www.lizfer
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/lizfer
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
I used `sudo a2ensite lizfer` to instruct Apache which is the current active server, and disabled the default website using `sudo a2dissite 000-default`. Ran the final configuration test with `sudo apache2ctl configtest` (with a `Syntax OK` response) and rebooted the instance.

### 5.b: Domain acquisition and configuration

This step may differ depending on the administrator. 

First, I acquired the [lizfer.com](http://lizfer.com) domain on [Google Domains](http://domains.google.com) ("Why?" you ask. "Billing" is the answer) and set it to route to Digital Ocean's domains:

![image](https://user-images.githubusercontent.com/22382891/132896021-4408c902-1109-43de-b34d-2fc050d0ce88.png)

Then, I told Digital Ocean that whenever a request to lizfer.com arrives to their DNS, they should associate it with my instance. I could have done this directly on Google Domains, but doing it this way ensures that I can connect to my cloud Digital Ocean VPC and assess all of its benefits. This was executed by clicking **Create** > **Domains/DNS**.

![image](https://user-images.githubusercontent.com/22382891/132896508-3874f50f-bd7a-44fc-bd93-00547a5020f0.png)

![image](https://user-images.githubusercontent.com/22382891/132896362-4906edb4-40a2-4f62-aa46-f13c6d80e7a7.png)

### 5.c: CA configuration

This process was added because WordPress deals with personal data, which requires some TLS/SSL security. For the Certificate Authority, I chose [Let's Encrypt](https://letsencrypt.org/) because it automatizes most processes with Certbot and it easily integrates with Apache. A few more sub-steps:

- Installed the `certbot` and `python3-certbot-apache` (integration between Apache and Certbot) packages: `sudo apt install certbot python3-certbot-apache`;
- Used the Apache plugin for Certbot to set up authentication and installation: `sudo certbot --apache` 

Then, up to configuring Wordpress! 🚀


## Thanks
- [Digital Ocean](https://www.digitalocean.com/), for great documentation where I learned from and hosting my website
- [@groovemonkey (Dave Cohen)](https://github.com/groovemonkey) for his awesome [Udemy course](https://www.udemy.com/share/101Kb63@HcZhn7EZt1DD74Bm9iS3QcfMfMpwo27nepAzAPgEOdA2rfJqDb5HNVY3Xmk8BSuo/) and other content on [Youtube](https://www.youtube.com/c/tutoriaLinux)
