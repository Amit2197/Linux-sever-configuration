# LINUX SERVER CONFIGURATION
*udacity linux server configuration*  
```
By: Amit kumar
```
# TABLE OF CONTENT
&emsp;[&#x261B;Introduction](#introduction)  
&emsp;[&#x261B;Server](#server-info)  
&emsp;[&#x261B;Getting Started](#getting-started)  
&emsp;[&#x261B;Installation](#installation)

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
  $ $ ssh root@139.59.67.159 -p 2200
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

<details>
<summary>Configure Apache to serve a Python mod_wsgi application
Clone the Udacity-item-catalog app from Github</summary>

+ Run following command:
  ```
  $ cd /var/www
  $ sudo mkdir catalog
  $ sudo chown -R grader:grader catalog
  $ cd catalog
  $ git clone https://github.com/Amit2197/Linux-sever-configuration.git
  ```
+ To make .git directory is not publicly accessible via a browser, create a .htaccess file in the .git folder and put the following in this file:
  ```
  RedirectMatch 404 /\.git
  ```

</details>

<details>
<summary>Install pip , setup virtual environment and also install Flask and its dependencies:</summary>

+ Run following command:
  ```
  $ sudo apt-get install python-pip
  $ sudo pip install virtualenv
  $ sudo virtualenv venv
  $ source venv/bin/activate(Rest of the commands should be excecuted in the virtual environment)
  $ sudo chmod -R 777 venv
  $ sudo pip install -r catalog/requirements.txt
  ```

+ Install Python's PostgreSQL adapter psycopg2:
  ```
  $ sudo apt-get install python-psycopg2
  ```

+ Configure and Enable a New Virtual Host
  ```
  $ sudo nano /etc/apache2/sites-available/catalog.conf
  ```
+ Add the following content:
  ```
  <VirtualHost *:80>
    ServerName 139.59.67.159
    ServerAlias 139.59.67.159.xip.io
    ServerAdmin grader@139.59.67.159
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/Item_Catalog_Application/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/Item_Catalog_Application/static
    <Directory /var/www/catalog/Item_Catalog_Application/static/>
       Order allow,deny
       Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
+ Enable the new virtual host:
  ```
  $ sudo a2ensite catalog
  ```
+ Create and configure the .wsgi File
  ```
  $ cd /var/www/catalog/
  $ sudo nano catalog.wsgi
  ```

+ Add the following content:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")
  from Item_Catalog_Application import app as application
  application.secret_key = 'secret'
  ```

</details>

<details>
<summary>Installing and Configuring PostgreSQL</summary>

+ Install some necessary Python packages for working with PostgreSQL:
  ```
  $ sudo apt-get install libpq-dev python-dev.
  ```

+ Install PostgreSQL:
  ```
  $ sudo apt-get install postgresql postgresql-contrib
  ```

+ connect to the database by using postgres username with:
  ```
  $ sudo -u postgres psql
  ```

+ Create a new user called 'catalog' with his password:
  ```
  # CREATE USER catalog WITH PASSWORD 'catalog';
  ```
+ Give catalog user the CREATEDB permission:
  ```
  # ALTER USER catalog CREATEDB;
  ```
+ Create the 'catalog' database owned by catalog user:
  ```
  # CREATE DATABASE catalog WITH OWNER catalog;
  ```
+ Connect to the database:
  ```
  # \c catalog
  ```
+ Revoke all the rights:
  ```
  # REVOKE ALL ON SCHEMA public FROM public;
  ```
+ Lock down the permissions to only let catalog role create tables:
  ```
  # GRANT ALL ON SCHEMA public TO catalog;
  ```
+ Log out from PostgreSQL:
  ```
  # \q
  ```
+ Edit the moredata.py and database.py file:
+ Change engine = create_engine('sqlite:///menu.db') to engine = create_engine('postgresql://catalog:catalog@localhost/catalog')

</details>

<details>
<summary>Change file name and path:</summary>

+ change file name application.py to &#95;&#95;init&#95;&#95;.py
+ change path client_secrets.json to /var/www/catalo/Item_Catalog_Application/client_secrets.json in &#95;&#95;init&#95;&#95;.py

</details>

<details>
<summary> Update Google login :</summary>

+ Go to Google Dev Console
+ Sign up or Login if prompted
+ Go to Credentials
+ Select Create Crendentials > OAuth Client ID
+ Select Web application
+ Enter name 'Item-Catalog'
+ Authorized JavaScript origins = 'http://139.59.67.159.xip.io'
+ Authorized redirect URIs = 'http://139.59.67.159.xip.io/login' && 'http://139.59.67.159.xip.io/gconnect'
+ Select Create
+ Copy the Client ID and paste it into the data-clientid in login.html
+ On the Dev Console Select Download JSON
+ Rename JSON file to client_secrets.json
+ Place JSON file in Item_Catalog_Application directory that you cloned from here.

</details>

<details>
<summary>Restart Apache to launch the app:</summary>

 ```
 $ sudo service apache2 restart
 ```

 </details>

## Debugging

If you are getting an _Internal Server Error_ or any other error(s), make sure to check out Apache's error log for debugging:

```
$ sudo cat /var/log/apache2/error.log
```
Make sure  you restart the server when you make any changes to your application.


First try creating a simple Flask app and see if the app is working to ensure that your configuration is right, then clone in your repository and start working on it. If there is an error after after cloning then the problem will mostly be with your cloned app.

## References

[1] <https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04>

[2] <http://terokarvinen.com/2016/deploy-flask-python3-on-apache2-ubuntu>

[3] <https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps>

[4] <https://askubuntu.com/questions/293426/system-monitoring-tools-for-ubuntu

[5] <https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps>

[6] <https://www.digitalocean.com/community/questions/can-i-create-a-clone-from-a-dropplet>

[7] <http://blog.dscpl.com.au/2014/09/using-python-virtual-environments-with.html>
