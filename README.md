# LINUX SERVER CONFIGURATION
*udacity linux server configuration*  
```
By: Amit kumar
```
# TABLE OF CONTENT
&emsp;[&#x261B;Intro](introduction)  
&emsp;[&#x261B;Server](server-info)  
&emsp;[&#x261B;Getting Started](getting-started)  
&emsp;[&#x261B;Installation](installation)

# Introduction
&emsp;&emsp;This project take a baseline installation of a Linux server and prepare it to host Web applications.  
&emsp;&emsp;In this project, I have set up an Ubuntu 18.04 image on a DigitalOcean droplet. The technical details of the server as well as the steps that have been taken to set it up can be found in the succeeding sections.
# Server Info
&emsp;**IP address&emsp;&nbsp;:**&emsp;139.59.67.159  
&emsp;**SSH port&emsp;&emsp;:**&emsp;80  
&emsp;**URL&emsp;&emsp;&emsp;&emsp;&nbsp;:**&emsp;[http://139.59.67.159.xip.io](http://139.59.67.159.xip.io)  
# Getting Started
<details>
<summary>Creating the RSA Key Pair</summary>

+ make directory and move:  
  ```console
   $ mkdir ~/.ssh
   $ cd ~/.ssh
   ```
+ To generate a key pair, run the following command:  
  ```console
   $ ssh-keygen
   ```
+ The whole process would look like this:
  ```
  Generating public/private rsa key pair.
  Enter file in which to save the key (/c/Users/GANAVI-PC/.ssh/id_rsa): item
  Enter passphrase (empty for no passphrase):
  Enter same passphrase again:
  Your identification has been saved in item.
  Your public key has been saved in item.pub.
  The key fingerprint is:
  SHA256:GgXdnbdJ7h6vfYt0Q9OmhfQN1R7oL8Axu+cn1TSmwgk
  ```

</details>

<details>
<summary>Setting Up a DigitalOcean Droplet</summary>

+ Log in or create an account on [DigtalOcean](https://cloud.digitalocean.com/login).
+ Go to the Dashboard, and click **Create Droplet**.
+ Choose **Ubuntu 18.04 x64** image from the list of given images.
+ Choose a preferred size. In this project, I have chosen the **1GB/1 vCPU/25GB** configuration.
+ In the section **Add Your SSH Keys**, paste the content of your public key, `item.pub`:
+ Click **Create** to create the droplet. This will take some time to complete. After the droplet has been created successfully, a public IP address will be assigned. In this project, the public IPv4 address that I have been assigned is `139.59.67.159`.

</details>

<details>
<summary>Working on `root` and `grader` in host machine.</summary>

+ you can now log into the server as `root`
  ```console
  ssh root@139.59.67.159
  ```
+ Updating the System
  ```
  # apt update && apt upgrade
  ```
+ Open the `/etc/ssh/sshd_config` file with `nano` or any other text editor of your choice:
  ```
   # nano /etc/ssh/sshd_config
   ```
+ Restart the SSH server to reflect those changes:
   ```
   # service ssh restart
   ```
+ To configure the timezone to use UTC, run the following command:

  ```
  # dpkg-reconfigure tzdata
  ```
+ Creating the User `grader`
  ```
  # adduser grader
  ```
+ Adding `grader` to the Group `sudo`
  ```
  # usermod -aG sudo grader
  ```
+ Run the following command also to eliminate errors.
  ```
  $ sudo nano /etc/sudoers.d/grader
  ```
  then add ```grader ALL=(ALL:ALL) ALL``` to the file then save and quit.
+ log into the account of the user `grader` from your virtual server:
```
# su - grader
```
+ Now enter the following commands to allow SSH access to the user `grader`:

```
$ mkdir .ssh
$ chmod 700 .ssh
$ touch .ssh/authorized_keys
$ nano .ssh/authorized_keys
```
after paste public key from ~/.ssh/item.public
+ restrict the access for other users
```
$ chmod 644 authorized_keys
```

</details>

<details>
<summary>Disabling Root Login</summary>

+ Run following command:
```
$ exit
$ $ ssh root@159.65.151.202 -p 2200
# nano /etc/ssh/sshd_config
# service ssh restart
```

</details>

<details>
<summary>Setting and Enable Up the Firewall</summary>

+ Run following command:
```
# sudo ufw allow 2200/tcp
# sudo ufw allow 80/tcp
# sudo ufw allow 123/udp
# sudo ufw enable
```

</details>

<details>
<summary>Setup to install apache2,mod_wsgi and git</summary>

+ Run following command:
```
$ sudo apt-get update
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo a2enmod wsgi
$ sudo service apache2 start
$ sudo apt-get install git
```

</details>

# Installation
