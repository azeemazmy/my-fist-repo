# Installing and Configuring Apache Web Server — Step by Step

This guide covers setting up Apache on Ubuntu Server to host a website on the local network.

## 1. Update the System

Before installing anything, make sure the system is up to date:
```
sudo apt update
sudo apt upgrade -y
```

## 2. Install Apache

Install the Apache package:
```
sudo apt install apache2 -y
```

Check that the service is running:
```
sudo systemctl status apache2
```

## 3. Allow Apache Through the Firewall

If `ufw` is active, allow web traffic:
```
sudo ufw allow 'Apache'
```

To also allow HTTPS later, you can instead use:
```
sudo ufw allow 'Apache Full'
```

## 4. Test the Default Page

1. Find the server's IP address:
   ```
   ip a
   ```
2. On another device on the network, open a browser and go to:
   ```
   http://<server-ip-address>
   ```
3. You should see the default Apache "It works!" page. This confirms Apache is installed and running correctly.

## 5. Locate the Web Root Directory

Apache serves files from:
```
/var/www/html
```

This is where your website's files go.

## 6. Back Up the Default Page

Before replacing anything, back up the default index file:
```
sudo mv /var/www/html/index.html /var/www/html/index.html.bak
```

## 7. Add Your Website Files

Create a simple test page:
```
sudo nano /var/www/html/index.html
```

Add basic HTML content, for example:
```html
<!DOCTYPE html>
<html>
<head>
    <title>My Server</title>
</head>
<body>
    <h1>Hello from my Ubuntu Server!</h1>
    <p>This page is being served by Apache.</p>
</body>
</html>
```

Save and exit (`Ctrl+O`, `Enter`, then `Ctrl+X` in nano).

## 8. Set File Permissions

Make sure Apache can read the files:
```
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

## 9. Restart Apache

Apply the changes:
```
sudo systemctl restart apache2
sudo systemctl enable apache2
```

`enable` makes sure Apache starts automatically on boot.

## 10. Test the Updated Site

On another device on the network, refresh the browser at:
```
http://<server-ip-address>
```

You should now see your custom page instead of the default Apache page.

## 11. (Optional) Set Up a Virtual Host

If you plan to host more than one site later, set up a virtual host config:
```
sudo nano /etc/apache2/sites-available/mysite.conf
```

Add:
```apache
<VirtualHost *:80>
    ServerAdmin admin@example.com
    DocumentRoot /var/www/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable the site and reload Apache:
```
sudo a2ensite mysite.conf
sudo systemctl reload apache2
```

## Outcome

At this point, Apache is installed, configured, and tested. The server hosts a working website that other devices on the local network can reach by typing its IP address into a browser.
