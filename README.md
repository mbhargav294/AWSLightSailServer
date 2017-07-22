# Deploying to Amazon Web Services (Light Sail)

### Connect to the server via SSH using following configuration:
- Public IP address: `34.227.58.96`
- SSH Port No.: `2200`


### Catalog Application is hosted at the following web address:
- `http://ec2-34-227-58-96.compute-1.amazonaws.com/`

### Summary of Operations performed to setup the server:

#### Creating a LightSail Instance and Loging in
 1. Create an Amazon AWS account and go to Amazon LightSail @ `https://lightsail.aws.amazon.com/ls/webapp/home/resources`
 2. Create a new ubuntu instance in LightSail and wait for it to boot.
 3. Download the public key present in Amazon LightSail Account and place it in ~/.ssh/ directory
 4. Now ssh to this from the terminal (Git Bash preferable) using the command `ssh -i ~/.ssh/LightSailDefaultPrivateKey.pem ubuntu@<ip-address>`
 5. If all the above steps are performed correctly you will see linux prompt: `ubuntu@ip-<local-ip>:~$`

#### Changing the default ssh port to 2200
 1. **The first rule of changing ssh port number is: talk about it with your machine**
 2. First make sure you allow port 2200 on your firewall
 3. run the following commands to configure your firewall
 ```
 sudo ufw default deny incoming
 sudo ufw default allow outgoing
 sudo ufw allow ssh
 sudo ufw allow 2200/ufw
 sudo ufw alow www
 sudo ufw allow ntp
 ```
 4. Now change the default ssh port from 22 to 2200, edit the `sshd_config` file present at `/etc/ssh/sshd_config` using nano or vim.
 5. Change the default port from 22 to 2200 in the line that says `Port 22` and at the end of this file add `AllowUsers grader`
 6.

#### Keep your machine up-to-date
 1. First run `sudo apt-get update` to update the list of available packages.
 2. Then run `sudo apt-get upgrade` to download all the updates and install them.

#### Adding a new user
 1. Once you successfully logged in to your ubuntu machine, you need to create a new user and give sudo access to that user.
 2. Use the command `sudo adduser grader` to create a new user named grader.
 3. Now give sudo permissions to this new user by editing the sudoers file at `/etc/sudoers` using vim/nano
 4. Find the line `root ALL=(ALL:ALL) ALL` and add `grader ALL=(ALL:ALL) ALL` right below this line.

#### Creating a ssh key
 1. Now get out of the remote machine and in your personal computer, open terminal (preferably Git Bash) and run `ssh-keygen`.
    - **Do not enter any password leave it blank**
    - **Give the default path `~/.ssh/<key-name>`**

 2. Get back to your remote machine and create a new folder .ssh in the `grader` user's home directory using the following list of commands:
 ```
 su - grader
 mkdir .ssh
 nano .ssh/authorized_keys
 ```
 3. Copy and paste the contents of `<key-name>.pub` from your personal machine placed at `~/.ssh/<key-name>.pub` and paste them in the `.ssh/authorized_keys` file.
 4. Now change the file permissions for this authorized_keys file using the commands:
 ```
 chmod 700 .ssh
 chmod 644 .ssh/authorized_keys
 ```
 5. After performing these actions you can only connect using the private key available in your personal machine.
 `ssh -i ~/.ssh/<key-name> grader@<public-ip-address> -p 2200`

#### Configuring the Time Zone
 1. Run the following command:
 ```
 sudo dpkg-reconfigure tzdata
 ```
 2. Change the time zone to `UTC (Other->UTC)`

#### Installing and setting up PostgreSQL, Git
 1. Install Git using `sudo apt-get install git`
 2. Install postgreSQL using `sudo apt-get install postgresql`
 3. Now postgreSQL is installed you need to set it up for our user to create and manipulate databases. Use the following command to do this:
 ```
 sudo -u postgres psql -c 'alter user grader with createdb' postgres
 sudo createdb catalog
 ```

