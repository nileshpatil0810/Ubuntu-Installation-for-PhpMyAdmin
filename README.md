# Ubuntu-Installation-for-PhpMyAdmin
Setup Ubuntu Installation for PhpMyAdmin

Prerequisite:
- Setup instance with Ubuntu on AWS with running status
- Generate pem file and public IP (Prefer to get elastic IP first)
- Connect your instance using SSH and follow steps

Step 1: Install Apache

sudo apt-get update

Update the dist to run all the packages working properly 

sudo apt-get dist-upgrade
sudo apt-get install apache2
sudo a2enmod rewrite

Note: 
If you getting error like 'sudo: a2enmod: command not found' then fire following command
sudo apt install apache2
Step 2: Install PHP

sudo apt-get install php libapache2-mod-php

Note:
If you would like to install php7.0 then try following command
sudo apt-get install -y php7.0 libapache2-mod-php7.0 php7.0-cli php7.0-common php7.0-mbstring php7.0-gd php7.0-intl php7.0-xml php7.0-mysql php7.0-mcrypt php7.0-zip

Add user ubuntu to group www-data and change permission

sudo adduser ubuntu www-data

sudo chown -R www-data:www-data /var/www
sudo chmod -R g+rw /var/www

Restart apache

sudo service apache2 restart

Step 3: Install MySQL
sudo apt-get install mysql-server

Step 4: Install PhpMyadmin

sudo apt update
sudo apt install phpmyadmin php-mbstring php-gettext

For the server selection, choose apache2
Select Yes when asked whether to use dbconfig-common to set up the database
You will then be asked to choose and confirm a MySQL application password for phpMyAdmin
Enable mbstring extension
sudo phpenmod mbstring
Restart Apache
sudo systemctl restart apache2

Error
1. If Phpmyadmin not connect to RDS then update HOST in config.inc.php at /etc/phpmyadmin
$cfg['Servers'][$i]['host']='xxx.us-west-2.rds.amazonaws.com'

Make sure you are updating host at proper place.

2. mysqli_real_connect(): (HY000/1698): Access denied for user 'root'@'localhost'
Ref: https://askubuntu.com/a/763359

You can't use the root user in 5.7 anymore without becoming a sudoer as well as in GUI(any non-command line application). That means you can't just run mysql -u root anymore and have to do sudo mysql -u root instead. 

To make it work you'll have to create a new user with the required privileges and use that instead.
2.1 Connect to MySQL
sudo mysql --user=root mysql
2.2 Create a user for phpMyAdmin
CREATE USER 'phpmyadmin'@'localhost' IDENTIFIED BY 'some_pass';
GRANT ALL PRIVILEGES ON *.* TO 'phpmyadmin'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
Note: 
phpmyadmin is user's name and some_pass is for password
2.3 and 2.4 is not required if you able to logged successfully with new user

2.3 Optional and unsafe: allow remote connections
CREATE USER 'phpmyadmin'@'%' IDENTIFIED BY 'some_pass';
GRANT ALL PRIVILEGES ON *.* TO 'phpmyadmin'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;

Remember: allow a remote user to have all privileges is a security concern and this is not required in most of cases.

2.4 Update phpMyAdmin
Using sudo, edit /etc/dbconfig-common/phpmyadmin.conf file updating user/password values in the following sections
# dbc_dbuser: database user
#       the name of the user who we will use to connect to the database.
dbc_dbuser='phpmyadmin'

# dbc_dbpass: database user password
#       the password to use with the above username when connecting
#       to a database, if one is required
dbc_dbpass='some_pass'
