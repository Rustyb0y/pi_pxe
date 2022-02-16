# Network Booting a Raspberry Pi 4B Using pfsense & TrueNAS

## pfsense Setup

1. Create/Edit DHCP Static Mapping for the Raspberry Pi 4B that you want to add.

![Image](https://github.com/Rustyb0y/pi_pxe/blob/master/images/chrome_twXjfWi6sZ.png)

2. Make sure to add your TrueNAS TFTP Server IP Address.  When the Raspberry Pi asks for it's IP from pfsense, pfsense will direct the Raspberry Pi to the TFTP server.

![Image 2](https://github.com/Rustyb0y/pi_pxe/blob/master/images/chrome_HLhIPvvha0.png)

## TrueNAS Setup

1. Setup the Pool to be used to hold the NFS Boot shares.

![Image4](https://github.com/Rustyb0y/pi_pxe/blob/master/images/chrome_kwQhzdS98Z.png)

2. Setup the Pool to be used to hold the NFS Root shares.

![Image5](https://github.com/Rustyb0y/pi_pxe/blob/master/images/chrome_w7uCbvdbBY.png)

3. Setup the NFS Boot shares making sure to add your authorised network/hosts as required.

![Image6](https://github.com/Rustyb0y/pi_pxe/blob/master/images/chrome_Uj36s1OcuM.png)

4. Setup the NFS Root shares making sure to add your authorised network/hosts as required.

![Image7](https://github.com/Rustyb0y/pi_pxe/blob/master/images/chrome_ZQlNmWkhzt.png)

5. Setup the TFTP Service pointing to the Boot pool.

![Image3](https://github.com/Rustyb0y/pi_pxe/blob/master/images/chrome_YUfjVpVHqh.png)

6. Make sure to run the TFTP server and set it to start automatically

![Image8](https://github.com/Rustyb0y/pi_pxe/blob/master/images/chrome_SLo1Aq9NsH.png)

## Raspberry Pi Setup

### Placeholders

    - [serial] = Raspberry Pi serial number
    - [ip] = TrueNAS IP address
    - [path] = NFS share path e.g. `/mnt/pool/`
    - [hostname] = Raspberry Pi hostname
    - [space] = Make sure there is a space here

1. **Update Pi (Optional)**

    `sudo apt update`

    `sudo apt dist-upgrade`

2. **Setup a specific Raspberry Pi hostname (Optional)**

    `sudo nano /etc/hostname`

    `sudo nano /etc/hosts`

3. **Reboot**

    `sudo reboot`

4. **Create NFS Boot Share**

    Create local folder for NFS boot share
    
    `sudo mkdir -p /nfs/rpi-tftpboot`

    Mount NFS boot share to local folder
    
    `sudo mount -t nfs -O rw,all_squash,anonuid=1001,anongid=1001`[space][ip]`:`[path]`/rpi-tftpboot /nfs/rpi-tftpboot/`

    Get Raspberry Pi serial number 
    
    `vcgencmd otp_dump | grep 28: | sed s/.*://g`

    Create Raspberry Pi serial number folder on NFS share
    
    `sudo mkdir /nfs/rpi-tftpboot/`[serial]

    Copy boot partition files to NFS share
	
    `sudo cp -r /boot/* /nfs/rpi-tftpboot/`[serial]

    Edit cmdline.text on NFS boot share to point to the NFS root share
	
    `sudo nano /nfs/rpi-tftpboot/`[serial]`/cmdline.txt`
		
    Replace contents with the following
        
    `console=serial0,115200 console=tty1 root=/dev/nfs nfsroot=`[ip]`:`[path]`/rpi-pxe/`[hostname]`,vers=3 rw ip=dhcp elevator=deadline rootwait`

5. **Create NFS Root Share**

    Create local folder for NFS root share
    
    `sudo mkdir -p /nfs/rpi-pxe`

    Mount NFS root share to local folder

	`sudo mount -t nfs -O rw,all_squash,anonuid=1001,anongid=1001`[space][ip]`:`[path]`/rpi-pxe /nfs/rpi-pxe`

    Create root folder on NFS root share

	`sudo mkdir /nfs/rpi-pxe/pibox`

    Rsync Raspberry Pi root to NFS root share

	`sudo rsync -xa --progress --exclude /nfs / /nfs/rpi-pxe/`[hostname]

    Edit fstab on NFS root share to mount the NFS boot share

	`sudo nano /nfs/rpi-pxe/pibox/etc/fstab`

    Replace local mounts with NFS boot share mount

	[ip] `:` [path] `/rpi-tftpboot/`[serial][space]`/boot nfs defaults,vers=3,proto=tcp 0 0`

6. **Setup the Raspberry Pi to boot from the network**

    Start the Raspberry Pi configuration tool

    `sudo raspi-config`

    - Select 6 Advanced Options
    - Select A6 Boot Order
    - Select B3 Network Boot
    - Select OK
    - Select Reboot

    Once the Raspberry Pi has rebooted, shut down the Raspberry Pi

    `sudo shutdown now`

    Remove the SD card and power cycle the Raspberry Pi

    - If all is good the Raspberry Pi should boot from the network
    - After booting I would recommend going over the boot log using `journalctl -b` to see if there is anything on your Pi that's going to have an issue with running from an NFS share.
