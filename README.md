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

    - Create local folder

        `sudo mkdir -p /nfs/rpi-tftpboot`

    - Mount TrueNAS NFS share to local share

	    `sudo mount -t nfs -O rw,all_squash,anonuid=1001,anongid=1001 10.0.0.3:/mnt/Vault/Pis/rpi-tftpboot /nfs/rpi-tftpboot/`

    - Get Raspberry Pi serial number

	    `vcgencmd otp_dump | grep 28: | sed s/.*://g`

    - Create Raspberry Pi serial number folder on TrueNAS NFS share
	
	    `sudo mkdir /nfs/rpi-tftpboot/[pi serial number]`

    - Copy boot partition to TrueNAS NFS share

	    `sudo cp -r /boot/* /nfs/rpi-tftpboot/[pi serial number]`

    - Edit cmdline.text on TrueNAS NFS boot share to point to the TrueNAS NFS root share

	sudo nano /nfs/rpi-tftpboot/[pi serial number]/cmdline.txt
		console=serial0,115200 console=tty1 root=/dev/nfs nfsroot=10.0.0.3:/mnt/Vault/Pis/rpi-pxe/pibox,vers=3 rw ip=dhcp elevator=deadline rootwait




5. **Create NFS Root Share**
