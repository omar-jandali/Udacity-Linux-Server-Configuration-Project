# Udacity Linux Server Configuration Project

List of conguration changes made, and shell commands used to make the changes.

### Server Details

IP address: `52.33.95.129`

SSH port: `2200`

URL: `http://52.33.95.129`

### Change timezone to UTC

`timedatectl set-timezone UTC`

### Update all currently installed packages

```
apt-get update
apt-get upgrade
```

### Change the SSH port from 22 to 2200

`nano /etc/ssh/sshd_config`
Change `Port 22` to `Port 2200` save and exit

### Configure the Uncomplicated Firewall

```
ufw allow 2200/tcp
ufw allow 80/tcp
ufw allow 123/udp
ufw enable
```

### Install Apache to serve a Python mod_wsgi application

```
apt-get install apache2
apt-get install libapache2-mod-wsgi
```

### Install and configure PostgreSQL

```
apt-get install postgresql
-u postgres createuser -P catalog
-u postgres createdb -O catalog catalog
```

### Install git, clone and setup the Catalog App project

```
apt-get install git
cd /var/www
mkdir catalog
git clone https://github.com/vk9141/udacity-item-catalog.git
mv ./udacity-item-catalog ./catalog
cd catalog
mv project.py catalog.py
```

Edit `database_setup.py`, `catalog.py` and `database_initial_data.py` and change `engine = create_engine('sqlite:///categoryrecipeswithusers.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`

### Install Flask, SQLAlchemy, etc

```
apt-get install python-psycopg2 python-flask
apt-get install python-sqlalchemy python-pip
pip install oauth2client
pip install requests
pip install httplib2
```

### Create the .wsgi File

```
cd /var/www/catalog
nano catalog.wsgi
``` 

Copy this code and save into the file:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from FlaskApp import app as application
application.secret_key = 'super_secret_key'
```

### Configure and Enable a New Virtual Host

`nano /etc/apache2/sites-available/catalog.conf`

Copy this code and save into the file:

```
<VirtualHost *:80>
    ServerName 52.33.95.129
    ServerAdmin vk9141@hotmail.com
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Enable site:
`a2ensite catalog`

### Restart Apache

`service apache2 restart`

### Add super user grader

```
useradd -m -s /bin/bash grader
usermod -aG sudo grader
```

### Set-up SSH keys for user grader

```
mkdir /home/grader/.ssh
chown grader:grader /home/grader/.ssh
chmod 700 /home/grader/.ssh
cp /root/.ssh/authorized_keys /home/grader/.ssh/
chown grader:grader /home/grader/.ssh/authorized_keys
chmod 644 /home/grader/.ssh/authorized_keys
```

Can now login as the grader user using the command: `ssh -i [privateKey] grader@52.33.95.129`

### Disable root login

`nano /etc/ssh/sshd_config`

Change the following line `PermitRootLogin without-password` to `PermitRootLogin no`


### References

Many thanks to the following github contributors for their helpful walkthroughs:

[https://github.com/kongling893/Linux-Server-Configuration-UDACITY](https://github.com/kongling893/Linux-Server-Configuration-UDACITY)
[https://github.com/SteveWooding/fullstack-nanodegree-linux-server-config](https://github.com/SteveWooding/fullstack-nanodegree-linux-server-config)




