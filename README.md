# pi-fleet

#### Ansible scripts to automate installs and updating of my Raspberry Pies

### Steps for a new Pi (non-graphical)

1. Download latest raspbian image from https://www.raspberrypi.org/downloads/raspbian/

1. Install image on available SD card https://www.raspberrypi.org/documentation/installation/installing-images/README.md

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