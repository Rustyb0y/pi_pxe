# Network Booting a Raspberry Pi 4B Using pfsense & TrueNAS

## pfsense Setup

## TrueNAS Setup

## Raspberry Pi Setup

- Placeholders

    - [serial] = Raspberry Pi serial number
    - [ip] = TrueNAS IP address
    - [path] = NFS share path e.g. `/mnt/pool/`
    - [hostname] = Raspberry Pi hostname
    - [space] = Make sure there is a space here

1. Update Pi (Optional)

    `sudo apt update`
    `sudo apt dist-upgrade`

2. Setup a specific hostname (Optional)

    `sudo nano /etc/hostname`
    `sudo nano /etc/hosts`

3. Reboot

    `sudo reboot`

4. Create NFS Boot Share

    Create local folder for NFS boot share
    
    `sudo mkdir -p /nfs/rpi-tftpboot`

    Mount NFS boot share to local folder
    
    `sudo mount -t nfs -O rw,all_squash,anonuid=1001,anongid=1001` [space][ip] `:` [path] `/rpi-tftpboot /nfs/rpi-tftpboot/`

    Get Raspberry Pi serial number 
    
    `vcgencmd otp_dump | grep 28: | sed s/.*://g`

    Create Raspberry Pi serial number folder on NFS share
    
    `sudo mkdir /nfs/rpi-tftpboot/` [serial]

    Copy boot partition files to NFS share
	
    `sudo cp -r /boot/* /nfs/rpi-tftpboot/` [serial]

    Edit cmdline.text on NFS boot share to point to the NFS root share
	
    `sudo nano /nfs/rpi-tftpboot/` [serial] `/cmdline.txt`
		
    Replace contents with the following
        
    `console=serial0,115200 console=tty1 root=/dev/nfs nfsroot=` [ip] `:` [path] `/rpi-pxe/` [hostname] `,vers=3 rw ip=dhcp elevator=deadline rootwait`

5. Create NFS Root Share

    Create local folder for NFS root share
    
    `sudo mkdir -p /nfs/rpi-pxe`

    Mount NFS root share to local folder

	`sudo mount -t nfs -O rw,all_squash,anonuid=1001,anongid=1001` [space][ip] `:` [path] `/rpi-pxe /nfs/rpi-pxe`

    Create root folder on NFS root share

	`sudo mkdir /nfs/rpi-pxe/pibox`

    Rsync Raspberry Pi root to NFS root share

	`sudo rsync -xa --progress --exclude /nfs / /nfs/rpi-pxe/` [hostname]

    Edit fstab on NFS root share to mount the NFS boot share

	`sudo nano /nfs/rpi-pxe/pibox/etc/fstab`

    Replace local mounts with NFS boot share mount

	[ip] `:` [path] `/rpi-tftpboot/` [serial] ` /boot nfs defaults,vers=3,proto=tcp 0 0`
