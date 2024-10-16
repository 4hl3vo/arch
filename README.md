# ARCH INSTALLATION

## LAYOUT TECLADO 
	loadkeys br-abnt2

## INTERNET
	ping -c 3 google.com

	iwctl
	device list
	station wlan0 scan
	station wlan0 get-networks
	station wlan0 connect "NOME_DA_REDE"

## PARTICIONAR OS DISCOS 
	lsblk
	
	cgdisk /dev/sdX 
		EFI = EF00
		SWAP = 8200

## SISTEMA DE ARQUIVOS
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

## PACOTES
	pacstrap -i /mnt 
					base 
					base-devel 
					linux 
					linux-headers 
					linux-lts 
					linux-lts-headers 
					linux-firmware
					intel-ucode
					vim 
					git 
					networkmanager 
					dhcpcd 
					wpa_supplicant 
					wireless_tools 
					netctl
					discord 
					firefox 
					spotify-launcher 
					code 
					xorg-xrandr 
					xorg-server 
					xorg-xinit 
					xorg-xinput
					libx11 
					libxinerama 
					libxft 
					webkit2gtk 
					mesa 
					feh 
					picom 
					nvidia 
					nvidia-lts 
					libglvnd 
					nvidia-utils 
					opencl-nvidia 
					nvidia-settings 
					grub 
					efibootmgr 
					dosfstools 
					os-prober 
					mtools      
					alsa-utils 
					pulseaudio
					htop 
					less 
					ttf-fira-code 
					unzip 
					unrar
					libreoffice-still 
					lf 
					scrot 
					usbutils
					qemu-desktop
					virt-manager
					qbittorrent
					xclip
					pavucontrol
		
	genfstab -U /mnt >> /mnt/etc/fstab
		

## CHROOT
    arch-chroot /mnt

## TIMEZONE
	ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
	localectl set-timezone America/Sao_Paulo
	localectl set-keymap --no-convert br-abnt2
	hwclock --systohc

	vim /etc/locale.gen
		#en_US.UTF-8
	echo LANG=en_US.UTF-8 > /etc/locale.conf
	echo KEYMAP=br-abnt2 > /etc/vconsole.conf
	locale-gen

## USER CONFIG
	passwd
	useradd -m guilherme
	usermod -aG wheel,storage,power guilherme
	passwd guilherme
	EDIT=vim visudo
		#%wheel ALL=(ALL) ALL

## HOSTNAME
	echo arch-linux > /etc/hostname
	vim /etc/hosts
		127.0.0.1     localhost
		::1           localhost
		127.0.1.1     arch-linux.localdomain    localhost

## GRUB
	vim /etc/default/grub
		#GRUB_DISABLE_OS_PROBER=false	
	grub-install --target=x86_64-efi --bootloader-id='ARCH LINUX' --recheck
	grub-mkconfig -o /boot/grub/grub.cfg
	
	systemctl enable NetworkManager.service
	nvidia-xconfig
	exit
	
	umount -a
	reboot

## DESKTOP ENVIRONMENT
	mkdir suckless
	cd suckless

	git clone https://git.suckless.org/dwm
	git clone https://git.suckless.org/st
	git clone https://git.suckless.org/dmenu
	git clone https://git.suckless.org/slstatus

	cd dwm
	sudo make clean install

	cd ../st
	patch -i alpha...
	patch -i ligatures-alpha...
	sudo make clean install

	cd ../dmenu
	patch -i dmenu-center..
	sudo make clean install

	cd ../slstatus
	sudo make clean install

	feh --bg-scale ~/Pictures/wallpapers/img.png

	sudo vim /etc/xdg/picom.conf

	vim .xinitrc
		~/suckless/slstatus/slstatus &
		~/.fehbg &
		picom &
		setxkb br &
		exec dwm

	vim .bash_profile
		startx

## AUDIO
	alsamixer
	alsactl restore
	pulseaudio -k
	pulseaudio --start

## NTP
	timedatectl set-ntp true			
	timedatectl status



