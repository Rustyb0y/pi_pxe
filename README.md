# Network Booting a Raspberry Pi 4B Using pfsense & TrueNAS

## pfsense Setup

## TrueNAS Setup

## Raspberry Pi Setup

1. **Update Pi (Optional)**

    `sudo apt update`
    `sudo apt dist-upgrade`

2. **Setup a specific hostname (Optional)**

    `sudo nano /etc/hostname`
    `sudo nano /etc/hosts`

3. **Reboot**

    `sudo reboot`

4. **Create NFS Boot Share**

    1. Create local folder

        `sudo mkdir -p /nfs/rpi-tftpboot`

    2. Mount TrueNAS NFS share to local share

	    `sudo mount -t nfs -O rw,all_squash,anonuid=1001,anongid=1001 10.0.0.3:/mnt/Vault/Pis/rpi-tftpboot /nfs/rpi-tftpboot/`

    3. Get Raspberry Pi serial number

	    `vcgencmd otp_dump | grep 28: | sed s/.*://g`

    4. Create Raspberry Pi serial number folder on TrueNAS NFS share
	
	    `sudo mkdir /nfs/rpi-tftpboot/[pi serial number]`

    5. Copy boot partition to TrueNAS NFS share

	    `sudo cp -r /boot/* /nfs/rpi-tftpboot/[pi serial number]`

    6. Edit cmdline.text on TrueNAS NFS boot share to point to the TrueNAS NFS root share

	    `sudo nano /nfs/rpi-tftpboot/[pi serial number]/cmdline.txt`
		
        Replace contents with the following

        `console=serial0,115200 console=tty1 root=/dev/nfs nfsroot=10.0.0.3:/mnt/Vault/Pis/rpi-pxe/pibox,vers=3 rw ip=dhcp elevator=deadline rootwait`




5. **Create NFS Root Share**
