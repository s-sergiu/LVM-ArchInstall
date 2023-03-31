
# Installing ArchLinux with encrypted LVM partitions

After you boot the arch live environment from the USB flash drive and you're 
presented with the zsh shell prompt, the first step will be to set up an internet connection.

If you are on a wired connection you should plug the cable into your device and have internet.
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
all filsystem signatures from the partitions, remove (LVM - logical volumes, volume groups and physical volume)
if we have LVM and close the encrypted volume (if it's encrypted) so that we can format it afterwards 
to have a clean disk for our fresh Archlinux install.

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


		

