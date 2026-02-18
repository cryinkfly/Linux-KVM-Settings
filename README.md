<div id="header" align="center">
<img align="center" src="https://github.com/user-attachments/assets/b2189ef6-26bb-453c-a394-5b23d438e617">
<h1>[GUIDE] Tips, Tricks & Tutorials for getting fully up and running KVM (Kernel-based Virtual Machine) across various Linux distributions</h1>
<br>
<img align="center" src="https://img.shields.io/github/license/cryinkfly/Linux-KVM?style=flat">
<img align="center" src="https://img.shields.io/github/last-commit/cryinkfly/Linux-KVM?style=flat">
<img align="center" src="https://img.shields.io/github/issues-raw/cryinkfly/Linux-KVM?style=flat"> 
<img align="center" src="https://img.shields.io/github/stars/cryinkfly/Linux-KVM?style=flat"> 
<img align="center" src="https://img.shields.io/github/forks/cryinkfly/Linux-KVM?style=flat"> 
</div>

<h2>📜 Description</h2>
<p>This project involves providing detailed information on the installation and configuration of KVM (Kernel-based Virtual Machine) across various Linux distributions. KVM is a popular open-source virtualization solution integrated into the Linux kernel, enabling users to run multiple virtual machines on a single host. The project covers key steps for setting up KVM on different Linux distros, including dependencies, installation of necessary packages, enabling virtualization support, configuring network bridges, and managing virtual machines through tools like libvirt, virt-manager, or command-line utilities. Each distro's specific nuances and commands are highlighted for a seamless setup experience.</p>

---

## 1.) Show PCI identification number and [Vendor-ID:Device-ID] of the graphics card and USB controller:

    lspci -nn | grep -i amd #All AMD graphics cards are displayed!

    lspci -nn | grep -i nvidia #All NVIDIA graphics cards are displayed!

    lspci -nn | grep -i usb #All USB devices (controllers) are displayed!

For example my GPU and PCI-USB controller:

    12:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 24 [Radeon PRO W6400] [1002:7422]
    12:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 21/23 HDMI/DP Audio Controller [1002:ab28]
    06:00.0 USB controller [0c03]: ASMedia Technology Inc. ASM2142/ASM3142 USB 3.1 Host Controller [1b21:2142]

And here the correct vfio-pci ids:

    # vfio-pci ids=<someID,someOtherID>
    vfio-pci ids=1002:7422,1002:ab28,1b21:2142

---

## 2.) Installation & Configuration of GRUB/Init/Modules

### Debian

    su -
    apt install bridge-utils guestfs-tools libguestfs-tools libvirt-clients libvirt-daemon ovmf qemu-system-x86 qemu-utils virt-manager virt-viewer virtinst

    nano /etc/default/grub
    
    # Enable the IOMMU feature and the [vfio-pci] kernel module on the KVM host (line 6).
    GRUB_CMDLINE_LINUX="rhgb quiet amd_iommu=on iommu=pt rd.driver.pre=vfio-pci video=efifb:off kvm.ignore_msrs=1 kvm.report_ignored_msrs=0" # For Intel CPU: intel_iommu=on

    update-grub # Or: grub-mkconfig -o /boot/grub/grub.cfg
    
    nano /etc/initramfs-tools/modules

    vfio
    vfio_iommu_type1
    vfio_pci
    vfio_virqfd
    options vfio-pci ids=1002:7422,1002:ab28,1b21:2142

    nano /etc/modprobe.d/vfio.conf
    
    options vfio-pci ids=1002:7422,1002:ab28,1b21:2142
    softdep amdgpu pre: vfio-pci
    softdep xhci_pci pre: vfio_pci
    #softdep radeon pre: vfio-pci
    #softdep snd_hda_intel pre: vfio-pci
    #softdep nouveau pre: vfio-pci
    #softdep nvidia pre: vfio-pci
    #softdep nvidia* pre: vfio-pci
    #softdep drm pre: vfio-pci
    #softdep xhci_hdc pre: vfio-pci
    #options kvm_amd avic=1

    update-initramfs -k all -u

    systemctl enable libvirtd
    usermod -aG libvirt steve #For example username: steve

    sudo reboot

---

