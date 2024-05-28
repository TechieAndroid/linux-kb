[[Proxmox VMs]] [[arch-docker-2]] [[Docker]] [[Diagram]] 

# Installation
```bash
cat /sys/firmware/efi/fw_platform_size
```

```bash
iwctl
```

```bash
station wlan0 connect <ssid>
```

```bash
ping 1.1.1.1
```

```bash
passwd
```

```bash
IP_ADDRESS=$(ip -4 addr show scope global | grep inet | awk '{print $2}' | cut -d/ -f1 | head -n 1);
echo $IP_ADDRESS
```

```bash
ssh root@<ip>
```

```bash
disk="/dev/sda";
disk1="${disk}1";
disk2="${disk}2";
disk3="${disk}3"
```

```bash
disk="/dev/nvme0n1";
disk1="${disk}p1";
disk2="${disk}p2";
disk3="${disk}p3"
```

```bash
gdisk $disk
```

```bash
cfdisk $disk
```

```bash
mkfs.fat -F 32 $disk1;
mkswap $disk2;
swapon $disk2;
mkfs.xfs $disk3;
mount $disk3 /mnt;
mkdir -p /mnt/boot;
mount $disk1 /mnt/boot
```

```bash
timedatectl set-ntp true;
timedatectl set-timezone America/New_York; 
hwclock --systohc
```

```bash
reflector -l 20 --sort rate -p https -c 'United States' -a 12 --save /etc/pacman.d/mirrorlist;
cat /etc/pacman.d/mirrorlist
```

```bash
pacman -Sy archlinux-keyring;
pacman -Su;
pacstrap -K /mnt base linux-zen linux-firmware
```

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
---
```bash
arch-chroot /mnt
```

```bash
#for sda
disk="/dev/sda";
disk1="${disk}1";
disk2="${disk}2";
disk3="${disk}3"
```

```bash
#for nvme0n1
disk="/dev/nvme0n1";
disk1="${disk}p1";
disk2="${disk}p2";
disk3="${disk}p3"
```

```bash
pacman -S efibootmgr networkmanager vim openssh sudo
```

```bash
systemctl enable NetworkManager ; systemctl enable sshd 
```

skip if you want to ssh with password after
```bash
sed -i -e 's/^#PasswordAuthentication yes$/PasswordAuthentication no/' -e 's/^PermitRootLogin yes$/PermitRootLogin no/' /etc/ssh/sshd_config
```

```bash
useradd -m tux ; passwd tux
```

```bash
echo "tux ALL=(ALL) ALL" >> /etc/sudoers ; echo "Defaults passwd_timeout=0" >> /etc/sudoers
```

```bash
ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime
```

```bash
hwclock --systohc
```

```bash
sed -i 's/#en_US.UTF-8/en_US.UTF-8/g' /etc/locale.gen
```

```bash
locale-gen
```

```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

```bash
echo "thinkpad" > /etc/hostname
```

```bash
cat << EOF > /etc/hosts
127.0.0.1 localhost
::1 localhost
127.0.1.1 thinkpad.localdomain thinkpad
EOF
````

```bash
passwd
```

>GRUB
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

>EFI Stub

nvme example
```bash
efibootmgr --create --disk /dev/nvme0n1 --label "archzen" --loader /vmlinuz-linux-zen --unicode 'root=/dev/nvme0n1p3 rw initrd=\initramfs-linux-zen.img'
```
sata example
```bash
efibootmgr --create --disk /dev/sda --label "archzen" --loader /vmlinuz-linux-zen --unicode 'root=/dev/sda3 rw initrd=\initramfs-linux-zen.img'
```
using short flags and uuid
```bash
efibootmgr -c -d /dev/<disk> -L "archzen" -l /vmlinuz-linux-zen -u 'root=UUID=<uuid-string> rw initrd=\<amd/intel>-ucode.img initrd=\initramfs-linux-zen.img'
```

```bash
initrd=\amd-ucode.img
initrd=\intel-ucode.img
```
must exist in `/boot/`
other kernel flags
nouveau.modset=0 amd_iommu=off idle=nomwait amdgpu.gpu_recovery=1

to remove boot entries:
```bash
efibootmgr -B -b <#>
```

```bash
exit
```

```bash
umount -R /mnt ; reboot
```

---
# Post Install

##### Automation of tasks
>fstrim
```bash
sudo systemctl enable fstrim.timer
```

>cron task to update, log, and reboot
```bash
crontab -e
```

