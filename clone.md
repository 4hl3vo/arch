1. Boot from a Live USB
You should boot into a Linux live environment (such as an Arch Linux or Ubuntu Live USB) to ensure that none of the partitions are mounted during the cloning process.

Ensure the Data Fits: Before cloning, make sure that the used space on the HDD partition is less than or equal to 240GB. You can check this with the df -h command:

$ df -h /dev/sdb1

2. Identify the Partitions

$ lsblk or fdisk -l command to identify your partitions. You need to identify:

The source partition where Arch Linux is currently installed (on the HDD).
The target partition where you want to clone Arch Linux (on the SSD).
For example:

$ lsblk

Let's assume:

The HDD Arch Linux partition is /dev/sdb1
The SSD unallocated space was partitioned and is now /dev/sda2

3. Clone the Partition

Resize the Filesystem on HDD: If the filesystem on the HDD takes up more than 240GB, you need to shrink it before cloning. For example, if it's an ext4 filesystem, you can shrink it using resize2fs:

$ sudo resize2fs /dev/sdb1 240G

You can use the dd command to clone the partition. Run the following command:

$ sudo dd if=/dev/sdb1 of=/dev/sda2 bs=64K conv=noerror,sync status=progress

$ sudo e2fsck -f /dev/sdb1 

5. Update the Fstab
Mount the cloned partition and update the /etc/fstab file with the new UUID of the SSD partition.

$ sudo mount /dev/sda2 /mnt
$ sudo blkid /dev/sda2

Find the UUID and replace the old one in the /mnt/etc/fstab file.

6. Install the Bootloader
If your bootloader (likely GRUB) is installed on the HDD, you need to reinstall it on the SSD.

First, mount the necessary filesystems:

$ sudo mount /dev/sda2 /mnt
$ sudo mount --bind /dev /mnt/dev
$ sudo mount --bind /proc /mnt/proc
$ sudo mount --bind /sys /mnt/sys

Then, chroot into the installed system

$ sudo arch-chroot /mnt

Reinstall GRUB on the SSD:

$ grub-install /dev/sda
$ grub-mkconfig -o /boot/grub/grub.cfg

Exit the chroot environment and unmount the filesystems:

$ exit
$ sudo umount -R /mnt

7. Reboot
Now, reboot your system and ensure that Arch Linux boots from the SSD.

$ sudo reboot

If everything went well, your Arch Linux installation should now be running from the SSD. You can verify this by using lsblk or df -h.