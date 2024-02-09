=================   arch linux installation ====================


##	LAYOUT TECLADO 
		loadkeys br-abnt2





##	INTERNET
		ping -c 3 google.com

		//	iwctl
		//	device list
		//	station wlan0 scan
		//	station wlan0 get-networks
		//	station wlan0 connect "NOME_DA_REDE"





##	NTP

		timedatectl set-ntp true			
		timedatectl status


	



##	PARTICIONAR OS DISCOS 
		lsblk
		
		cgdisk /dev/sdX 
			// EFI = EF00
			// SWAP = 8200
	



##	CRIAR O SISTEMA DE ARQUIVOS

		mkfs.fat -F32 /dev/sda1
		mkswap /dev/sda2
		mkfs.ext4 /dev/sda3
		mkfs.ext4 /dev/sda4

		swapon /dev/sda2
		
		mount /dev/sda3 /mnt
		
		mkdir /mnt/home
  		mkdir -p /mnt/boot/efi
		
  		mount /dev/sda4 /mnt/home
		mount /dev/sda1 /mnt/boot/efi






##	PACKAGES

		pacstrap -i /mnt base base-devel linux linux-headers linux-lts linux-lts-headers linux-firmware
				 intel-ucode vim git networkmanager dhcpcd wpa_supplicant wireless_tools netctl
				 discord firefox spotify-launcher code xorg-server xorg-init libx11 libxinerama 
				 libxft webkit2gtk mesa feh picom
		
		genfstab -U /mnt >> /mnt/etc/fstab
		
		arch-chroot /mnt
  	
		ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
		hwclock --systohc
		systemctl enable NetworkManager
	
		vim /etc/locale.gen
			#en_US.UTF-8
		echo LANG=en_US.UTF-8 > /etc/locale.conf
		echo KEYMAP=br-abnt2 > /etc/vconsole.conf
		locale-gen
	
		passwd
		useradd -m guilherme
		usermod -aG wheel,storage,power guilherme
		passwd guilherme
		EDIT=vim visudo
			#%wheel ALL=(ALL) ALL

	echo arch-linux > /etc/hostname
	vim /etc/hosts
		127.0.0.1     localhost
		::1           localhost
	127.0.1.1     arch-linux.localdomain    localhost







##	GRUB

	pacman -S grub efibootmgr dosfstools os-prober mtools
	
	vim /etc/default/grub
		#GRUB_DISABLE_OS_PROBER=false
	
	grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
	grub-mkconfig -o /boot/grub/grub.cfg
	exit
	umount -a
	reboot








##	NVIDIA

	sudo pacman -S nvidia nvidia-lts libglvnd nvidia-utils opencl-nvidia lib32-libglvnd lib32-nvidia-utils lib32-opencl-nvidia nvidia-settings
	
	mkdir -p /etc/X11/xorg.conf.d
 
	vim /etc/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf
		Section "OutputClass"
    			Identifier "intel"
    			MatchDriver "i915"
    			Driver "modesetting"
		EndSection

		Section "OutputClass"
    			Identifier "nvidia"
    			MatchDriver "nvidia-drm"
    			Driver "nvidia"
    			Option "AllowEmptyInitialConfiguration"
    			Option "PrimaryGPU" "yes"
    			ModulePath "/usr/lib/nvidia/xorg"
    			ModulePath "/usr/lib/xorg/modules"
		EndSection
	
	mkdir -p /usr/share/gdm/greeter/autostart
	mkdir -p /etc/xdg/autostart

	vim /usr/share/gdm/greeter/autostart/optimus.desktop
		[Desktop Entry]
		Type=Application
		Name=Optimus
		Exec=sh -c "xrandr --setprovideroutputsource modesetting NVIDIA-0; xrandr --auto"
		NoDisplay=true
		X-GNOME-Autostart-Phase=DisplayServer

	vim /etc/xdg/autostart/optimus.desktop
		[Desktop Entry]
		Type=Application
		Name=Optimus
		Exec=sh -c "xrandr --setprovideroutputsource modesetting NVIDIA-0; xrandr --auto"
		NoDisplay=true
		X-GNOME-Autostart-Phase=DisplayServer
	
	mkdir -p /etc/modprobe.d
	
	vim /etc/modprobe.d/nvidia-drm-nomodeset.conf
		options nvidia-drm modeset=1
	
	vim /etc/pacman.d/hooks/nvidia.hook
		[Trigger]
		Operation=Install
		Operation=Upgrade
		Operation=Remove
		Type=Package
		Target=nvidia
		Target=linux
		# Change the linux part above if a different kernel is used

		[Action]
		Description=Update NVIDIA module in initcpio
		Depends=mkinitcpio
		When=PostTransaction
		NeedsTargets
		Exec=/bin/sh -c 'while read -r trg; do case $trg in linux*) exit 0; esac; done; /usr/bin/mkinitcpio -P'

	sudo mkinitcpio -P
	sudo nvidia-xconfig
	
	systemctl enable NetworkManager.service
	exit
	umount -lR /mnt
	reboot
	




-#-#- DESKTOP ENVIRONMENT -#-#-
	
	mkdir suckless
	cd suckless

	git clone https://git.suckless.org/dwm
	git clone https://git.suckless.org/st
	git clone https://git.suckless.org/dmenu

	cd dwm
	sudo make clean install
	
	cd ../st
	patch -i alpha...
	sudo make clean install
	
	cd ../dmenu
	patch -i dmenu-center..
	sudo make clean install

	vim .xinitrc
		~/.fehbg &
		picom &
		exec dwm
	
	cd suckless/st
	sudo vim config.h
		ESC /SHCMD >> "/bin/sh" >> "/usr/local/bin/st"

	vim .bash_profile
		startx