### Ubuntu

    sudo apt update
    sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager

    sudo nano /etc/default/grub
    
    # Enable the IOMMU feature and the [vfio-pci] kernel module on the KVM host (line 6).
    GRUB_CMDLINE_LINUX="rhgb quiet amd_iommu=on iommu=pt rd.driver.pre=vfio-pci video=efifb:off kvm.ignore_msrs=1 kvm.report_ignored_msrs=0" # For Intel CPU: intel_iommu=on

    update-grub # Or: grub-mkconfig -o /boot/grub/grub.cfg

    sudo nano /etc/initram-fs/modules
    
    vfio
    vfio_iommu_type1
    vfio_pci
    vfio_virqfd
    options vfio-pci ids=1002:7422,1002:ab28,1b21:2142

    nano /etc/modprobe.d/vfio.conf
    
    options vfio-pci ids=1002:7422,1002:ab28,1b21:2142
    softdep amdgpu pre: vfio-pci
    softdep xhci_pci pre: vfio_pci
    #softdep radeon pre: vfio-pci
    #softdep snd_hda_intel pre: vfio-pci
    #softdep nouveau pre: vfio-pci
    #softdep nvidia pre: vfio-pci
    #softdep nvidia* pre: vfio-pci
    #softdep drm pre: vfio-pci
    #softdep xhci_hdc pre: vfio-pci
    #options kvm_amd avic=1

    sudo update-initramfs -k all -u

    sudo systemctl enable libvirtd
    sudo usermod -aG libvirt $(whoami)

    sudo reboot

---

### Fedora

#### Fedora Workstation

    sudo dnf install virt-install libvirt-daemon-config-network libvirt-daemon-kvm qemu-kvm virt-manager virt-viewer guestfs-tools libguestfs-tools virt-top libvirt-devel bridge-utils edk2-ovmf
    
    sudo nano /etc/default/grub

    # Enable the IOMMU feature and the [vfio-pci] kernel module on the KVM host (line 6).
    GRUB_CMDLINE_LINUX="rhgb quiet amd_iommu=on iommu=pt rd.driver.pre=vfio-pci video=efifb:off kvm.ignore_msrs=1 kvm.report_ignored_msrs=0" # For Intel CPU: intel_iommu=on
    
    sudo grub2-mkconfig -o /etc/grub2-efi.cfg

    su -c 'echo "options vfio-pci ids=1002:7422,1002:ab28,1b21:2142" > /etc/modprobe.d/vfio.conf && echo "vfio-pci" > /etc/modules-load.d/vfio-pci.conf'

    sudo nano /etc/dracut.conf.d/gpu-passthrough.conf

    add_drivers+=" pci_stub vfio vfio_iommu_type1 vfio_pci kvm kvm_amd "

    sudo dracut -f --kver $(uname -r)

    sudo systemctl enable libvirtd
    sudo usermod -aG libvirt $(whoami)

    sudo reboot

#### Fedora Atomics (Silverblue, ...)

    rpm-ostree install virt-install libvirt-daemon-config-network libvirt-daemon-kvm qemu-kvm virt-manager virt-viewer guestfs-tools libguestfs-tools virt-top bridge-utils edk2-ovmf
    
    sudo rpm-ostree kargs --append-if-missing="rhgb" --append-if-missing="amd_iommu=on" --append-if-missing="iommu=pt" --append-if-missing="video=efifb:off" --append-if-missing="rd.driver.pre=vfio_pci" --append-if-missing="kvm.ignore_msrs=1" --append-if-missing="kvm.report_ignored_msrs=0"  --reboot # For Intel CPU: intel_iommu=on

    sudo rpm-ostree kargs \
      --append-if-missing="vfio-pci.ids=1002:7422,1002:ab28,1b21:2142" \
      --reboot

    sudo rpm-ostree initramfs \
      --enable \
      --arg="--add-drivers" \
      --arg="vfio vfio_iommu_type1 vfio_pci kvm kvm_amd" \
      --reboot

    sudo systemctl enable --now libvirtd
    grep -E '^libvirt:' /usr/lib/group | sudo tee -a /etc/group
    sudo usermod -aG libvirt $USER
    sudo reboot

---

### openSUSE

#### openSUSE Leap & Tumbleewed

    sudo zypper install -t pattern kvm_server kvm_tools
    sudo zypper install libvirt libvirt-client libvirt-daemon virt-manager virt-install virt-viewer qemu qemu-kvm qemu-ovmf-x86_64 qemu-tools

    su -c 'nano /etc/default/grub'

    # Enable the IOMMU feature and the [vfio-pci] kernel module on the KVM host (line 6).
    GRUB_CMDLINE_LINUX="rhgb quiet amd_iommu=on iommu=pt rd.driver.pre=vfio-pci video=efifb:off kvm.ignore_msrs=1 kvm.report_ignored_msrs=0" # For Intel CPU: intel_iommu=on

    sudo grub2-mkconfig -o /boot/grub2/grub.cfg

    # Two files (/etc/modprobe.d/vfio.conf &/etc/modules-load.d/vfio-pci.conf) must be created and your device-specific numbers must be entered there:
    su -c 'echo "options vfio-pci ids=1002:7422,1002:ab28,1b21:2142" > /etc/modprobe.d/vfio.conf && echo "vfio-pci" > /etc/modules-load.d/vfio-pci.conf'

    # You need to rebuild the initial ram disk to include all the needed modules. Create a file named /etc/dracut.conf.d/gpu-passthrough.conf:
    su -c 'nano /etc/dracut.conf.d/gpu-passthrough.conf'
    add_drivers+="pci_stub vfio vfio_iommu_type1 vfio_pci vfio_virqfd kvm kvm_amd" # For Intel CPU: kvm_intel

    sudo mkinitrd

    sudo systemctl enable libvirtd
    sudo usermod -aG libvirt $(whoami)

    sudo reboot

