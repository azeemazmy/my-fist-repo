# Connecting WordPress to Apache — Step by Step

This guide covers installing WordPress on the Ubuntu Server and connecting it to Apache, including the database and PHP setup it needs.

## 1. Update the System

```
sudo apt update
sudo apt upgrade -y
```

## 2. Install PHP and Required Extensions

WordPress needs PHP and several PHP modules to run properly:
```
sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip -y
```

## 3. Install MySQL (Database Server)

WordPress stores its data in a MySQL database:
```
sudo apt install mysql-server -y
```

Secure the installation:
```
sudo mysql_secure_installation
```

Follow the prompts to set a root password and remove insecure defaults.

## 4. Create a WordPress Database and User

Log into MySQL:
```
sudo mysql -u root -p
```

Inside the MySQL prompt, create the database and user (replace `yourpassword` with a strong password):
```sql
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 5. Download WordPress

Move to a temporary directory and download the latest version:
```
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
```

## 6. Move WordPress Files to the Web Directory

```
sudo mv wordpress /var/www/html/wordpress
```

## 7. Set File Permissions

Give Apache ownership of the WordPress files:
```
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress
```

## 8. Create the wp-config.php File

Move into the WordPress folder:
```
cd /var/www/html/wordpress
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php
```

Update the database settings to match what you created earlier:
```php
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wpuser' );
define( 'DB_PASSWORD', 'yourpassword' );
define( 'DB_HOST', 'localhost' );
```

Save and exit (`Ctrl+O`, `Enter`, then `Ctrl+X` in nano).

## 9. Create an Apache Virtual Host for WordPress

Create a new config file:
```
sudo nano /etc/apache2/sites-available/wordpress.conf
```

Add the following:
```apache
<VirtualHost *:80>
    ServerAdmin admin@example.com
    DocumentRoot /var/www/html/wordpress
    ServerName <server-ip-address>

    <Directory /var/www/html/wordpress>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/wordpress_error.log
    CustomLog ${APACHE_LOG_DIR}/wordpress_access.log combined
</VirtualHost>
```

`AllowOverride All` is important — it lets WordPress use `.htaccess` for things like permalinks.

## 10. Enable the Site and Required Apache Modules

Enable the new virtual host:
```
sudo a2ensite wordpress.conf
```

Enable the rewrite module (needed for WordPress permalinks):
```
sudo a2enmod rewrite
```

(Optional) Disable the old default site if WordPress should be the main site:
```
sudo a2dissite 000-default.conf
```

## 11. Restart Apache

```
sudo systemctl restart apache2
```

## 12. Allow Apache Through the Firewall (If Not Already Done)

```
sudo ufw allow 'Apache'
```

## 13. Finish Setup in the Browser

1. On another device, open a browser and go to:
   ```
   http://<server-ip-address>
   ```
2. The WordPress installation wizard should load.
3. Choose your language, then enter:
   - Site title
   - Admin username and password
   - Admin email
4. Click **"Install WordPress."**
5. Log in using the admin account you just created.

## 14. Test the Site

1. Confirm the homepage loads correctly at `http://<server-ip-address>`.
2. Log into the WordPress dashboard at:
   ```
   http://<server-ip-address>/wp-admin
   ```
3. Try creating a test post or page to confirm the database connection is working.

## Outcome

At this point, WordPress is installed, connected to its MySQL database, and served through Apache. The site is reachable by any device on the local network, and the admin dashboard is ready for further setup (themes, plugins, content).
