# pi-fleet
### Goal
Manage fleet of raspberry pies with Ansible scripts using Raspbian OS. 
### Real Goal 
I am lazy so want to be able to deploy code patches and OS updates with a single click.
### Parts
My fleet is a mix of (_but honestly any Raspberry Pi should be fine_):
* [Raspberry Pi Model Zero W](https://www.adafruit.com/product/3400)
    - Single core ARM processor, small form factor, built in Wifi    
* [Raspberry Pi Model 2B](https://www.adafruit.com/product/2358)
    - Older model, 4 core ARM processor, built in ethernet
* [Raspberry Pi Model 3B](https://www.adafruit.com/product/3055)
    - Newer model, 4 core ARM processor, built in Wifi and ethernet
* [Raspberry Pi Model B+](https://www.adafruit.com/product/1914)
    - Old model, single core processor, built in ethernet
* [Raspberry Pi Model Zero](https://www.adafruit.com/product/2885)
    - Single core ARM processor, small form factor, built in Wifi    

_All of which can be purchased on [Amazon](https://amazon.com) or at my favorite gadget store [Adafruit](https://www.adafruit.com/)._

##### Recommendation
Buy the Raspberry Pi Model 3B if you need some horsepower. 
Otherwise get a Raspberry Pi Model Zero W especially if you need a small form factor.
Skip the Raspberry Pi Model B+, Raspberry Pi Model Zero W is cheaper, smaller, has built in wifi and is just as powerful.


### Steps for a new Pi (non-graphical)
_Assumption: using Mac OS command line_
1. Download latest [raspbian image](https://www.raspberrypi.org/downloads/raspbian/)

1. Install image on available SD card ([instructions](https://www.raspberrypi.org/documentation/installation/installing-images/README.md))

1. As of the November 2016 release, Raspbian has the SSH server disabled by default. 
You will have to enable it manually.
SSH can be enabled by placing an empty file named 'ssh', without any extension, onto the boot partition of the SD card.
While the SD card is still mounted:
    ```bash
    sudo -i
    cd /Volumes/boot
    touch ssh
    ```

1. Join your Pi to your network:
    - For Pies with wifi adapter only
        While the SD card is still mounted:
        ```bash
        sudo -i
        cd /Volumes/boot
        ```
        Create a wpa_supplicant.conf file for your specific network in this directory. 
        - Example: wpa_supplicant.conf.example
        
1. Find your Pi on the network
    1. Insert SD card into new Pi, connect live ethernet cable (if applicable) from local network, and power up the Pi   
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
    
1. Copy your ssh keys onto the new Pi (insert your IP address (or maybe raspberrypi.local) for the example IP of 192.168.1.24)
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