# P5_Configuring_Linux_Servers

A baseline installation of Ubuntu Linux on a virtual machine to host a Flask web application. This includes the installation of updates, securing the system from a number of attack vectors and installing/configuring web and database servers.

IP address: 35.165.0.82
URL: http://ec2-35-165-0-82.us-west-2.compute.amazonaws.com/

**Note:** The below step-by-step walkthrough is the solution to Project 5 of the [Udacity Full Stack Web Developer Nanodegree][1] and deploys a modified version of the [solution of project 3][2] on the virtual machine.

## Issues
* The web application will need to be shifted to a new server to be reachable when the AWS-server instance is no longer accessible.

## Step by Step Walkthrough
See below for the steps required

### 1 - Create Development Environment: Launch Virtual Machine and SSH into the server
Source: [Udacity][3]  

1. Create new development environment.
2. Download private keys and note public IP address: in my case, 35.165.0.82
3. Move the private key file into the folder ~/.ssh:  
  `$ mv ~/Downloads/udacity_key.rsa ~/.ssh/`
4. Set file rights (only owner can write and read.):  
  `$ chmod 600 ~/.ssh/udacity_key.rsa`
5. SSH into the instance:  
  `$ ssh -i ~/.ssh/udacity_key.rsa root@35.165.0.82 -o ServerAliveInterval=5 -v`
  `-o ServerAliveInternal=5` keeps login active by writing a packet every 5 seconds
  `-v` shows login details in the console for debugging purposes

### 2 - User Management: Create a new user named `grader` and give user `grader` "sudo" permission   

1. Create a new user:  
  `$ sudo adduser grader`
2. Give new user the permission to sudo
  1. Open the sudo configuration:  
    `$ visudo`
  2. Add the following line below `root ALL...` to give the new user sudo access:  
    `grader ALL=(ALL:ALL) ALL`
    Note that the space between `grader` and `ALL=(ALL:ALL) ALL` is a tab.

### 3 - From local machine, generate SSH keys
	1. From your local machine -- and not from your server
	`$ ssh-keygen`
	take a note of your filenames.  In my case:
	Your identification has been saved in /Users/adamstober/.ssh/P5Try4
	2. Set file permissions to "only owner can write and read"
	`$ chmod 600 ~/.ssh/P5Try4
	3. Copy keys from local machine to local file for future use
	`$ cat ~/.ssh/P5Try4.pub`

### 4 - Prepare `grader` to accept SSH connection
	1. Switch to "grader" user
	`$ su grader`

	2. As grader, create .ssh folder in /home/grader
	`$ mkdir ~/.ssh`

	3. As grader, create empty authorized_keys file
	`$ touch ~/.ssh/authorized_keys`

	4. As grader, paste public keys into authorized_keys
	`$ nano ~/.ssh/authorized_keys`
	[Paste contents of public key]

	5. You should now be able to login as grader in new terminal window
  	`$ ssh -i ~/.ssh/P5Try4 grader@35.165.0.82 -o ServerAliveInterval=5 -v`

### 5 - Update and upgrade all currently installed packages
  	
	1. Update the list of available packages and their versions:  
	  `$ apt-get update`
	2. Install newer vesions of packages.  Do not over-write sudo or local files:  
	  `$ apt-get upgrade`
	3. Check timezone is UTC:
	  `$ date +'%:z %Z'`
	  If not UTC, update to UTC.

### 6 - Change the SSH port from 22 to 2200

Login as root:  

1. open the config file and change 22 to 2200
`$ nano /etc/ssh/sshd_config`

2. reload ssh
`$ reload ssh`

3. WHILE STILL LOGGED IN: In new window, test login for grader
`$ ssh -i ~/.ssh/P5Try4 grader@35.165.0.82 -o ServerAliveInterval=5 -v -p 2200

4. WHILE STILL LOGGED IN: In new window, test login for root
`$ ssh -i ~/.ssh/udacity_key.rsa root@35.165.0.82 -o ServerAliveInterval=5 -v -p 2200`

### 7 - Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

