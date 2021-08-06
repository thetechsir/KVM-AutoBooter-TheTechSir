# KVM-AutoBooter-TheTechSir

![alt text](https://github.com/thetechsir/KVM-AutoBooter-TheTechSir/blob/main/color-banner.png)
![alt text](https://github.com/thetechsir/KVM-AutoBooter-TheTechSir/blob/main/kvm-autoboot.png)


boot any vfio guest right from your boot loader, I made this solution since I don't always want macOS &amp; Windows booting. until now I had three host os for the purpose of auto booting with just X11 on all opus, one for my Nvidia and win11 autobot KVM, and one for my AMD Mac KVM. This method will guide you to create systems service for each guest you want to be able to trigger auto booting from boot loader.


Cheat sheet, if you want to avoid my script, just reffer to these notes and keep in mind all you need is 3 things
1 - a script you want executed essentially only when you have the kernel command line variable (bootloader variable like mac=0)
2 - a systemd service that will execute that script when started. The service will be enabled all the time, only started with boot condition met
3 - your kernel command line option in a new boot entry with virtually any bootloader you're using. 

################################################    CHEAT SHEET    #####################################################################################

Systemd service creation

sudo nano /etc/systemd/system/kvmBoot-Win.service
####### in file 
[Unit]
Description=win-kvm-boot
ConditionKernelCommandLine=win=0
[Service]
#User=<user e.g. root>
WorkingDirectory=/
#ExecStart=/bin/true ./kvmBoot-Win.sh
Type=oneshot
TimeoutSec=0
RemainAfterExit=yes
ExecStart=/bin/bash -c 'cd / && ./kvmBoot-Win.sh'
[Install]
WantedBy=multi-user.target

sudo nano /etc/systemd/system/kvmBoot-mac.service
##### in file
[Unit]
Description=mac-kvm-boot
ConditionKernelCommandLine=mac=0
[Service]
#User=<user e.g. root>
WorkingDirectory=/
#ExecStart=/bin/true ./kvmBoot-mac.sh
Type=oneshot
TimeoutSec=0
RemainAfterExit=yes
ExecStart=/bin/bash -c 'cd / && ./kvmBoot-mac.sh'
[Install]
WantedBy=multi-user.target

### inside youre mac or win kvmBoot.sh
sudo virsh start {vmname} 


### win
menuentry "Nvidia VFIO Win11" {
     icon     /EFI/refind/os_win.png
    volume   "gtxArch"

    loader   /boot/vmlinuz-linux-vfio
    initrd   /boot/initramfs-linux-vfio.img
    options  "root=LABEL=gtxArch rw resume=/dev/nvme0n1p2 win=0 
video=efifb:off quiet text iommu=pt intel_iommu=on vt.global_cursor_default=0 loglevel=3 optimus-manager.startup=integrated 
systemd.unit=multi-user.target vfio-pci.ids=1462:7b98,8086:a305,1462:7b98,8086:a348,1462:9b98,8086:a323,1462:7b98
,8086:a3234,1462:7b98,8086:a324
,8086:15bc,14e4:43a0,106b:0117,1462:341b,1462:aaf0,1002:67df,1002:aaf0,10de:2182,10de:1aeb,10de:1aec,10de:1aed 
initrd=boot/initramfs-linux-vfio.img" 
  submenuentry "Boot macOS also" {
        add_options "mac=0 vfio-pci.ids=8086:a36d,1462:7b98,8086:a36f,1002:67df,1462:341b,1002:aaf0,1462:aaf0,14e4:43a0,106b:0117"
 
    }

    submenuentry "Boot to SDDM" {
        add_options "systemd.unit=graphical.target"
    }
    enabled
}


### gtxArch for Mac vfio
menuentry "VFIO Kernel MacOS" {
    icon     /EFI/refind/os_mac.png
    volume   "gtxArch"
    loader   /boot/vmlinuz-linux-vfio
    initrd   /boot/initramfs-linux-vfio.img
    options  "root=LABEL=gtxArch rw resume=/dev/nvme0n1p2 mac=0 
video=efifb:off quiet text iommu=pt intel_iommu=on vt.global_cursor_default=0 loglevel=3 optimus-manager.
startup=integrated systemd.unit=multi-user.target vfio-pci.ids=1462:7b98,8086:a305,1462:7b98,8086:a348,1462:9b98,
8086:a323,1462:7b98,8086:a3234,1462:7b98,8086:a324,8086:15bc,14e4:43a0,106b:0117,
1462:341b,1462:aaf0,1002:67df,1002:aaf0,8086:a36d,1462:7b98,8086:a36f,8086:7270 initrd=boot/initramfs-linux-vfio.img"
    submenuentry "Boot using fallback initramfs" {
        initrd /boot/initramfs-linux-fallback.img
    }
    submenuentry "Boot to sddm" {
        add_options "systemd.unit=graphical.target"
    }
    enabled
}
#######################################################    End of CHEET SHEET      #########################################################################


~Exact Contents of Script~

tput bold
echo  '-------------KVM Auto Boot Systemd Helper--------------'
tput setaf 6 
cat thetechsir.txt
echo
tput setaf 4
tput setaf 3
cat banner.txt
tput setaf 2
tput setaf 7
echo
tput setaf 2
echo  '-------------Select Your Option--------------'
tput setaf 4
echo  '-- 1 create your kvm virt guest start script'
tput setaf 4
echo '-- 2 create&enable systemd service with boot condition'
tput setaf 4
echo '-- 3 dump your manual.conf with new option added for refind'
tput setaf 2
echo  '-------------Select Your Option--------------'
echo  '-------------By' "$(tput setaf 5)TheTechSir" "$(tput setaf 7)--------------"


#select num in 1 2 3 4
#do 
#echo "you have chosen $num"
#if [[ -z "$num" ]]; then
#   printf '%s\n' "No input entered"
#done


read -ep "enter 1 to begin: " numSelect
if [ "$numSelect" == "1" ]
then
read -ep "VM Name Plz:  " vmName
sudo mkdir /etc/libvirt/auto-kvm/

echo "sudo virsh start ${vmName}" > "/tmp/${vmName}.sh"
sudo chmod +x "/tmp/${vmName}.sh"
sudo cp "/tmp/${vmName}.sh"  "/etc/libvirt/auto-kvm/${vmName}.sh"
echo start script was created here /etc/libvirt/auto-kvm/${vmName}.sh
vmstart=/etc/libvirt/auto-kvm/${vmName}.sh
fi

echo "Create systemd service for this kvm 2"

vmstart=/etc/libvirt/auto-kvm/${vmName}.sh
read -ep "Service Name Plz:  " serviceName
echo $serviceName

touch /tmp/${serviceName}.service

cat <<EOT >> /tmp/${serviceName}.service

[Unit]
Description=${serviceName}-kvm-boot
ConditionKernelCommandLine=${vmName}=0
[Service]
WorkingDirectory=/
Type=oneshot
TimeoutSec=0
RemainAfterExit=yes
ExecStart=/bin/bash -c 'cd / && .${vmstart}'
[Install]
WantedBy=multi-user.target
EOT
sudo mv /tmp/${serviceName}.service /etc/systemd/system/${serviceName}.service
tput setaf 2
cat /etc/systemd/system/${serviceName}.service
sudo systemctl enable $serviceName
echo 
echo 
tput setaf 7
echo now simply add ${vmName}=0 to your bootloader, guest will only boot when 
echo your bootloader has this line in it. 
echo enjoy!
echo 
echo
tput setaf 4
sudo systemctl status $serviceName







![alt text](https://github.com/thetechsir/KVM-AutoBooter-TheTechSir/blob/main/1.png)
![alt text](https://github.com/thetechsir/KVM-AutoBooter-TheTechSir/blob/main/2.png)
