# Network Configuration — Step by Step

This guide covers connecting the server to the local network and setting it up with a fixed, reachable address.

## 1. Plan the Network Layout

1. Decide where the server sits on the network: connected to a switch, which connects to the router.
2. Layout:
   ```
   Router  →  Network Switch  →  Server (and other devices)
   ```
3. Using a switch instead of plugging directly into the router lets multiple devices share the connection and keeps traffic organized.

## 2. Physically Connect the Hardware

1. Connect an Ethernet cable from the router to the network switch.
2. Connect another Ethernet cable from the switch to the server's network port.
3. Connect other devices (PCs, laptops) to the same switch or router so they're on the same local network.
4. Power on the switch and confirm the link lights are active on both ends of each cable.

## 3. Check the Server's Network Interface

On the server, list network interfaces to find the correct one (usually something like `enp0s3` or `eth0`):
```
ip a
```

## 4. Confirm Initial Connectivity (DHCP)

Before setting a static IP, confirm the server can reach the network using the router's automatic (DHCP) address:
```
ping -c 4 google.com
```

If this works, the cabling and switch/router are fine, and you can move on to setting a static IP.

## 5. Choose a Static IP Address

1. Check the router's IP range (usually found in the router's admin page, e.g. `192.168.1.1`).
2. Pick an IP address outside the router's DHCP range so it won't conflict with other devices (e.g. `192.168.1.100`).
3. Note the router's gateway address and preferred DNS servers (can use public DNS like `8.8.8.8`).

## 6. Set a Static IP on the Server

Edit the netplan configuration file (name may vary):
```
sudo nano /etc/netplan/00-installer-config.yaml
```

Set the static IP details:
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.1.100/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

Save and exit (`Ctrl+O`, `Enter`, then `Ctrl+X` in nano).

## 7. Apply the Network Changes

```
sudo netplan apply
```

Confirm the new IP is active:
```
ip a
```

## 8. Test Connectivity After the Change

1. From the server, ping the router:
   ```
   ping -c 4 192.168.1.1
   ```
2. Ping an external site to confirm internet access:
   ```
   ping -c 4 google.com
   ```
3. From another device on the network, ping the server's new static IP:
   ```
   ping 192.168.1.100
   ```

## 9. Reserve the IP on the Router (Optional but Recommended)

1. Log into the router's admin page.
2. Find the DHCP reservation / static lease settings.
3. Reserve the chosen IP address for the server's MAC address.
4. This prevents the router from ever handing that IP to another device, even if the server's netplan config is reset.

## 10. Test File and Web Access Over the Network

1. From another device, connect to the Samba share:
   ```
   \\192.168.1.100\Shared
   ```
2. From another device's browser, load the Apache site:
   ```
   http://192.168.1.100
   ```
3. Confirm both work reliably and check transfer/loading speed is stable.

## 11. Check Network Speed and Stability

1. Copy a large file to/from the shared folder and note the transfer speed.
2. Leave the server running and reachable for a few hours/days to confirm the connection stays stable.
3. If speeds are slow, check cable quality and switch port speed (e.g. confirm Gigabit vs 100Mbps ports).

## Outcome

At this point, the server has a fixed IP address, is properly connected through the switch and router, and is reliably reachable by all devices on the local network for both file sharing and web access.
