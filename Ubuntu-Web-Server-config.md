Ubuntu Web Server - configuration steps | Ubuntu 22.04 LTS

Part I - Install Ubuntu

a) create an Admin account with Administrator privileges
    --> open a terminal
    --> set a root password [sudo passwd root]
b) switch to root
    --> update [apt-get update]
    --> upgrade [apt-get upgrade -y]
c) reboot

Part II - Install and configure Apache2
    based on https://ubuntu.com/tutorials/install-and-configure-apache#1-overview | copy the link to get more info

a) log into Admin
b) open terminal
    --> su - [root]
c) apt install apache2
d) creating a simply website:
    --> mkdir /var/www/gci/
    --> nano index.html [insert here whatever you with html syntax or paste a code from tutorial site above]
        ---> press Ctrl + X , then 'y' to save the changes
e) setting up a VirtualHost:
    --> cd /etc/apache2/sites-available/
        ---> cp 000-default.conf gci.conf [default configuration copied to your subdomain file]
    --> nano gci.conf
    --> set an email to ServerAdmin [fictional]
    --> replace 'html' to 'gci' in DocumentRoot path
    --> set an example of your website address [with gci. in the beginning] in ServerName like gci.example.com
        ---> press Ctrl + X , then 'y' to save the changes
f) activating VirtualHost:
    --> a2ensite gci.conf
    --> service apache2 reload
    --> systemctl status apache2
g) type your host name in a browser to see the result

helpful directory info:
    --> /var/www/gci - html website example location
    --> /etc/apache2/sites-available - default and gci configuration files

Part III - Install and configure UFW (Uncomplicated Firewall) | Optional step to use SSH connection

a) log into Admin account
b) open terminal
c) su -
d) apt install ufw
    --> ufw status verbose
e) setting up:
    --> ufw default deny incoming [to deny incoming connections]
    --> ufw default allow outgoing [to allow outgoing connections]
    --> ufw allow ssh [allows to connect via ssh protocol]
        ---> ufw allow 22 [specified port]
    --> ufw enable [if inactive]
        ---> press 'y'
f) open http on 80 port
    --> ufw allow http
    --> ufw allow 80/tcp
g) open https on 443 port
    --> ufw allow https
    --> ufw allow 443/tcp

Part IV - OpenSSL configuration | Self-Signed SSL Certificate
    based on https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-20-04
    | copy the link to get more info

a) log into Admin account
b) open terminal
c) su -
d) apt-get update & apt-get upgrade -y [if it needs]
e) ufw allow "Apache Full"

1. Enabling - mod_ssl
    --> a2enmod ssl
    --> systemctl restart apache2
it is now ready to use

2. Creating the SSL Certificate
    --> openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
    [fill these names below, the rest leave with dot '.' sign]
        ---> Country Name: (your country marks)
        ---> Locality: (your city)
        ---> Organization Name: (can be imagine or the same which you used as website name)
        ---> Common Name: (the website name = gci.<example>.com) [this is one of the most important info here]
        ---> Email Address: (email that you specified in gci.conf)

3. Configuring Apache to Use SSL
    --> cd /etc/apache2/sites-available/
    --> nano gci.conf
        [add this elements to the bottom of the file]:

        SSLEngine on
        SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
        SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
    
    --> save it
    --> replace the port inside VirtualHost to 443

        <VirtualHost *:80> === <VirtualHost *:443>

    --> apache2ctl configtest [to be sure that apache syntax is OK]
        ---> systemctl reload apache2
        ---> systemctl status apache2
    
    [add https:// at the beginning to your website and reload]

4. Redirecting http to https
    --> cd /etc/apache2/sites-available/
    --> nano gci.conf 
        [open another VirtualHost at the bottom of the file and add the 80 port]:

        <VirtualHost *:80>
	        ServerName (your website address)
	        Redirect / https://gci.<website-name>.com/
        </VirtualHost>
    
    --> save it
    --> apachectl configtest
    --> systemctl reload apache2

[visit your site with plain 'http://' in front of the address, now you should be redirected to 'https://' automatically]

helpful directory info:
    --> /etc/ssl/private/ - keys stored
    --> /etc/ssl/certs/ - certificate stored ---> ll /etc/ssl/certs/apache-selfsigned.crt (check if exists)

Part V - Installing PHP on Ubuntu | PHP 8.2 
    based on https://www.liquidweb.com/kb/install-php-on-ubuntu/ | copy the link to get more info

a) log into Admin account
b) open terminal
c) su -
d) apt-get update & apt-get upgrade -y [if it needs]

