# Installing and Configuring Samba — Step by Step

This guide covers setting up Samba on Ubuntu Server so files can be shared with both Windows and Linux devices on the network.

## 1. Update the System

Before installing anything, make sure the system is up to date:
```
sudo apt update
sudo apt upgrade -y
```

## 2. Install Samba

Install the Samba package:
```
sudo apt install samba -y
```

Check that it installed correctly and the service is running:
```
sudo systemctl status smbd
```

## 3. Create a Shared Directory

Create a folder that will be shared over the network:
```
sudo mkdir -p /srv/shared
```

Set ownership and permissions so it can be accessed properly:
```
sudo chown -R nobody:nogroup /srv/shared
sudo chmod -R 0775 /srv/shared
```

## 4. Create a Samba User

Samba uses its own user accounts, separate from regular Linux logins. First create a Linux user (if one doesn't already exist), then add them to Samba:

```
sudo adduser sambauser
sudo smbpasswd -a sambauser
```

You'll be asked to set a password for Samba access. This is the password used when connecting from other devices.

## 5. Back Up the Original Config File

Before editing, make a backup of the default Samba config:
```
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

## 6. Edit the Samba Configuration File

Open the config file:
```
sudo nano /etc/samba/smb.conf
```

Scroll to the bottom and add a new section for the shared folder:
```ini
[Shared]
   path = /srv/shared
   browseable = yes
   read only = no
   writable = yes
   valid users = sambauser
   create mask = 0775
   directory mask = 0775
```

Save and exit (`Ctrl+O`, `Enter`, then `Ctrl+X` in nano).

## 7. Test the Configuration File

Check the config for syntax errors before restarting:
```
testparm
```

If there are no errors, it will print the current config back to you.

## 8. Restart Samba

Apply the changes by restarting the service:
```
sudo systemctl restart smbd
sudo systemctl enable smbd
```

`enable` makes sure Samba starts automatically on boot.

## 9. Allow Samba Through the Firewall

If `ufw` is active, allow Samba traffic:
```
sudo ufw allow samba
```

## 10. Test Access from Windows

1. On a Windows PC, open File Explorer.
2. In the address bar, type:
   ```
   \\<server-ip-address>\Shared
   ```
3. Enter the Samba username and password when prompted.
4. Confirm you can view, add, and edit files in the shared folder.

## 11. Test Access from Linux

On another Linux machine, install a client if needed:
```
sudo apt install smbclient
```

Then connect:
```
smbclient //<server-ip-address>/Shared -U sambauser
```

Or mount it directly:
```
sudo mount -t cifs //<server-ip-address>/Shared /mnt/point -o username=sambauser
```

## Outcome

At this point, Samba is installed, configured, and tested. Both Windows and Linux devices on the network can connect to the shared folder, log in with the Samba user, and read/write files.
