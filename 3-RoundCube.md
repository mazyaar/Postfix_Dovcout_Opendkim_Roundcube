# Part 3

# 5. Rouncube 

## 5.1 update and upgrade:
```bash
sudo apt-get update
```
```bash
sudo apt-get upgrade
```

## 5.2 Install apache2 and php7:
```bash
sudo apt-get install apache2 mariadb-server php7.2 php7.2-gd php-mysql php7.2-curl php7.2-zip php7.2-ldap php7.2-mbstring php-imagick php7.2-intl php7.2-xml unzip wget curl -y
```
```bash
sudo nano /etc/php/7.2/apache2/php.ini
```
Set TimeZone:
```
date.timezone = Asia/Tehran
```

## 5.3 install Mysql:
```bash
sudo apt install mysql-client mysql-server
```
```bash
sudo mysql -p
```
```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'MasterCyber123!@#';
exit
```
```bash
sudo mysql_secure_installation
```
```bash
sudo mysql -u root -p
```
```bash
CREATE DATABASE rc_db;
CREATE USER 'cyberrc'@'localhost' IDENTIFIED BY 'P@ssw0rdRc';
GRANT ALL PRIVILEGES ON rc_db.* TO 'cyberrc'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 5.4 restart services 
```bash
sudo systemctl start apache2
```
```bash
sudo systemctl enable apache2
```
```bash
sudo systemctl start mysql
```
```bash
sudo systemctl enable mysql
```

## 5.5 Roundcube Install:
```bash
cd /tmp
```

>_wget https://github.com/roundcube/roundcubemail/releases/download/1.3.8/roundcubemail-1.6.3-complete.tar.gz_
>
```bash
tar -xvzf roundcubemail-1.6.2-complete.tar.gz
mkdir -p /var/www/html/roundcub
chown www-data -R www-data:www-data /var/www/html/roundcub
sudo chmod -R 775 /var/www/html/roundcube
rsysnc -av /tmp/* /var/www/html/roundcub
```
**Now initiate the database by executing the following command:**
```bash
mysql -u cyberrc -p rc_db < /var/www/roundcube/SQL/mysql.initial.sql 
```
## 5.6 apache Configuration:
```bash
sudo nano /etc/apache2/sites-available/roundcube.conf
```
```bash
<VirtualHost *:80>
 ServerName example.com
 ServerAdmin admin@example.com
 DocumentRoot /var/www/html/roundcube

ErrorLog ${APACHE_LOG_DIR}/roundcube_error.log
 CustomLog ${APACHE_LOG_DIR}/roundcube_access.log combined

<Directory /var/www/html/roundcube>
 Options -Indexes
 AllowOverride All
 Order allow,deny
 allow from all
 </Directory>
 </VirtualHost>
```
## 5.7 Enable sites
 ```bash
sudo a2ensite roundcube
sudo a2enmod rewrite
sudo systemctl restart apache2
```

## 5.8 Options to crate free cert and redirect to https:
```bash
certbot --apache
```
## 5.9 Configure Roundcube
>_Now, Open your browser and navigate to  **http://host.com/roundcubemail/installer.**

![Roundcube installation Wizzard ](https://tecadmin.net/wp-content/uploads/2021/07/roundcube-step-1.png)

![Roundcube installation Step 2](https://tecadmin.net/wp-content/uploads/2021/07/roundcube-step-2.png)


![](https://www.nginxweb.ir/blog/images/2019/05/15568927536446.png)

![](https://www.nginxweb.ir/blog/images/2019/05/15568927545714.png)

**Create Configuration file:**
![](https://www.nginxweb.ir/blog/images/2019/05/15568928263420.png)

**Continue**
![](https://www.nginxweb.ir/blog/images/2019/05/15568928619111.png)

## 5.10 Removing Sensitive Information from Email Headers
> _By default, Roundcube will add a User-Agent email header, indicating that you are using Roundcube webmail and the version number. You can tell Postfix to ignore it so recipient can not see it. Run the following command to create a header check file_
```bash
sudo nano /etc/postfix/smtp_header_checks
```
**Put the following lines into the file.**
```
/^User-Agent.*Roundcube Webmail/            IGNORE
```
```bash
sudo nano /etc/postfix/main.cf
```
**Add the following line at the end of the file.**
```bash
smtp_header_checks = regexp:/etc/postfix/smtp_header_checks
```
_Save and close the file. Then run the following command to rebuild hash table._
```bash
sudo postmap /etc/postfix/smtp_header_checks
```
***Reload Postfix for the change to take effect.***
```bash
sudo systemctl reload postfix
```
## 5.11 Increase Upload File Size Limit
> _If you use PHP-FPM to run PHP scripts, then files such as images, PDF files uploaded to Roundcube can not be larger than 2MB. To increase the upload size limit, edit the PHP configuration file._
```bash
sudo nano /etc/php/7.2/apache2/php.ini
```
```
upload_max_filesize = 50M
post_max_size = 50M
```
***Alternatively, you can run the following two commands to change the value without manually opening the file.***
```bash
sudo sed -i 's/upload_max_filesize = 2M/upload_max_filesize = 50M/g' /etc/php/7.2/fpm/php.ini
sudo sed -i 's/post_max_size = 8M/post_max_size = 50M/g' /etc/php/7.2/fpm/php.ini
```
```bash
systemctl restart apache2
```
* There are 3 plugins in Roundcube for attachments/file upload:

* database_attachments
* filesystem_attachments
* redundant_attachments
_Roundcube can use only one plugin for attachments/file uploads. I found that the database_attachment plugin can be error_prone and cause you trouble. To disable it, edit the Roundcube config file._
```bash
sudo nano /var/www/roundcube/config/config.inc.php
```
_Scroll down to the end of this file. You will see a list of active plugins. Remove 'database_attachments' from the list. Note that you need to activate at least one other attachment plugin, for example, filesystem_attachments._
```bash
// ----------------------------------
// PLUGINS
// ----------------------------------
// List of active plugins (in plugins/ directory)
$config['plugins'] = ['acl', 'additional_message_headers', 'archive', 'attachment_reminder', 'autologon', 'debug_logger', 'emoticons', 'enigma', 'filesystem_attachments', 'help', 'hide_blockquote', 'http_authentication', 'identicon', 'identity_select', 'jqueryui', 'krb_authentication', 'managesieve', 'markasjunk', 'new_user_dialog', 'new_user_identity', 'newmail_notifier', 'password', 'reconnect', 'redundant_attachments', 'show_additional_headers', 'squirrelmail_usercopy', 'subscriptions_option', 'userinfo', 'vcard_attachments', 'virtuser_file', 'virtuser_query', 'zipdownload'];
```
***Save and close the file.***
