# Item Catalog Project deployed on Amazon Web Services
This project is a submission for the Udacity Full Stack Nanodegree Linux server configuration project.  The task is to configure a Linux server running on the Amazon Web Services platform with the appropriate software, user accounts and firewall and security settings, and to deploy a previously created website to run on the server.

# Requirements
1. The original Item Catalog Project, which can be found at https://github.com/melissapott/FSND_Item_Catalog
2. An Amazon Web Services Lightsail account with an instance of Ubuntu

# Configuration Steps

## Security Settings
1. Edit the AWS lightsail firewall to allow ports 2200 and 123
2. Log in to the server and make the following settings for the Universal Firewall
  1. set the UFW default to allow outgoing and deny incoming
  2. set the UFW to allow SSH, 2200/tcp, 123, and www
  3. edit the /etc/ssh/sshd_config file, changing the "port 22" line to "port 2200"
  4. re-start the sshd service
3. Edit the AWS Lightsail firewall to disallow port 22
4. Disallow Root SSH by editing the /etc/ssh/sshd_config file and adding "PermitRootLogin no"
5. Disable password authentication, thus forcing key pair authentication by editing /etc/ssh/sshd_config file and setting "password authentication" to no
6. Restart the ssh service
  
## Timezone Settings
Ensure that the timezone is configured for UTC by running dpkyg-reconfigure tzdata

## Add user for project review
1. create a user called "grader" to be used by person doing the project evaluation
2. usermod -aG sudo grader to allow the user to have sudo access
3. on local machine, use ssh-keygen to create a key pair for the grader user
4. on the server, as the grader user, mkdir .ssh
5. create the file .ssh/authorized_keys, and paste the contents of the public key created in step 3
6. chmod the .ssh directory to 700 and the .ssh/authorized_keys file to 644

## Install software / application dependencies
1. use the sudo apt-get install command to install the following:
  - apache2
  - apache2-utils
  - postgresql
  - git
  - python-pip
  - libapache2-mod-wsgi
2. use the sudo pip install command to install the following:
  - Flask
  - Oauth2client
  - sqlalchemy
  - psycopg2
  - requests
3. Copy the web application files to the server by using the git clone command

## Database setup
1. Create the database and database user accounts by running:
  - `sudo -i -u postgres` 
  - `createuser catalog`
  - `createdb catalog catalog`
2. Note: the word "user" is a reserved word in Postgresql, but not in sqLite, where the app was first written, I have opted to change my original use of the word 'user' in table and column names to 'person' in order to avoid problems and confusion.  Therefore, the database_setup.py and catalog.py files should be edited to replace the use of 'user' to 'person' when referencing database tables and fields.
3. Update the database_setup.py and catalog.py files to reflect the change from sqLite to Postgresql
  - add a line `import psycopg2` to each file
  - change create engine lines to: `engine = create_engine('postgresql://catalog:catalogpwd@localhost/catalog`
4. Initialize the database by running: `python database_setup.py`

## Edit the web application to reflect the new host environment
  - update the location of the Facebook and Google client secrets files with the correct path
  - enable uploads to the /image directory for a web user by running `sudo chown www-data:www-data /var/www/Catalog/images`
  - ensure that catalog.wsgi file is present in the /var/www/Catalog directory, which should be included in this repository
  - create the host configuration file as /etc/apache2/sites-available/catalog.conf which should contain: 
  
  ```<VirtualHost *:80>
        ServerName 18.217.175.142
        ServerAdmin melissa.pott@gmail.com
        WSGIScriptAlias / /var/www/Catalog/catalog.wsgi
        <Directory /var/www/Catalog/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/Catalog/static
        <Directory /var/www/Catalog/static/>
                Order allow,deny
                Allow from all
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>```



