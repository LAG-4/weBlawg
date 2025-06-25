---
title: How To Make A NAS Using Raspberry Pi
date: 2025-06-26T04:45:03+05:30
draft: false
tags:
  - raspberrypi
  - nas
  - linux
---

Hello all,

In this guide I’ll show you, step by step, how to turn an old Raspberry Pi into a Network Attached Storage (NAS) box and local media server using Samba and Jellyfin. Your Pi won’t bake you a pie, but it’ll serve up your movies and files just fine!

## Why I Built a NAS
I had:
- An old Raspberry Pi (Model 3 B+)
- A lonely external HDD gathering dust  
I wanted to learn more about networking, reuse my hardware, and never let good tech go to waste.

## Requirements
- Raspberry Pi (3 B+ or newer)  
- SD card (≥ 32 GB) with reader  
- Stable 5 V/3 A power supply & quality cable  
- External HDD (I used a 2 TB drive)  
- PC or laptop (Windows, Linux or macOS)

---

## Step 1: Flash Raspberry Pi OS
1. Download **Raspberry Pi Imager** from raspberrypi.org.  
		![[Pasted image 20250626051143.png]]
2. Choose **Raspberry Pi OS Lite** (headless) for minimal overhead.  
3. Select your SD card and write the image.  
4. You can press `ctrl+shift+x` in the imager and set hostname login password and also configure ssh it would be very useful if you dont have an external monitor.
5. Click next and write the changes to the SD card.
6. Eject the SD card and place it into the Raspberry Pi.

---

## Step 2: First Boot & Network Access
1. Insert the SD card, connect Pi → router (Ethernet) or Wi-Fi, then power on.  
		- You can see the Pi's IP address directly from your router page if the router doesnt support it then connect the raspberrypi to a monitor or TV via HDMI and follow the steps below.
2. Find its IP:
   ```bash
   hostname -I
   ```  
3. SSH in:
   ```bash
   ssh pi@<PI_IP>
   # Default password: raspberry
   ```
4. Update packages:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

---

## Step 3: Mount Your External HDD
1. (Optional) Install NTFS support if your drive is NTFS:
   ```bash
   sudo apt install ntfs-3g -y
   ```
2. Plug in the HDD, then check its device name and UUID:
   ```bash
   lsblk -f
   sudo blkid /dev/sda1
   ```
3. Create mount point and edit `fstab`:
   ```bash
   sudo mkdir /mnt/hdd
   sudo nano /etc/fstab
   ```
   Add a line (replace `<UUID>` with your drive’s UUID, adjust `ntfs-3g` or `ext4`):
   ```
   UUID=<UUID>  /mnt/hdd  ntfs-3g  defaults,noatime  0  2
   ```
4. Mount it:
   ```bash
   sudo mount -a
   ls /mnt/hdd
   ```

---

## Step 4: Set Up Samba (Windows-Friendly NAS)
1. Install Samba:
   ```bash
   sudo apt install samba samba-common-bin -y
   ```
2. Create a Samba user (will mirror your Linux `pi` user):
   ```bash
   sudo smbpasswd -a pi
   ```
3. Edit share config:
   ```bash
   sudo nano /etc/samba/smb.conf
   ```
   Append at the end:
   ```
   [NAS]
      path = /mnt/hdd
      browseable = yes
      read only = no
      guest ok = no
      create mask = 0664
      directory mask = 0775
   ```
4. Restart Samba:
   ```bash
   sudo systemctl restart smbd
   ```
5. From Windows Explorer, go to `\\<PI_IP>\NAS`, enter your Samba credentials, and voilà!

---

## Step 5: Install Jellyfin (Your Personal Netflix)
1. Add Jellyfin’s repo and key:
   ```bash
   sudo apt install -y apt-transport-https gnupg2
   curl -fsSL https://repo.jellyfin.org/debian/jellyfin_team.gpg.key \
     | sudo gpg --dearmor -o /usr/share/keyrings/jellyfin.gpg
   echo "deb [signed-by=/usr/share/keyrings/jellyfin.gpg] \
     https://repo.jellyfin.org/debian \
     $(. /etc/os-release && echo $VERSION_CODENAME) main" \
     | sudo tee /etc/apt/sources.list.d/jellyfin.list
   sudo apt update
   ```
2. Install Jellyfin:
   ```bash
   sudo apt install jellyfin -y
   ```
3. Enable & start the service:
   ```bash
   sudo systemctl enable --now jellyfin
   ```
4. In your browser, navigate to `http://<PI_IP>:8096` and follow the setup wizard.  
   - Point Jellyfin’s media library to `/mnt/hdd/YourMediaFolder`.

---

## Final Thoughts
– Keep your Pi cool (a heatsink helps).  
– Use your router’s DHCP reservation for a stable IP.  
– Back up configs if you get too adventurous.  

Congratulations! Your Pi is now a lean, mean NAS & media server. Time to binge that show—no subscription required. Enjoy!



