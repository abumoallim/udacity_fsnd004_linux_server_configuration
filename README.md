Project: Linux Server Configuration (nd004)
===========================================
This is part of the project submission under the Full Stack Nano Degree program of Udacity, authored by Ashish Nitin Patil. Kindly respect the [Udacity Honor Code](https://udacity.zendesk.com/hc/en-us/articles/210667103-What-is-the-Udacity-Honor-Code-).

[Live Demo](http://ec2-13-228-25-156.ap-southeast-1.compute.amazonaws.com/)
-----------

Project brief
-------------
The project briefly describes the steps followed for setting up a linux server that hosts the [item catalog application](https://github.com/ashishnitinpatil/udacity_fsnd004_item_catalog_application/)


Steps Followed
--------------
- Create a Ubuntu 16.04 instance on Amazon lightsail
    - Download the private pem file for accessing the instance (private.pem)
    - Change file permissions on the private key file to secure it  
      `chmod 600 /path/to/private.pem`
- SSH into the server (e.g. IP - 13.228.25.156)  
  Create a static IP & associate with our instance if necessary  
  `ssh -i /path/to/private.pem ubuntu@13.228.25.156`
- Update all packages  
  `sudo apt-get -y update && sudo apt-get -y upgrade`  
  (If grub popup shows up, just accept default setting (keep local installed))
- Configure Uncomplicated Firewall
    - Enable: `sudo ufw enable`
    - Default incoming : `sudo ufw default deny incoming`
    - Default outgoing : `sudo ufw default allow outgoing`
    - Enable HTTP ports: `sudo ufw allow www`
    - Enable  NTP port : `sudo ufw allow 123/tcp`
    - Add new SSH port : `sudo ufw allow 2200/tcp`
    - Check if it was done correctly: `sudo ufw status`
- Change SSH port
    - Edit config: `sudo vi /etc/ssh/sshd_config`  
      Change port to 2200, save & exit vi
    - Restart SSH daemon: `sudo service sshd restart`
    - Close the existing SSH connection
    - Allow the new port in instance networking settings (Firewall)
    - Test the new custom SSH port by logging in  
      `ssh -i /path/to/private.pem -p 2200 ubuntu@13.228.25.156`
- Give **grader** access
    - Add user: `sudo useradd grader`
    - Add to sudoers: `sudo usermod -aG sudo grader`
    - Allow passwordless sudo access:  
        - Edit sudoers - `sudo visudo -f /etc/sudoers`
        - Add privilege (before #include directives) - `grader  ALL=(ALL) NOPASSWD:ALL`
        - Save & exit visudo
        - Test access
            - `sudo su grader`
            - `sudo ls /etc/`
            - Exit
- Create an SSH keypair for grader
    - Switch to grader user: `sudo su grader`
    - Create reqd. directories: `mkdir /home/grader && mkdir /home/grader/.ssh`
    - Generate SSH keypair: `ssh-keygen -t rsa`
    - Allow login via new key: `sudo cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys`
- DateTime:
    - Enable NTP: `sudo apt-get install ntp`
    - Change timezone to UTC: `sudo timedatectl set-timezone Etc/UTC`
- Install Apache to serve a Python mod_wsgi application
    - `sudo apt-get install -y python3-pip apache2 libapache2-mod-wsgi-py3`
- Clone the application repo
    - `sudo apt-get install -y git`
    - `cd /var/www && sudo git clone https://github.com/ashishnitinpatil/udacity_fsnd004_item_catalog_application.git item_catalog_app`
- Setup virtualenv & install python dependencies
    - `python3 -m pip install virtualenv`
    - `cd /var/www/item_catalog_app`
    - `virtualenv -p python3 .venv`
    - `source .venv/bin/activate` # You should now get the .venv prompt
    - `pip install -r requirements.txt`
- Install & setup postgres
    - `sudo apt-get install -y postgresql postgresql-contrib`
    - Connect to psql: `sudo -u postgres psql`
    - Create user, DB & grant permissions:
        - `postgres=> CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';`
        - `postgres=> CREATE DATABASE catalog;`
        - `postgres=> GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
        - `postgres=> \q`
    - Test the connection via psycopg2
        - `python`
        - `>>> import psycopg2`
        - `>>> psycopg2.connect('postgresql://catalog:catalog@localhost/catalog')`
- Setup the project
    - Copy sample_secrets.json to secrets.json and edit the secrets  
      `SQLALCHEMY_DATABASE_URI: "postgresql://catalog:catalog@localhost/catalog"`
    - Edit the `REDIRECT_URI` in item_catalog_app/settings.py with correct IP
    - Activate virtualenv, and do `flask db upgrade`
- Setup Apache
    - Ensure mod_wsgi is enabled: `sudo a2enmod wsgi`
    - Create a item_catalog_app.wsgi file  
      See item_catalog_app.wsgi for contents
    - Create symlink to secrets: `sudo ln -s secrets.json /var/www/secrets.json`
    - `sudo vi /etc/apache2/sites-available/item_catalog_app.conf`  
      See apache.conf for file contents that should be put
    - Enable the site: `sudo a2ensite item_catalog_app`
    - Reload apache: `sudo service apache2 reload`


Licensing
---------
Please refer to [LICENSE](/LICENSE)
