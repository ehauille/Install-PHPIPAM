# Install-PHPIPAM on Ubuntu 20.04/18.04 / Debian 10


# Once the database installation and setup is complete, create a database for phpipam user:
    sudo mysql -u root -p

    CREATE DATABASE phpipam;
    GRANT ALL ON phpipam.* TO phpipam@localhost IDENTIFIED BY 'StrongDBPassword';
    FLUSH PRIVILEGES;
    QUIT;

# Step 2: Install PHP and required modules. Next phase is the installation of php and required modules. Run the following commands:
    sudo apt update 
    sudo apt -y install php php-{mysql,curl,gd,intl,pear,imap,memcache,pspell,tidy,xmlrpc,mbstring,gmp,json,xml,fpm}

# Step 3: Install phpIPAM on Debian 10 / Ubuntu 20.04/18.04 LTS. We’ll download phpIPAM from Github. Install git first:
    sudo apt -y install git

# Clone phpIPAM code from github
    sudo git clone --recursive https://github.com/phpipam/phpipam.git /var/www/html/phpipam

# Change to clone directory.
    cd /var/www/html/phpipam

- You can also download phpipam from official Sourceforge repository and extract it to your web server directory.

# Step 4: Configure phpIPAM on Ubuntu 20.04/18.04 / Debian 10. Change your working directory to /var/www/html/phpipam and copy config.dist.php to config.php, then edit it.
    cd /var/www/html/phpipam
    sudo cp config.dist.php config.php

# Edit the file to configure database credentials as added on Step 1:
    sudo vim config.php
    /**
    * database connection details
    ******************************/
    $db['host'] = 'localhost';
    $db['user'] = 'phpipam';
    $db['pass'] = 'StrongDBPassword';
    $db['name'] = 'phpipam';
    $db['port'] = 3306;

- Option 1: Using Nginx Web server

# Install nginx using the command:
    sudo systemctl stop apache2 && sudo systemctl disable apache2
    sudo apt -y install nginx
    
# Configure nginx:
    sudo vim /etc/nginx/conf.d/phpipam.conf

# Ubuntu 20.04:

    server {
        # root directory
        root   /var/www/html;

        # phpipam
        location /phpipam/ {
            try_files $uri $uri/ /phpipam/index.php;
            index index.php;
        }
        # phpipam - api
        location /phpipam/api/ {
            try_files $uri $uri/ /phpipam/api/index.php;
        }

        # php-fpm
        location ~ \.php$ {
            fastcgi_pass   unix:/run/php/php7.4-fpm.sock;
            fastcgi_index  index.php;
            try_files      $uri $uri/ index.php = 404;
            include        fastcgi_params;
        }
     }
     
# Ubuntu 18.04:

    server {
      # root directory
      root   /var/www/html;

      # phpipam
      location /phpipam/ {
          try_files $uri $uri/ /phpipam/index.php;
          index index.php;
      }
      # phpipam - api
      location /phpipam/api/ {
          try_files $uri $uri/ /phpipam/api/index.php;
      }

      # php-fpm
      location ~ \.php$ {
          fastcgi_pass   unix:/run/php/php7.2-fpm.sock;
          fastcgi_index  index.php;
          try_files      $uri $uri/ index.php = 404;
          include        fastcgi_params;
      }
    }
    
# Debian 10:

    server {
        # root directory
        root   /var/www/html;

        # phpipam
        location /phpipam/ {
            try_files $uri $uri/ /phpipam/index.php;
            index index.php;
        }
        # phpipam - api
        location /phpipam/api/ {
            try_files $uri $uri/ /phpipam/api/index.php;
        }

        # php-fpm
        location ~ \.php$ {
            fastcgi_pass   unix:/run/php/php7.3-fpm.sock;
            fastcgi_index  index.php;
            try_files      $uri $uri/ index.php = 404;
            include        fastcgi_params;
        }
     }

# Change ownership of the /var/www/ directory to www-data user and group.
    sudo chown -R www-data:www-data /var/www/html
    sudo systemctl restart nginx
    
# Option 2: Using Apache Web Server. If you would like to use Apache web server, first install it using:
    sudo systemctl stop nginx && sudo systemctl disable nginx
    sudo apt -y install apache2
    sudo a2enmod rewrite
    sudo systemctl restart apache2
    
# Install apache php module:
    sudo apt -y install libapache2-mod-php php-curl php-xmlrpc php-intl php-gd

# Add Apache phpipam configuration:
    sudo vim /etc/apache2/sites-available/phpipam.conf

# Here are the contents:
    <VirtualHost *:80>
        ServerAdmin admin@example.com
        DocumentRoot "/var/www/html/phpipam"
        ServerName phpipam.computingforgeeks.com
        ServerAlias www.phpipam.computingforgeeks.com
        <Directory "/var/www/html/phpipam">
            Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted
        </Directory>
        ErrorLog "/var/log/apache2/phpipam-error_log"
        CustomLog "/var/log/apache2/phpipam-access_log" combined
    </VirtualHost>
# Enable site:
    sudo  a2ensite phpipam

# Restart Apache server for changes to be made.
    sudo systemctl restart apache2
