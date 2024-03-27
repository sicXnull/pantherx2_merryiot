## How To: PantherX2 Running Browan/Merryiot Firmware

 - Download Balena Etcher
   [Download](https://etcher.balena.io/#download-etcher)
 - Download Browan image  [Download](https://github.com/sicXnull/pantherx2_merryiot/releases/tag/1.0)
 - Flash Browan image to SD Card

## SSH Access

Generate a SSH key. 
[Windows](https://www.techtarget.com/searchsecurity/tutorial/How-to-use-PuTTY-for-SSH-key-based-authentication) 
[Linux](https://docs.oracle.com/en/cloud/cloud-at-customer/occ-get-started/generate-ssh-key-pair.html#GUID-8B9E7FCB-CEA3-4FB3-BF1A-FD3406A2432F) 
[MacOS](https://docs.tritondatacenter.com/public-cloud/getting-started/ssh-keys/generating-an-ssh-key-manually/manually-generating-your-ssh-key-in-mac-os-x) 

- Make note of the private key that’s been generated. Linux - `cat /path/to/your/private/key` Windows - Puttygen 
- Mount Browan SD card to a Linux/MacOS machine. 
- Navigate to the root/.ssh directory of the SD card (as root). Example `cd /media/sic/0e2c42bf-d359-42fc-b24d-7565e9b5e2d5/root/.ssh`
- Open `authorized_keys` (`sudo nano authorized_keys`) file, paste your SSH key in the second line. 
- Insert SD card back into Miner
- SSH into Merryiot miner with attached SSH public key. 
- Windows - [Putty Instructions](https://www.techtarget.com/searchsecurity/tutorial/How-to-use-PuTTY-for-SSH-key-based-authentication) 
- Linux/Mac - `ssh -p 168 -i /path/to/your/private/key root@miner-ip`


## How to Keep SSH Access After Updates

- Update Packages `sudo apt update` 
- Install Nano `sudo apt install nano`
- Navigate to ssh config file `sudo nano /etc/ssh/sshd_config` 
- Change `PasswordAuthentication` from `no` to `yes`. Optional - Change port to 22 if you desire. Save file
- Restart SSH `sudo systemctl restart ssh`
- Modify file permissions so it is not overwritten by OTA updates `sudo chattr +i /etc/ssh/sshd_config`
- Set root password. `sudo passwd root`
- Modify file permissions so it is not overwritten `sudo chattr +i /etc/passwd /etc/shadow`

## How to fix MAC Address

- For some reason, the browan image comes hardcoded with a MAC address different than your machines. To correct this:
  - Note your device MAC address
- Install macchanger `sudo apt update && sudo apt-get install macchanger`
- Select No to changing MAC automatically
- Create the following file by running `sudo nano /etc/network/if-pre-up.d/change_mac_address`
  - Paste the following (be sure to change < your:mac:address >):
 ``` #!/bin/bash
#!/bin/bash

# Log file path
LOG_FILE="/var/log/change_mac_address.log"

/usr/bin/macchanger -m your:mac:address eth0 >> "$LOG_FILE" 2>&1

# Check the exit status of the macchanger command
if [ $? -ne 0 ]; then
    echo "Failed to change MAC address. See $LOG_FILE for details." >&2
fi
```
- Save file
- Make script executable `sudo chmod +x /etc/network/if-pre-up.d/change_mac_address`
- Prevent updates from deleting this fix `sudo chattr +i /etc/network/if-pre-up.d/change_mac_address`
- Reboot

## Force OTA Update

To update to the latest firmware, ensure all of the above has been completed. Then run ``/usr/local/sbin/gw_ota stable``