#### Installing python & packages
 1. First we need to install python and pip using `apt-get`. Use the following 4 commands to install python:
 ```
 sudo apt-get install python
 sudo apt-get install python-pip python-dev build-essential
 sudo pip install --upgrade pip
 sudo pip install --upgrade virtualenv
 ```
 2. Now verify python installation by typing python at the prompt.
 3. Now that the python is installed, we need to install packages required for setting up the Flask Application.
 4. Following are the list of packages that are installed on my machine for running the Catalog Application.
 ```
 bleach (2.0.0)
 certifi (2017.4.17)
 chardet (3.0.4)
 click (6.7)
 Flask (0.9)
 Flask-HTTPAuth (3.2.3)
 Flask-Login (0.1.3)
 Flask-SQLAlchemy (2.2)
 get (0.0.21)
 html5lib (0.999999999)
 httplib2 (0.10.3)
 idna (2.5)
 itsdangerous (0.24)
 Jinja2 (2.9.6)
 MarkupSafe (1.0)
 oauth2client (4.1.2)
 packaging (16.8)
 passlib (1.7.1)
 pip (9.0.1)
 post (0.0.13)
 psycopg2 (2.7.1)
 public (0.0.38)
 pyasn1 (0.2.3)
 pyasn1-modules (0.0.9)
 pyparsing (2.2.0)
 query-string (0.0.12)
 redis (2.10.5)
 request (0.0.13)
 requests (2.18.1)
 rsa (3.4.2)
 setupfiles (0.0.50)
 setuptools (20.7.0)
 six (1.10.0)
 SQLAlchemy (1.1.11)
 urllib3 (1.21.1)
 virtualenv (15.1.0)
 webencodings (0.5.1)
 Werkzeug (0.8.3)
 wheel (0.29.0)
 ```

#### Installing Apache web server & setting up the project
 1. Use the following commands to install apache web server:
 ```
 sudo apt-get install apache2
 sudo apt-get install libapache2-mod-wsgi
 sudo service apache2 restart
 ```
 2. You can test if your apache server is working correctly by going to `http://<public-ip-address>` which will show a default static html page for ubuntu that came with apache.
 3. Now apache directories are setup and you can import your project to this directory.
 4. Goto `/var/www/` using `cd /var/www/` and run the following commands:
 ```
 sudo mkdir FlaskApp
 cd FlaskApp
 sudo mkdir Catalog
 git clone <link-to-your-catalog-git-repo>
 ```
 The directory structure will look like follows:
 ```
 ---/
   ---var
     ---www
       ---FlaskApp
         ---Catalog
           ---static
           ---templates
           ---all_other_files
         ---html
 ```
 5. Rename your `app.py` file to `__init__.py` and change the database connection string to `postgresql://grader:grader@localhost/catalog` in both `database_setup.py` and `__init__.py` files.
 6. Run `database_setup.py` file to create required tables.

#### Now setting up the apache to serve Flask Application
 1. Create the FlaskApp.conf file using the command `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
 2. Edit this file so it looks as follows:
 ```
 <VirtualHost *:80>
    	ServerName <public-dns-address>
    	ServerAdmin <Your-email-address>
    	WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
    	<Directory /var/www/FlaskApp/FlaskApp/>
    		Order allow,deny
    		Allow from all
    	</Directory>
    	Alias /static /var/www/FlaskApp/FlaskApp/static
    	<Directory /var/www/FlaskApp/FlaskApp/static/>
    		Order allow,deny
    		Allow from all
    	</Directory>
    	ErrorLog ${APACHE_LOG_DIR}/error.log
    	LogLevel warn
    	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
 ```
 3. Now enable the virtual host using the command `sudo a2ensite FlaskApp.conf`
 4. Then create a .wsgi file in the `FlaskApp` folder present at `/var/www/FlaskApp` and add the following lines to that file:
 ```
 #!/usr/bin/python
 import sys
 import logging
 logging.basicConfig(stream=sys.stderr)
 sys.path.insert(0,"/var/www/FlaskApp/")

 from Catalog import app as application
 application.secret_key = 'insert-your-app-secret-key-here-and-remove-it-from-the-__init__-file'
 ```
 5. Now restart the server using `sudo service apache2 restart`
 6. Go to `http://ec2-34-227-58-96.compute-1.amazonaws.com/` to test the application.
 7. Use browser's JavaScript Console and apache logs accessed at `sudo cat /var/log/apache2/error.log` to debug your application.

### References
 - Setting up users: [Digital Ocean Link](https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart)
 - Setting up FlaskApp through Apache: [Digital Ocean Link](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
 - Root user settings: [a2hosting Link](https://www.a2hosting.com/kb/getting-started-guide/accessing-your-account/disabling-ssh-logins-for-root)
