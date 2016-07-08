# pi-fleet

#### Ansible scripts to automate installs and updating of my Raspberry Pies

### Steps for a new Pi (non-graphical)

1. Download latest raspbian image from https://www.raspberrypi.org/downloads/raspbian/

1. Install image on available SD card https://www.raspberrypi.org/documentation/installation/installing-images/README.md

1. Join your Pi to your network:
    - For Pi Zeros (without the built in ethernet adapter)
    
        1. With your fresh SD card still mounted, open up the boot partition (example: mac)
            ```bash
            cd /Volumes/boot
            ```
            
        1. Add to the tail end of the config.txt then save file:
            ```
            dtoverlay=dwc2
            ```
        
        1. Edit cmdline.txt, inserting **modules-load=dwc2,g_ether** after **rootwait**.
        something like:
            ```
            dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait modules-load=dwc2,g_ether quiet init=/usr/lib/raspi-config/init_resize.sh
            ```
        
        1. Eject the mounted disk, something like (mounted disk may very, disk4 on my box):
            ```bash
            diskutil unmountDisk /dev/disk4
            ```
        
        1. Insert SD card into new Pi and plus USB cable from Mac USB port into USB port on Pi Zero.
        
        1. Pi Zero should boot (drawing power from Mac USB port) within about 60 seconds
        
        1. RNDIS/Ethernet Gadget should now show in Network Properties on Mac
        
        1. Should be able to ssh into the pi via:
            ```bash
            ssh pi@raspberrypi.local
            ```
        
        1. Enable internet on Pi Zero by starting Internet Sharing on Mac and setting **To computers using** to the **RNDIS** entry.
        
        1. This will probably break your current ssh connection into Pi. However, within a few seconds, the Pi should reset network interface with new IP address and be internet aware.
        
    - For Pies with built-in ethernet adapters (everybody else)
        1. Insert SD card into new Pi, connect live ethernet cable (from local network), and power up the Pi
    
        1. On your desktop/laptop, populate you network cache (example shown using my local network CIDR): 
            ```bash
            nmap -sP 192.168.1.0/24
            ```
        
        1. Look for the IP address of your new Pi
            ```bash
            arp -na | grep b8:27:eb
            ```
           This will display the IP addresses of all the Pies in your local network, you may have to determine which one is your new Pi.
           *Let's assume the IP address of the new Pi is 192.168.1.24*   
   
1. Make sure you have generated your local ssh keys. If not:
    ```bash
    ssh-keygen
    ```
    
1. Copy your ssh keys onto the new Pi:
    *insert your IP address (or maybe raspberrypi.local) for the example IP of 192.168.1.24
    ```bash
    cat ~/.ssh/id_rsa.pub | ssh pi@192.168.1.24 "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"
    ```
   You will be prompted for the password for the new Pi, **default: raspberry**

1. In pi-fleet working directory, update variables for site specific values:
   - host_vars/*
   - roles/*/vars
   - hosts (initially set hostname and IP address of new Pi)
   
   ```
      [site]
        my_new_pi         ansible_host=192.168.1.24
   ```

1. Run ansible scripts:
    ```
    ansible-playbook -s site.yml --limit my_new_pi --ask-vault-pass
    ```
    *The ask-vault-pass is used if you are using an encrypted vault to store variables*
    *Note: Had to disable bluetooth on new Pi 
    ```
    update-rc.d bluetooth disable
    ```