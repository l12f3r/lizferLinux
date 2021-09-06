# lizferLinux
Provisioning of a website implementation on cloud based on a server with LAMP (Linux, Apache2, MySQL and PHP). Check it at http://188.166.90.196.

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

## 2. Apache2 configuration

Configuring the "A" in LAMP was a handful of work. From this moment onwards, I decided to authenticate using the PID 1000 account (lizfer) for testing reasons - hence why lots of `sudo` will appear. 

### 2.a: Downloading

The first step was getting Apache2 from Ubuntu's repos using `sudo apt install apache2`. Just had to accept the installation terms.

### 2.b: Firewall settings

By downloading Apache, UFW automatically creates profiles for Apache based on ports available for access. By entering `sudo ufw app list`, the list of available applications was prompted on the stdout (tl;dr: **Apache** profile opens only port 80, **Apache Full** opens both port 80 and port 443 and **Apache Secure** opens only port 443):

    Available applications:
      Apache
      Apache Full
      Apache Secure
      OpenSSH

As there was no TLS/SSL certificate configured, I allowed communication in port 80 only by enabling the **Apache** profile: `sudo ufw allow in "Apache"`.