```bash
0 4 * * * /usr/bin/pacman -Syu --noconfirm > /home/tux/updates/pacman_update_$(date +\%Y\%m\%d_\%H\%M\%S).log 2>&1 && /usr/bin/reboot
```

>reflector
```bash
sudo pacman -S reflector
```

```bash
sudo rm -rf /etc/xdg/reflector/reflector.conf
```

```bash
sudo mkdir -p /etc/xdg/reflector/reflector.conf
```

```bash
echo "--latest 20 --sort rate --protocol https -c 'United States' --age 12 --save /etc/pacman.d/mirrorlist" | sudo tee -a /etc/xdg/reflector/reflector.conf
```

```bash
sudo systemctl enable --now reflector.timer
```

>pacman eyecandy
```bash
sudo sed -i '/^#Color/s/^#//' /etc/pacman.conf && sudo sed -i '/^Color/a ILoveCandy' /etc/pacman.conf
```

>Post install
```
sudo pacman -S ufw lsd neofetch htop bpytop git docker docker-compose fish
```
>Optional
```
wget curl nano vim
```

>Optional
```bash
xdg-user-dirs-update
```

>ufw
```bash
sudo ufw enable ; sudo ufw default deny incoming ; sudo ufw default allow outgoing ; sudo ufw allow ssh ; sudo ufw logging off ; sudo ufw status verbose
```

>For Minecraft Java Edition open port 25565:
```bash
ufw allow 25565
```

>Docker
```bash
sudo systemctl enable --now docker
```

```bash
mkdir -p docker
```

```bash
sudo usermod -aG docker tux
```

```bash
newgrp docker
```

>Plasma Full
```bash
sudo pacman -S plasma-meta kde-applications-meta
```

>SDDM
```bash
sudo systemctl enable sddm
```

---
# Misc
##### Removing and Adding Users and Groups
```bash
vim /etc/passwd
useradd -m username
userdel username
useradd -m username -g groupname
groupadd groupname
groupdel groupname
```

##### Package Removal Along with Dependencies Not Used by Other Packages
>(s) Removes dependencies that aren't used by other applications.
>(n)
>(R) Remove package(s)
```bash
pacman -Rns plasma-meta kde-applications-meta
```

---
# Security
##### Enforce a delay after a failed login attempt
```bash
echo "auth optional pam_faildelay.so delay=60000000" | sudo tee -a /etc/pam.d/system-login
```

---
# Pacman
##### Fix Updates
```bash
sudo timedatectl status ; sudo timedatectl set-ntp true ; sudo timedatectl set-timezone America/New_York ; sudo hwclock --systohc ; sudo timedatectl status
```
```bash
sudo pacman -S reflector
```
```bash
sudo reflector --latest 20 --sort rate --protocol https -c 'United States' --age 12 --save /etc/pacman.d/mirrorlist
```
```bash
cat /etc/pacman.d/mirrorlist
```
```bash
sudo pacman -Sy archlinux-keyring
```
```bash
sudo pacman -Syu
```
```bash
neofetch
```
```bash
exit
```

---
# Performance
##### hdparm
lsblk -o NAME,MODEL

```bash
sudo pacman -S hdparm
```

