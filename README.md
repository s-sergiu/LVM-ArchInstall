
# Installing ArchLinux with encrypted LVM partitions (UEFI boot)

After you boot the arch live environment from the USB flash drive and you're 
presented with the zsh shell prompt, the first step will be to set up an internet connection.

To make sure you have your machine is booted in UEFI mode execute the follwing command:
	ls /sys/firmware/efi/efivars
	If the directory doesn't exist and the above command doesn't print anything your system is not booted in UEFI mode (refer to BIOS mode installation -- link to be added --);			

If you are on a wired connection you should plug the cable into your device and have an internet 
connection.
Otherwise if you're on a wireless connection: 
	Start by typing the command: iwctl (you will be presented with an [iwd]# prompt);
	Find out your wireless device name by typing: station list;
	Scan for networks by typing: station <device name> scan;
	List all networks found: station <device name> get-networks;
	And finally connect to your desired network: station <device name> connect <network name>;
	You should be prompted for a password. After you're done just type exit to leave iwd;

Setting up the system clock:
	timedatectl set-ntp true;
	timedatectl set-timezone Europe/Berlin (change it depending on your location);
	timedatectl status (check that everything looks ok);

Before we go on to partitioning the disk, we can reset the disk partition table, remove 
all filsystem signatures from the partitions, remove LVM - logical volumes, volume groups 
and physical volumes (if we have LVM) and decrypt the volume (if it's encrypted) 
so that we can format it afterwards to have a clean disk for our fresh Archlinux install.

# Section for resetting the disks for a fresh install.
	1.First we need to open the encrypted volume if you have encryption enabled, if not just skip this step.
		cryptsetup open --type luks /dev/sda2 <volume name>
	2.Then we need to remove all logical volumes 
		list them with lvs;
		remove volumes with lvremove /dev/mapper/<volume name>
	3.Remove the logical volume group: vgremove /dev/mapper/<volume name>
	4.Finally remove the physical volume: pvremove dev/mapper/<volume name>
	5.Close the encrypted volume (skip if you don't use encryption).
		cryptsetup close /dev/mapper/<volume name>;
	6.Format the disk with ext4 extension: mkfs.ext4 /dev/sda
	7.Wipe all filesystem signatures so you have an empty blank disk: wipefs --all /dev/sda;

Now we begin partitioning the disk:
	We are going to use fdisk to make partitions: fdisk /dev/sda;
		## Section for partitioning the disk with fdisk;
After partitioning is complete we format the EFI boot partition with mkfs.vfat /dev/ <device name> 
or mkfs.fat -F32 /dev/<device name>;
Then we begin by encrypting the disk with the following command: 
	cryptsetup luksFormat <device>
Open the encrypted partition so we can start to create the LVM group and logical volumes:
	cryptsetup open --type luks <device> <name of volume>

Now we begin creating an LVM partition layout, first by creating the physical volume:
	pvcreate <device> "/dev/mapper/name"
We create the volume group:
	vgcreate "volume name" /dev/mapper/name;
And finally create the logical volume groups:
	lvcreate -L <4096MiB> volume_name -n swap;
	lvcreate -L <40G> volume_name -n root;
	lvcreate -L <40G> volume_name -n home;
	lvcreate -L <8G> volume_name -n var;
	lvcreate -L <3G> volume_name -n tmp;

After we created our LVM layout we can start formatting them and mounting + activating the swap.
	mkfs.ext4 /dev/mapper/volume_name-root;
	mkfs.ext4 /dev/mapper/volume_name-home;
	mkfs.ext4 /dev/mapper/volume_name-var;
	mkfs.ext4 /dev/mapper/volume_name-tmp;
	mkswap /dev/mapper/volume_name-swap;

	mount /dev/mapper/volume_name-root /mnt;
	mount -m /dev/mapper/volume_name-home /mnt/home;
	mount -m /dev/mapper/volume_name-var /mnt/var;
	mount -m /dev/mapper/volume_name-tmp /mnt/tmp;
	mount -m /dev/sda1 /mnt/boot;
	swapon /dev/mapper/volume_name-swap;

Before we can install the base filesystem we need to make sure we update the pacman mirrors based on our location.
No we can begin installing the base filesystem and the linux kernel:
	pacstrap -K /mnt base linux linux firmware;
Aditionally we can install a file editor, wifi network manager, lvm2 package and sudo
	pacstrap -K /mnt vim sudo iwd lvm2;

We can generate an fstab file :
	genfstab -U /mnt >> /mnt/etc/fstab
And chroot into the system:
	arch-chroot /mnt
Set up time zone:
	ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime
Generate /etc/adjtime:
	hwclock --systohc
Open /etc/locale.gen and select your desired locale. (ex: en_US.UTF-8, default one);
	vim /etc/locale.gen and uncomment the desired locale;
Create /etc/locale.conf and add the following:
	LANG=en_US.UTF-8
Config your hostname:
	vim /etc/hostname -> insert desired hostname;
Add the following to /etc/hosts:
	127.0.0.1	localhost

Before running mkinitcpio -P we need to modify the config so that we load encryption and lvm preferably after keyboard;
	vim /etc/mkinitconfig.conf 
	go to HOOKS and add encrypt and lvm2 after keyboard and before filesystem;
Generate a root password:
	passwd
And finally create the initramfs:
	mkinitcpio -P

To use systemd-boot as a bootloader we first need to install it:
	bootctl install;
After install we have 2 more steps until we can boot our fresh new installation.
We need to open /boot/loader/loader.conf and add the following:
	default arch.conf
	timeout 3
	console-mode max
	editor no
Open /boot/loader/entries/arch.conf and add the following:
	title Arch Linux
	linux /vmlinuz-linux
	initrd /initramfs-linux.img
	options cryptdevice =UUID=<UUID_NUMBER>:volume root=/dev/mapper/volume-root rw;
to get the UUID_NUMBER use the following command 
	blkid /dev/sda2		

We are done with installing the bootloader, thus we can exit chroot, unmount the usb disk and reboot;
	exit;
	umount -R /mnt;
	reboot;

# Booting for the first time
We can login user the root username and password and create a new user using systemd-homed;
	homectl create user --storage=luks
Add the user to the sudoers file:
	vim /etc/sudoers;
And configure our network using networkd and resolved;
	sudo systemctl enable systemd-networkd;
	sudo systemctl start systemd-resolved;
	sudo systemctl start systemd-networkd;
	sudo systemctl enable systemd-resolved;
Last step into configuring our network we need to create a wireless config profile
Open /etc/systemd/network/25-wireless.network and add the following:
	[Match]
	name=<name of your wlan device>
	[Network]
	DHCP=yes

Finally enable iwd and make sure you configure the connection to your wireless device again;
	sudo systemctl enable iwd;
	reboot and after you can configure the wireless lan connection again;
