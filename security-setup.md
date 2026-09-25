# Security — Step by Step

This guide covers basic security hardening for the server: firewall setup and access control for file and web services.

## 1. Update the System First

Always start with an updated system so security patches are in place:
```
sudo apt update
sudo apt upgrade -y
```

## 2. Install and Enable the Firewall

Ubuntu uses `ufw` (Uncomplicated Firewall) by default.

Install it if not already present:
```
sudo apt install ufw -y
```

Check its current status:
```
sudo ufw status
```

## 3. Set Default Firewall Rules

Set sensible defaults: deny incoming traffic by default, allow outgoing:
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

## 4. Allow Only Necessary Ports

Allow SSH so you can still manage the server remotely:
```
sudo ufw allow ssh
```

Allow Samba (file sharing):
```
sudo ufw allow samba
```

Allow Apache (web server, HTTP):
```
sudo ufw allow 'Apache'
```

If using HTTPS later:
```
sudo ufw allow 'Apache Full'
```

## 5. Enable the Firewall

```
sudo ufw enable
```

Confirm the active rules:
```
sudo ufw status verbose
```

## 6. Restrict SSH Access (Recommended)

1. Consider limiting SSH access to only trusted devices on the local network instead of the whole internet.
2. Example: allow SSH only from a specific IP range:
   ```
   sudo ufw allow from 192.168.1.0/24 to any port 22
   ```
3. Remove the broader SSH rule if it's no longer needed:
   ```
   sudo ufw delete allow ssh
   ```

## 7. Secure SSH Login

Edit the SSH config file:
```
sudo nano /etc/ssh/sshd_config
```

Recommended changes:
- Disable root login:
  ```
  PermitRootLogin no
  ```
- (Optional, more advanced) Disable password login and use SSH keys instead:
  ```
  PasswordAuthentication no
  ```

Restart SSH to apply changes:
```
sudo systemctl restart ssh
```

## 8. Access Control for Samba (File Sharing)

1. Only give Samba access to specific users, not everyone:
   ```
   sudo smbpasswd -a sambauser
   ```
2. In `/etc/samba/smb.conf`, make sure the shared folder restricts access:
   ```ini
   [Shared]
      valid users = sambauser
      read only = no
   ```
3. Avoid using `guest ok = yes` unless the share is meant to be fully public.

## 9. Access Control for Apache (Web Server)

1. Set correct file ownership and permissions so only the web server user can modify site files:
   ```
   sudo chown -R www-data:www-data /var/www/html
   sudo chmod -R 755 /var/www/html
   ```
2. Disable directory listing so visitors can't browse folder contents directly:
   ```
   sudo nano /etc/apache2/apache2.conf
   ```
   Find the relevant `<Directory>` block and change:
   ```apache
   Options -Indexes
   ```
3. Restart Apache to apply changes:
   ```
   sudo systemctl restart apache2
   ```

## 10. Disable Unused Services

Check what's running:
```
systemctl list-units --type=service --state=running
```

Disable anything not needed for this server's purpose (file/web hosting), to reduce the attack surface:
```
sudo systemctl disable <service-name>
sudo systemctl stop <service-name>
```

## 11. Keep the System Updated

Set up automatic security updates so patches are applied without manual work:
```
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

## 12. Test the Setup

1. From another device, confirm only the allowed services (SSH, Samba, Apache) respond.
2. Try connecting to a port that should be blocked and confirm it's refused.
3. Confirm SSH, Samba, and Apache still work correctly after all firewall changes.

## Outcome

At this point, the server has a working firewall that only allows necessary traffic, restricted SSH access, controlled access to shared files, a locked-down web directory, and automatic security updates enabled.