```bash
sudo hdparm -B 255 /dev/sdb ; sudo hdparm -B 255 /dev/sdc ; sudo hdparm -S 0 /dev/sdb ; sudo hdparm -S 0 /dev/sdc
```
make a [udev rule](https://wiki.archlinux.org/title/Hdparm#Persistent_configuration_using_udev_rule "Hdparm") to apply them on boot-up

To create a udev rule to apply `hdparm` settings on boot-up, you can follow these steps. Udev rules are typically stored in the `/etc/udev/rules.d/` directory, and you can create a new file with the desired rule. Here's a step-by-step guide:

1. Open a terminal.
    
2. Create a new udev rule file using a text editor. For example, you can use the `nano` editor:
    

bash

`sudo nano /etc/udev/rules.d/99-hdparm.rules`

3. In the editor, add the following lines to set the `hdparm` settings:

bash

`ACTION=="add", KERNEL=="sdb", RUN+="/sbin/hdparm -B 255 -S 0 /dev/sdb" ACTION=="add", KERNEL=="sdc", RUN+="/sbin/hdparm -B 255 -S 0 /dev/sdc"`

These rules specify that when a device with the specified KERNEL (e.g., sdb or sdc) is added, the specified `hdparm` commands should be run.

4. Save the file. In `nano`, you can do this by pressing `Ctrl + X`, then `Y` to confirm, and finally `Enter` to exit.
    
5. Now, you need to reload the udev rules. You can do this with the following command:
    

bash

`sudo udevadm control --reload-rules`

6. Reboot your system or trigger a udev event for the changes to take effect:

bash

`sudo udevadm trigger`

The udev rule you created will be applied on boot-up, ensuring that the specified `hdparm` settings are set for the specified devices.
##### cpupower
```bash
sudo pacman -S cpupower
```

```bash
sudo cpupower frequency-set -g performance
```
```bash
sudo cpupower frequency-info
```

```bash
sudo cpupower frequency-set -g performance && echo 'cpupower frequency-set -g performance' | sudo tee -a /etc/rc.local
```

```bash
echo -e "[Unit]\nDescription=Set CPU governor to performance\n\n[Service]\nType=oneshot\nExecStart=/usr/bin/cpupower frequency-set -g performance\n\n[Install]\nWantedBy=multi-user.target" | sudo tee /etc/systemd/system/set-cpugovernor.service && sudo systemctl enable set-cpugovernor.service
```

```bash
echo '@reboot /usr/bin/cpupower frequency-set -g performance' | sudo crontab -
```

##### CPU Mitigations Off
```bash
sudo sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT="\(.*\)"/GRUB_CMDLINE_LINUX_DEFAULT="\1 mitigations=off"/' /etc/default/grub && sudo grub-mkconfig -o /boot/grub/grub.cfg
```

```bash
sudo efibootmgr --create --disk /dev/sda --part 1 --label "Arch Linux" --loader /vmlinuz-linux --unicode 'root=UUID=<Your-Root-Partition-UUID> rw mitigations=off' --verbose
```

#### Changing I/O scheduler

**Note:** The best choice of scheduler depends on both the device and the exact nature of the workload. Also, the throughput in MB/s is not the only measure of performance: deadline or fairness deteriorate the overall throughput but may improve system responsiveness. [Benchmarking](https://wiki.archlinux.org/title/Benchmarking "Benchmarking") may be useful to indicate each I/O scheduler performance.

To list the available schedulers for a device and the active scheduler (in brackets):

$ cat /sys/block/_**sda**_/queue/scheduler

mq-deadline kyber [bfq] none

To list the available schedulers for all devices:

$ grep "" /sys/block/*****/queue/scheduler

/sys/block/pktcdvd0/queue/scheduler:none
/sys/block/sda/queue/scheduler:mq-deadline kyber [bfq] none
/sys/block/sr0/queue/scheduler:[mq-deadline] kyber bfq none

To change the active I/O scheduler to _bfq_ for device _sda_, use:

# echo _**bfq**_ > /sys/block/_**sda**_/queue/scheduler

The process to change I/O scheduler, depending on whether the disk is rotating or not can be automated and persist across reboots. For example the [udev](https://wiki.archlinux.org/title/Udev "Udev") rules below set the scheduler to _bfq_ for rotational drives, _bfq_ for [SSD](https://wiki.archlinux.org/title/SSD "SSD")/eMMC drives and _none_ for [NVMe](https://wiki.archlinux.org/title/NVMe "NVMe") drives:

/etc/udev/rules.d/60-ioschedulers.rules

# HDD
ACTION=="add|change", KERNEL=="sd[a-z]*", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="bfq"

# SSD
ACTION=="add|change", KERNEL=="sd[a-z]*|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="bfq"

# NVMe SSD
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="none"

Reboot or force [udev#Loading new rules](https://wiki.archlinux.org/title/Udev#Loading_new_rules "Udev").

#### Tuning I/O scheduler

Each of the kernel's I/O scheduler has its own tunables, such as the latency time, the expiry time or the FIFO parameters. They are helpful in adjusting the algorithm to a particular combination of device and workload. This is typically to achieve a higher throughput or a lower latency for a given utilization. The tunables and their description can be found within the [kernel documentation](https://docs.kernel.org/block/index.html).

To list the available tunables for a device, in the example below _sdb_ which is using _deadline_, use:

$ ls /sys/block/_**sdb**_/queue/iosched

fifo_batch  front_merges  read_expire  write_expire  writes_starved

To improve _deadline'_s throughput at the cost of latency, one can increase `fifo_batch` with the command:

# echo _32_ > /sys/block/_**sdb**_/queue/iosched/**fifo_batch**