1. While still logged in as root, login as grader in new terminal tab
	1. *Check the status of UFW:  
	  `$ sudo ufw status verbose`
	2. Allow incoming TCP packets on port 2200 (SSH):  
	  `$ sudo ufw allow 2200/tcp` 
	3. Allow incoming TCP packets on port 80 (HTTP):  
	  `$ sudo ufw allow 80/tcp` 
	4. Allow incoming UDP packets on port 123 (NTP):  
	  `$ sudo ufw allow 123/udp`
	5. Turn UFW on:  
	  `$ sudo ufw enable`

2. WHILE STILL LOGGED IN: In new window, test login for grader
`$ ssh -i ~/.ssh/P5Try4 grader@35.165.0.82 -o ServerAliveInterval=5 -v -p 2200

3. WHILE STILL LOGGED IN: In new window, test login for root
`$ ssh -i ~/.ssh/udacity_key.rsa root@35.165.0.82 -o ServerAliveInterval=5 -v -p 2200`

### 8 - Install and configure Apache to serve a Python mod_wsgi application)

	1. Install Apache
		`$ sudo apt install apache2`
	2. Install mod_wsgi
		`$ sudo apt install libapache2-mod-wsgi`
	4. Restart Apache
		`$ sudo apache2ctl restart`

### 9 - Install and Configure PostgreSQL with 'Catalog' DB and 'Catalog' User

	1. Install PostgreSQL
	`$ sudo apt install postgresql`
  2. As root
  `$ # su - postgres`
  3. Connect to database server:
  `$ psql template1`
  4. Create user
  `CREATE USER catalog WITH PASSWORD '[DAS PASSWORD]';`
  5. Create database catalog:
  `CREATE DATABASE catalog;`
  6. Grant catalog DB privileges to 'catalog' user
  `GRANT ALL PRIVILEGES ON DATABASE catalog to catalog;`
  7. Create user
  `CREATE USER "www-data";`
  8. Redirect environment to postgresql DB. From (venv) catalog@ip-10-20-56-96:
  `export DATABASE_URL='postgresql://catalog:[DAS PASSWORD]@localhost/catalog'`


### 10 - Create New Linux User "catalog"

	1. Create a new user:  
	`$ sudo adduser catalog`
	2. As r00t, be l33t and login as postgres, the user that was created on the remote system when installing postgres:
	`$ su - postgres`
	3. As postgres, create user "catalog".  Instructions available here: https://www.postgresql.org/docs/9.1/static/app-createuser.html
	`$ createuser catalog`
	4. As postgres, check that user catalog was created
	`$ psql`
	5. In Postgres, view the user catalog
	`# \du`
	6. Logout of Postgres
	`# \q`

### 11 Put Program on Github and clone repo to Server
	Locally:
  1. Make sure client_secrets files don't get to GitHub by "ignoring"
  2. Publish program to github: https://github.com/AdamStober/P3_item_catalog_for_P5
  On server:
  1. As grader, install GIT
	`$ sudo apt-get install git-all`
	From: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
  2. As catalog, clone repo.  Become root to become catalog
  `$ sudo -i`
  3. As root, become catalog
  `$ su - catalog`
  4. cd /home/catalog/app.  Clone repo
  `$ git clone https://github.com/AdamStober/P3_item_catalog_for_P5.git`
  5. rename folder to "app"
  `$ mv P3_item_catalog_for_P5/ app`

### 12 Put client secrets on the server
  1. From local terminal, move client secrets files to the server:
  `scp -i ~/.ssh/P5Try4 -P 2200 client_secrets.json grader@35.165.0.82:~/`
  `scp -i ~/.ssh/P5Try4 -P 2200 fb_client_secrets.json grader@35.165.0.82:~/`
  2. As root, move files to application directory. Navigate to /home/catalog/app and run:
  `$ mv /home/grader/*.json .`

### 13 More Apache, and create wsgi file locally to put in repository
References:
[Digital Ocean][4]
[TrevOps][5]

  1. Install mod_wsgi
  `$ sudo apt-get install libapache2-mod-wsgi python-dev`
  2. Enable mod_wsgi
  `$ sudo a2enmod wsgi` --> "module already enabled"
  3. Prepare to automatically keep track of changes to /etc
  `$ sudo apt install etckeeper`
  4. Edit /etc/etckeeper/etckeeper.conf so that git is used instead of bzr
  `$ sudo nano /etc/etckeeper/etckeeper.conf`
    1. uncomment: #VCS="git" --> VCS="git"
    2. comment VCS="bzr" --> #VCS="bzr"
  5. Begin running etckeeper
  `$ sudo etckeeper init`
 
