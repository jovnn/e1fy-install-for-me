# Update mirrors
pacman -Syyyy --noconfirm

export user="jovnn"
export userpass="jovan1"
export rootpass="jovan1"
export hostn="main"
export devicename="/dev/sda"

function make_efi()
{
    printf "g\nn\n\n\n+100MB\nyes\nt\n1\nn\n\n\n\n\nyes\nw\n" | fdisk $devicename
    export first_part="$devicename"1
    export second_part="$devicename"2
    mkfs.fat -F32 $first_part
    mkfs.ext4 $second_part
    mount $second_part /mnt
}

function make_mbr()
{
    printf "o\nn\np\n\n\n\nyes\na\nw\n" | fdisk $devicename
    export first_part="$devicename"1
    mkfs.ext4 $first_part
    mount $first_part /mnt
}

function configure_grub_bios()
{
    chroot /mnt /bin/bash -c "pacman -S grub --noconfirm"
    chroot /mnt /bin/bash -c "grub-install --target=i386-pc $devicename"
    chroot /mnt /bin/bash -c "grub-mkconfig -o /boot/grub/grub.cfg"
}

function configure_grub_efi()
{
    chroot /mnt /bin/bash -c "pacman -S efibootmgr grub --noconfirm"
    chroot /mnt /bin/bash -c "mkdir /boot/efi"
    chroot /mnt /bin/bash -c "mount $first_part /boot/efi"
    chroot /mnt /bin/bash -c "grub-install --target=x86_64-efi --efi-directory=/boot/efi --removable"
    chroot /mnt /bin/bash -c "grub-mkconfig -o /boot/grub/grub.cfg"
}

[ -d /sys/firmware/efi ] && make_efi || make_mbr 

# give base 
pacstrap /mnt base linux-zen linux-firmware vim nano 
genfstab -U /mnt >> /mnt/etc/fstab
cp /etc/resolv.conf /mnt/etc/resolv.conf

# mount chroot preps
mount --types proc /proc /mnt/proc
mount --rbind /sys /mnt/sys
mount --make-rslave /mnt/sys # rslave for systemd bind
mount --rbind /dev /mnt/dev
mount --make-rslave /mnt/dev # rslave for systemd bind

# Generate locales
echo 'en_US.UTF-8 UTF-8' >> /mnt/etc/locale.gen
chroot /mnt /bin/bash -c "locale-gen"

# Hostname
echo "Hostname"
echo "$hostn" > /mnt/etc/hostname 
cat >> /mnt/etc/hosts << EOF
127.0.0.1	localhost
::1	$hostn
127.0.1.1	$hostn
EOF

# Update mirrors
chroot /mnt /bin/bash -c "pacman -Syyyy --noconfirm"

# Install sudo
chroot /mnt /bin/bash -c "pacman -S sudo --noconfirm"

# root password
chroot /mnt /bin/bash -c 'printf "$rootpass\n$rootpass\n" | passwd'

# Username
chroot /mnt /bin/bash -c "useradd -m -g users -G wheel $user"
chroot /mnt /bin/bash -c 'printf "$userpass\n$userpass\n" | passwd $user'
echo "%wheel ALL=(ALL) ALL" >> /mnt/etc/sudoers

[ -d /sys/firmware/efi ] && configure_grub_efi || configure_grub_bios

# install tools
chroot /mnt /bin/bash -c "pacman -S xorg xorg-xinit pulseaudio pavucontrol git base-devel --noconfirm"
chroot /mnt /bin/bash -c "pacman -S wget feh firefox xorg-xsetroot xcompmgr --noconfirm"

# install utils
chroot /mnt /bin/bash -c "pacman -S kitty xfce4-screenshooter --noconfirm"

# enable NetworkManager
chroot /mnt /bin/bash -c "pacman -S networkmanager network-manager-applet --noconfirm"
chroot /mnt /bin/bash -c "systemctl enable NetworkManager.service"

# quick deploy
chroot /mnt /bin/bash -c "pacman -S tor --noconfirm"
chroot /mnt /bin/bash -c "systemctl enable tor.service"

chroot /mnt /bin/bash -c "pacman -S noto-fonts-emoji adobe-source-han-sans-jp-fonts --noconfirm" 
pacman -S i3-status 
#chroot /mnt /bin/bash -c "git clone https://git.suckless.org/dwm;mv dwm /home/$user/ ;cd /home/$user/dwm;make clean install;"
#chroot /mnt /bin/bash -c "git clone https://git.suckless.org/dmenu;mv dmenu /home/$user;cd /home/$user/dmenu;make clean install;"
#chroot /mnt /bin/bash -c "git clone https://github.com/e1fy/dwm-config;mv dwm-config /home/$user;mv /home/$user/dwm-config/config.h /home/$user/dwm/config.h"
#chroot /mnt /bin/bash -c "cd /home/$user/dwm;sudo make clean install"
mkdir /mnt/home/$user/pics
chroot /mnt /bin/bash -c "wget https://images6.alphacoders.com/102/thumb-1920-1020177.jpg;mv *.jpg /home/$user/pics/1020177.jpg"
cat >> /mnt/home/$user/.xinitrc << EOF
/bin/bash /home/$user/.load.sh &
exec i3
EOF
cat >> /mnt/home/$user/.load.sh << EOF
/bin/bash /home/$user/.time.sh &
feh --bg-scale /home/$user/pics/1020177.jpg
xcompmgr -n &
EOF
cat >> /mnt/home/$user/.time.sh << EOF
while true; do 
    xsetroot -name "$(date)"
    sleep 6
done
EOF
reboot
