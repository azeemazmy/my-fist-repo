# File Server and Web Server Setup Using Ubuntu Server on an Old PC

Repurposing an old PC into a working file server and web server using Ubuntu Server.

**Duration:** 1 year
**Technologies:** Ubuntu Server, Apache, Samba, Networking, Old PC hardware

## Overview

This project turns an old PC into a file server and web server. It uses Samba for file sharing and Apache for hosting a website. The setup uses a router and network switch to connect all devices on a local network.

## Objectives

- **File Server:** Set up Samba on Ubuntu Server for file sharing across devices on the local network.
- **Web Server:** Configure Apache to host a website for testing and showcasing web apps.
- **Networking:** Use a router and switch to build a stable local network for file transfers and device communication.

## Key Steps

1. **System Setup** — Installed Ubuntu Server on the old PC and set up partitions and file systems.
2. **File Server Configuration** — Set up Samba so both Windows and Linux devices can access shared files.
3. **Web Server Configuration** — Installed and configured Apache to serve a static website and test web app.
4. **Network Configuration** — Connected all devices with a network switch for stable, fast access.
5. **Security** — Added basic security: firewall setup and access control for file and web services.

## Outcomes

- Turned an old PC into a reliable file and web server.
- Gained hands-on experience with server management, networking, and security.
- Improved skills in system administration and network configuration.

# **Installing Ubuntu Server — Step by Step**

This guide covers the actual installation process for Ubuntu Server, from creating the boot media to first login.

## **1. Download Ubuntu Server**

1. Go to the official Ubuntu website: https://ubuntu.com/download/server
2. Download the latest *Ubuntu Server LTS* ISO file (LTS = Long Term Support, more stable).
3. (Optional but recommended) Verify the download using the checksum listed on the site, to make sure the file isn't corrupted.

## 2. Create a Bootable USB Drive

**On Windows:**
1. Download and open **Rufus** (free tool).
2. Insert a USB drive (8GB or larger). All data on it will be erased.
3. In Rufus, select the USB drive under "Device."
4. Click "Select" and choose the Ubuntu Server ISO file.
5. Leave other settings as default and click "Start."
6. Wait for the process to finish, then safely eject the USB.

**On Linux/Mac:**
1. Open a terminal.
2. Find the USB device name using:
   
   lsblk
   
3. Write the ISO to the USB (replace sdX with your USB device, and be careful — this erases the drive):
   
   sudo dd if=ubuntu-server.iso of=/dev/sdX bs=4M status=progress
   

## 3. Boot From the USB

1. Plug the USB into the old PC.
2. Power it on and enter the boot menu or BIOS/UEFI settings (usually by pressing F2, F12, Del, or Esc — depends on the motherboard).
3. Set the USB drive as the first boot device, or select it directly from the boot menu.
4. Save and exit. The PC should boot into the Ubuntu Server installer.

## 4. Start the Installer

1. On the first screen, select your **language**.
2. Choose your **keyboard layout** (or let it auto-detect).
3. Select **"Install Ubuntu Server"** (not the cloud image option).

## 5. Network Configuration

1. The installer will detect network interfaces (Ethernet is recommended for a server).
2. If a DHCP connection is found, it will auto-configure temporarily. You can set a static IP later after installation.
3. Continue to the next screen.

## 6. Proxy and Mirror Settings

1. Leave the proxy field blank (unless your network requires one).
2. Leave the Ubuntu archive mirror as default, then continue.

## 7. Storage Configuration (Partitioning)

1. Choose **"Custom storage layout"** instead of the guided option, to control partitioning manually.
2. Select the disk to install on.
3. Create the following partitions:
   - **Root partition (/)** — main OS partition (e.g. 20–30GB).
   - **Swap partition** — typically equal to or half of your RAM size.
   - **Separate partition for shared files** (e.g. /data or /srv) — for files and web content, kept apart from the OS.
4. Format all partitions as **ext4**.
5. Confirm and apply the changes. This will erase the disk.

## 8. Profile Setup

1. Enter your **name**, a **server name** (hostname), a **username**, and a **password**.
2. This account will have sudo (admin) privileges.

## 9. SSH Setup

1. When prompted, check the box for **"Install OpenSSH server."**
2. This lets you manage the server remotely later instead of needing a monitor and keyboard connected.

## 10. Featured Server Snaps (Optional Software)

1. The installer may offer optional packages (Docker, Nextcloud, etc.).
2. Skip these for now — you'll install Samba and Apache manually in later steps.

## 11. Installation

1. The installer copies files and sets up the system. This can take several minutes.
2. Once done, remove the USB drive when prompted.
3. Select **"Reboot Now."**

## 12. First Login

1. After reboot, log in using the username and password created earlier.
2. Update the system before doing anything else:
   
   sudo apt update
   sudo apt upgrade -y
   

## 13. Set a Static IP (Recommended)

1. Find your network interface name:
   
   ip a
   
2. Edit the netplan config file (name may vary, e.g. 00-installer-config.yaml):
   
   sudo nano /etc/netplan/00-installer-config.yaml
   
3. Set a static IP, gateway, and DNS, for example:
   yaml
   network:
     version: 2
     ethernets:
       enp0s3:
         dhcp4: no
         addresses: [192.168.1.100/24]
         gateway4: 192.168.1.1
         nameservers:
           addresses: [8.8.8.8, 8.8.4.4]
   
4. Apply the changes:
   
   sudo netplan apply
   

## 14. Confirm Everything Works

1. Test internet access:
   
   ping google.com
   
2. Test SSH access from another computer on the network:
   
   ssh username@192.168.1.100
   

## Outcome

"At this point, Ubuntu Server is fully installed, updated, and reachable on the network with a fixed IP address and SSH access — ready for the next step of installing Samba"  
