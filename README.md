# Web app Hosted by Linux server
#### This is a guide for hosting a web app on a ubuntu server as a part of udacity Full-stack nanodegree.

The web app available on: http://www.52.2.6.25.xip.io

The Public IP: 52.2.6.25

The SSH port : 2200

## Setup Amazon Lightsail server:-

### Create ubuntu server:-
- Visit (https://lightsail.aws.amazon.com/) and Create new instance.
- Download the ssh private key from the Lightsail console.
- Connect using your own SSH client by using this command
````
ssh <your-user-name>@<server-public-IP-address> -p <port-number> -i <your-local-ssh-private-key-name>
````

### Update the server:-
- Update package lists (`sudo apt-get update`).
- Update all the installed packages (`sudo apt-get upgrade`).
- Change the Timezone to UTC (`sudo dpkg-reconfigure tzdata`) and select `None of the above` then select `UTC`.

### Setup the firewall:-
- First deny all the incoming connections (`sudo ufw default deny incoming`).
- Then allow all the outgoing connections (`sudo ufw default allow outgoing`).
- Allow the ssh connections (`sudo ufw allow ssh`).
- Allow the connections for the port 2200 (`sudo ufw allow 2200/tcp`).
- Allow the http connections (`sudo ufw allow www`).
- Allow the connections for the port 80 (`sudo ufw allow 80/tcp`).
- Allow the ntp (`sudo ufw allow ntp`).
- Allow the connections for the port 123 (`sudo ufw allow 123/udp`).
- Enable the firewall by (`sudo ufw enable`)
- View the firewall status (`sudo ufw status`)

### Change the SSH port:-
- Open the SSH Configuration file (`sudo nano /etc/ssh/sshd_config`)
- Search for the Port and change it from `22` to `2200`.
- Add the prot on the firewall as custom in Lightsail console.
- Restart the ssh to apply the change (`sudo service ssh restart`).

### Add a new user to the server:-
- Create a new user (`sudo adduser <username>`)
- give the sudo access to the new user (`sudo nano /etc/sudoers.d/<username>`) and write (`grader ALL=(ALL:ALL) ALL`) inside the file and save it.

### Generating the public key:-
- On your local computer run the command (`ssh-keygen`).
- use the same provided path to store your key, give it a name and it will generate two file.
- Login to your new user by (` sudo login <username>`).
- Create new directory on your home (`mkdir .ssh`).
- Create a new file on .ssh (`touch .ssh/authorized_keys`).
- Open your file and paste the content of your <public_key>.pub file from your local computer by (`nano .ssh/authorized_keys`).
- Change the permissions for the .ssh directory (`chmod 700 .ssh`).
- Change the permissions for the authorized_keys file (`chmod 644 .ssh/authorized_keys`).
- Restart the ssh to apply the change (`sudo service ssh restart`).

### Disable the root access:-
- Open the SSH Configuration file (`sudo nano /etc/ssh/sshd_config`)
- Search for the section `Authentication`.
- Change the `PermitRootLogin` to `no`.
- Restart the ssh to apply the change (`sudo service ssh restart`).

## Setup the web application server:-

### Setup Apache HTTP Server:-
- Install Apache (`sudo apt-get install apache2`).
- Install mod_wsgi to handle the python file (`sudo apt-get install libapache2-mod-wsgi`).

### Setup the PostgreSQL database server:-
- Access root by (`sudo su -`).
- Install PostgreSQL (`apt-get install postgresql postgresql-contrib`).
- Configure PostgreSQL to start up upon server boot(`update-rc.d postgresql enable`).
- Start PostgreSQL (`service postgresql start`).
- Log into the postgres user (`su - postgres`).
- Log into the database by typing (`psql`).
- Write these lines line-by-line :
````
CREATE USER <app_name> WITH PASSWORD 'password';
ALTER USER <app_name> CREATEDB;
CREATE DATABASE <app_name> WITH OWNER <app_name>;
\c <app_name>
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO <app_name>;
\q
exit
````
- Exit the root by (`exit`).

## Setup your app files:-

### Clone your app from Github:-
- Install git by (`sudo apt-get install git`).
- go the public directory (`cd /var/www`).
- Clone your app here and give it a name (`git clone <LINK_FOR_APP_REPOSETORY> <app_name>`).
- Create a <app_name>.wsgi file (`sudo nano <app_name>.wsgi`) and inside it write:
````
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/<app_name>/")
from <app_name> import app as application
application.secret_key = 'super_secret_key'
WTF_CSRF_ENABLED = True
````
- Go to the <app_name> directory (`cd <app_name>`).
- Rename <app_name>.py to init.py (`sudo mv <app_name>.py __init__.py`).

### Install virtual environment
- Install pip by (`sudo apt-get install python-pip`).
- Install the virtual environment by (`sudo pip install virtualenv`).
- Create a new virtual environment by (`sudo virtualenv venv`).
- Activate the virtual environment by  (`source venv/bin/activate`).
- Give the permissions (`sudo chmod -R 777 venv`).

### Install Flask and other dependencies
- Install Flask (`pip install Flask`).
- Install other project dependencies (`sudo pip2 install httplib2 oauth2client sqlalchemy flask-sqlalchemy psycopg2-binary bleach requests sqlalchemy_utils`).

### Update path of client_secrets.json file
- Open the app file by (`sudo nano __init__.py`).
- Change client_secrets.json path to `/var/www/<app_name>/client_secrets.json`

### Configure and enable a new virtual host
- Configure Apache to handle the app requests using the WSGI module by (`sudo nano /etc/apache2/sites-enabled/000-default.conf`) with this lines:
````
<VirtualHost *:80>
    ServerName <your_public_ip>
    ServerAlias http://<your_public_ip>
    ServerAdmin <admin>@<your_public_ip>
    WSGIDaemonProcess <app_name> python-path=/var/www/<app_name>:venv/lib/python2.7/site-packages
    WSGIProcessGroup <app_name>
    WSGIScriptAlias / /var/www/<app_name>/<app_name>.wsgi
    <Directory /var/www/<app_name>/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/<app_name>/static
    <Directory /var/www/<app_name>/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
````

### Launch your database:-
- Change create engine line in your `__init__.py` , `database_setup.py` and `seeder.py` to: `engine = create_engine('postgresql://<app_name>:password@localhost/<app_name>')`.
- Launch your database_setup file by (`sudo python database_setup.py`).
- Launch your seeder file by (`sudo python seeder.py`).

- Restart the apache2 (`sudo apache2ctl restart`).

## Resources:-
[- Udacity FSND ](https://mena.udacity.com/course/full-stack-web-developer-nanodegree--nd004)
[- Ask ubuntu ](https://askubuntu.com/)
[- ubuntu forums ](https://ubuntuforums.org/index.php)
[- mod_wsgi ](https://modwsgi.readthedocs.io/en/develop/)
[- mod_wsgi (Apache)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
[- Amazon EC2 Linux Instances ](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
[- Amazon Lightsail ](https://lightsail.aws.amazon.com/ls/webapp/home/instances)
[- xip ](http://xip.io/)
[- The Bookshelf ](https://github.com/KAsiri/TheBookshelf)