1. Adding the official PHP repository
    --> add-apt-repository ppa:ondrej/php
    --> press enter to accept

2. Performing the PHP installation
    --> apt-get install php8.2 -y

3. Config the OS to use the new PHP vesion
    --> update-alternatives --set php /usr/bin/php8.2

4. Veryfing version
    --> php -v

5. Config Apache to use the PHP 8.2 version
    --> ll /etc/apache2/mods-available/php* [see which mods are already installed]
    --> a2enmod php8.2 [to enable PHP 8.2]
    --> apachectl -t [show parsed run settings]
    --> service apache2 restart

6. Verifying Apache's PHP version
    --> ll /etc/apache2/mods-enabled/php*
    --> nano /var/www/gci/info.php
        ---> [type inside the file]
            <?php
                phpinfo();
            ?>
        ---> save
        ---> https://<your-website>/info.php [plain this to the browser to verify php version]
    --> php -v

7. Adding modules to PHP 8.2
    --> apt-get install php8.2 (press double Tab to see available mods)
    --> apt-get install php8.2-mysql php8.2-soap

[in addition you may compare them to see which modules are missing - the compare process is presented on the website]

Part VI - Install and configure MariaDB
    based on
        https://mariadb.com/kb/en/mariadb-secure-installation/
        https://www.digitalocean.com/community/tutorials/how-to-install-mariadb-on-ubuntu-22-04 [do not proceed with step 3 on the website]
        | copy the link to get more info

a) log into Admin account
b) open terminal
c) su -
d) apt-get update & apt-get upgrade -y [if it needs]

1. Install MariaDB
    --> apt install mariadb-server mariadb-client

2. Configure MariaDB
    --> mysql_secure_installation
    [proceed with steps during configuration]
        ---> Switch to unix_socket authentication - n
        ---> Change/Set the root password? - n
        ---> Remove anonymous users? - y
        ---> Disallow root login remotely? - y
        ---> Remove test database and access to it? - y
        ---> Reload privilege tables now? - y

3. Testing MariaDB
    --> systemctl status mariadb
        ---> systemctl start mariadb {if it's not running}
    --> mysqladmin version

Part VII - Install and configure WordPress
    based on https://www.digitalocean.com/community/tutorials/install-wordpress-on-ubuntu | copy the link to get more info

a) log into Admin account
b) open terminal
c) su -
d) apt-get update & apt-get upgrade -y [if it needs]

1. Create WordPress Database
    --> mysql -u root -p [enter your root password]
    --> CREATE DATABASE wordpress_db;
    --> CREATE USER '<chosen_name>'@'<ip_host>' IDENTIFIED BY 'password';
    --> GRANT ALL ON wordpress_db.* TO '<chosen_name>'@'<ip_host>' IDENTIFIED BY 'password';
    --> FLUSH PRIVILEGES;
    --> exit

2. Install WordPress CMS
    --> cd /tmp
    --> wget https://wordpress.org/latest.tar.gz
    --> tar -xvf latest.tar.gz
    --> cp -R wordpress /var/www/gci/
    --> chown -R www-data:www-data /var/www/gci/wordpress/
    --> chmod -R 755 /var/www/gci/wordpress/
    --> cd /var/www/html/wordpress/wp-content/
        ---> mkdir uploads
    --> chown -R www-data:www-data /var/www/gci/wordpress/wp-content/uploads/
    --> https://<your-website>/wordpress
    [Now you can set up WordPress by yourself]