#### openSUSE Micro OS - Aeon, Kalpa & Baldur

    sudo transactional-update pkg install -t pattern kvm_server kvm_tools
    sudo transactional-update -c pkg install libvirt libvirt-client libvirt-daemon virt-manager virt-install virt-viewer qemu qemu-kvm qemu-ovmf-x86_64 qemu-tools

    sudo reboot

    su -c 'nano /etc/default/grub'

    # Enable the IOMMU feature and the [vfio-pci] kernel module on the KVM host (line 6).
    GRUB_CMDLINE_LINUX="rhgb quiet amd_iommu=on iommu=pt rd.driver.pre=vfio-pci video=efifb:off kvm.ignore_msrs=1 kvm.report_ignored_msrs=0" # For Intel CPU: intel_iommu=on

    sudo transactional-update grub.cfg

    # Two files (/etc/modprobe.d/vfio.conf &/etc/modules-load.d/vfio-pci.conf) must be created and your device-specific numbers must be entered there:
    su -c 'echo "options vfio-pci ids=1002:7422,1002:ab28,1b21:2142" > /etc/modprobe.d/vfio.conf && echo "vfio-pci" > /etc/modules-load.d/vfio-pci.conf'

    # You need to rebuild the initial ram disk to include all the needed modules. Create a file named /etc/dracut.conf.d/gpu-passthrough.conf:
    su -c 'nano /etc/dracut.conf.d/gpu-passthrough.conf'
    add_drivers+="pci_stub vfio vfio_iommu_type1 vfio_pci vfio_virqfd kvm kvm_amd" # For Intel CPU: kvm_intel
    
    sudo transactional-update -c initrd

    sudo reboot

    sudo systemctl enable libvirtd
    sudo usermod -aG libvirt $(whoami)

    sudo reboot


---

#### More Linux Distro instructions will follow here...

---

### Optional for all Linux Distro's - Change the default directory/drive for the virtual machines (guests)

In order to be able to change the default storage location of KVM Libvirt, you should also change this file (/etc/libvirt/qemu.conf):

![222960741-8770a034-e1e1-40b9-bd70-6e052f67b053](https://github.com/user-attachments/assets/c0504bdd-6138-4c1a-99a3-729c0c9e1f7d)

Further information can be found here:

- https://ostechnix.com/how-to-change-kvm-libvirt-default-storage-pool-location/
- https://ostechnix.com/solved-cannot-access-storage-file-permission-denied-error-in-kvm-libvirt/

---

## 3.) Check & Verify VFIO control over PCI Devices

After your machien reboots, run lspci -nnk to show which kernel driver has control over each PCI device. All devices should show vfio-pci as the kernel driver in use. If not, you will need to repeat the previous steps to disable that driver.

    lspci -nnk

---

## 4.) Configure Virt-Manager (Example)

I have already published a [video on my YouTube channel](https://www.youtube.com/live/6u-ZKKVg9-A?feature=shared&t=10884) where I showed how, for example, you can pass a graphics card and a PCI USB card to the guest.

---

## 5.) Troubleshootings

### KVM cannot start the "default" network in MicroOS!

If you get this failure message by running the "default" network:

    sudo virsh net-start default
    error: Failed to start network default
    error: internal error: Failed to apply firewall rules /sbin/iptables -w --table filter --list-rules:
    libvirt: error : cannot execute binary /sbin/iptables: Permission denied

You can temporarily bypass this issue with these commands:

    sudo setenforce 0
    sudo systemctl stop firewalld # If installed & configured firewalld
    sudo systemctl restart libvirtd # If installed & configured firewalld
    sudo systemctl start firewalld # If installed & configured firewalld
    sudo virsh net-start default

---

*Note 1: The "video=efifb:off" option should only be added if your system is configured to automatically load the graphical environment! If you want to switch to the graphical environment via the terminal after booting, you may no longer see the terminal.*

*Note 2: In addition, the option causes problems with some NVIDIA graphics cards!*

*Note 3: Basically, the "amd_iommu=on" or "intel_iommu=on" option would also suffice, but you get better performance in the guest VM with the "iommu=pt" option and with the "video=efifb:off" option will prevent the driver from stealing the GPU.*

*Note 4: The kernel parameter "rd.driver.pre=vfio_pci" ensures that the vfio_pci module is loaded early, during the initramfs stage of the boot process. It is commonly used for binding PCI devices to the vfio-pci driver, which allows them to be passed through to virtual machines.*

*Note 5: The kernel parameters "kvm.ignore_msrs=1" and "kvm.report_ignored_msrs=0" solved some issues with running Windows 10 & 11 guests!*