### 14: Install Flask
  1. If pip is not installed, install it on Ubuntu through apt-get
    `$ sudo apt-get install python-pip`
  2. If virtualenv is not installed, use pip to install it using following command:
    `$ sudo pip install virtualenv`
  3. From home/catalog/app: name your temporary virtual environment:
    `$ sudo virtualenv venv`
  4. Install Flask in that environment by activating the virtual environment with the following command:
    `$ source venv/bin/activate`
  5. Give this command to install Flask inside:
    `$ sudo pip install Flask`
  6. Run the following command to test if the installation is successful and the app is running:
    `$ python database_setup.py` 
    It should display “Running on http://localhost:5000/” or "Running on http://127.0.0.1:5000/". If you see this message, you have successfully configured the app.
  7. To deactivate the environment, give the following command:
    `deactivate`

### 15: Update FlaskApp.conf
1. As root
`nano /etc/apache2/sites-enabled/FlaskApp.conf`

2. Add the following lines:
`WSGIScriptAlias / /home/catalog/app/flaskapp.wsgi
`SetEnv DATABASE_URL postgresql://catalog:[DAS PASSWORD]@localhost/catalog

So:

<VirtualHost *:80>
                ServerName adam
                ServerAdmin adam@adamstober.com

                DocumentRoot /home/catalog/app
                WSGIScriptAlias / /home/catalog/app/flaskapp.wsgi
                SetEnv DATABASE_URL postgresql://catalog:[DAS PASSWORD]@localhost/catalog
                Alias /static /home/catalog/app/static

                ErrorLog ${APACHE_LOG_DIR}/restaurants_error.log
                CustomLog ${APACHE_LOG_DIR}/restaurants_access.log combined

                #LogLevel info

                <Directory /home/catalog/app>
                        Order deny,allow
                        Allow from all
                        Require all granted
                </Directory>

                <Directory /home/catalog/app/static>
                        Options Indexes FollowSymLinks
                        Order deny,allow
                        Allow from all
                        Require all granted
                </Directory>
</VirtualHost>


### 15: Update FlaskApp.wsgi

1. From /home/catalog/app:
`$ nano flaskapp.wsgi`

So:
#!/usr/bin/python
import sys
import logging
import site
import os

logging.basicConfig(stream=sys.stderr)

os.environ['DATABASE_URL'] = 'postgresql://catalog:[DAS PASSWORD]@localhost/catalog'

# Add the site-packages of the chosen virtualenv to work with
site.addsitedir('/home/catalog/app/venv/lib/python2.7/site-packages')
activate_env='/home/catalog/app/venv/bin/activate_this.py'
execfile(activate_env, dict(__file__=activate_env))

sys.path.insert(0,"/home/catalog/app/")

os.chdir("/home/catalog/app/")

from project import app as application
application.secret_key = 'super_secret_key'

### 16 Remove login of the root user:
1. open the config file and set PermitRootLogin to "no"
`$ nano /etc/ssh/sshd_config`

2. reload ssh
`$ reload ssh`

3. WHILE STILL LOGGED IN: In new window, test login for grader
`$ ssh -i ~/.ssh/P5Try4 grader@35.165.0.82 -o ServerAliveInterval=5 -v -p 2200

4. WHILE STILL LOGGED IN: In new window, test login for root
`$ ssh -i ~/.ssh/udacity_key.rsa root@35.165.0.82 -o ServerAliveInterval=5 -v -p 2200`

### 17 Smile

:)

#### References
[1]: https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004 "Full Stack Web Developer | Udacity Nanodegree"
[2]: https://github.com/AdamStober/fullstack-nanodegree-vm/tree/master/vagrant/item_catalog_final "GitHub repository of item catalog web app"
[3]: https://www.udacity.com/account#!/development_environment "Instructions for SSH access to the instance"
[4]: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps "How to deploy a flask application on an ubuntu VPS"
[5]: https://twitter.com/trevorjay "@trevorjay aka TrevOps aka Trev @Disqus aka code-beast-genius-who-needs-no-sleep"
