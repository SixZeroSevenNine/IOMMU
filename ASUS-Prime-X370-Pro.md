# The Ultimate Linux Workstation?

My ASUS Maximus IV Gene-Z system was repurposed for pure file server purposes (FreeNAS) so I moved my kids VM to my gaming rig. I also needed another gaming rig so that 2.0 and 3.0 can play Minecraft with me.


**Goals:**
- Three Minecraft Java version capable rigs.
- A powerful gaming rig.
- A powerful Linux workstation.
- File serving capability as backup for FreeNAS rig.
- Has to fit in one case, one PC.


**Hardware:**
- ASUS Prime X370 Pro motherboard.
- Ryzen 7 1700X + 32 GB RAM.
- PCI x16 slot 1: Gigabyte Radeon RX 570 4GB for Windows gaming VM#1.
- PCI x16 slot 2: ASUS Strix GTX 970 for Windows gaming VM#2.
- PCI x16 slot 3: ASUS Radeon RX 550 for host GPU, used as workstation and as Minecraft play station.
- 256 GB Intel NVME SSD for KVM and workstation duties.
- 256 GB Samsung SSD for gaming VM#1.
- 256 GB Samsung SSD for gaming VM#2 OS.
- 512 GB Samsung SSD for gaming VM#2 Games / Steam library.
- 3 x 2 TB hard drive for backing up FreeNAS server.
- Seasonic Platinum 850W PSU.
- Be Quiet! Dark Base Pro 700 case.


**Preparation:**
1. Update the BIOS to latest version.
2. Enable virtualization in the BIOS: IOMMU and SVT.


**Installation:**
1. Install Ubuntu 18.10 Workstation. I added SSH and SMB in the installation options.
2. Edit `/etc/default/grub` and change the **GRUB_CMDLINE_LINUX_DEFAULT** entry to:
```
quiet amd_iommu=on iommu=pt kvm_amd.npt=1 rd.driver.pre=vfio-pci
```
3. Execute ```update-grub``` and reboot.
4. Gather IOMMU groups using this tiny script:
```
#!/bin/bash
for d in /sys/kernel/iommu_groups/*/devices/*; do
  n=${d#*/iommu_groups/*}; n=${n%%/*}
    printf 'IOMMU Group %s ' "$n"
    lspci -nns "${d##*/}"
done
```

Output should be something like this (If you don't see IOMMU Group ? ??:??:?? lines, then you forgot something in the previous steps):
```
IOMMU Group 0 00:01.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 10 00:08.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 11 00:08.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Internal PCIe GPP Bridge 0 to Bus B [1022:1454]
IOMMU Group 12 00:14.0 SMBus [0c05]: Advanced Micro Devices, Inc. [AMD] FCH SMBus Controller [1022:790b] (rev 59)
IOMMU Group 12 00:14.3 ISA bridge [0601]: Advanced Micro Devices, Inc. [AMD] FCH LPC Bridge [1022:790e] (rev 51)
IOMMU Group 13 00:18.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 0 [1022:1460]
IOMMU Group 13 00:18.1 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 1 [1022:1461]
IOMMU Group 13 00:18.2 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 2 [1022:1462]
IOMMU Group 13 00:18.3 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 3 [1022:1463]
IOMMU Group 13 00:18.4 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 4 [1022:1464]
IOMMU Group 13 00:18.5 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 5 [1022:1465]
IOMMU Group 13 00:18.6 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 6 [1022:1466]
IOMMU Group 13 00:18.7 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 7 [1022:1467]
IOMMU Group 14 01:00.0 Non-Volatile memory controller [0108]: Intel Corporation Device [8086:f1a5] (rev 03)
IOMMU Group 15 02:00.0 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Device [1022:43b9] (rev 02)
IOMMU Group 15 02:00.1 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] Device [1022:43b5] (rev 02)
IOMMU Group 15 02:00.2 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Device [1022:43b0] (rev 02)
IOMMU Group 15 03:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 300 Series Chipset PCIe Port [1022:43b4] (rev 02)
IOMMU Group 15 03:04.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 300 Series Chipset PCIe Port [1022:43b4] (rev 02)
IOMMU Group 15 03:06.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 300 Series Chipset PCIe Port [1022:43b4] (rev 02)
IOMMU Group 15 03:07.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 300 Series Chipset PCIe Port [1022:43b4] (rev 02)
IOMMU Group 15 04:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Lexa PRO [Radeon RX 550/550X] [1002:699f] (rev c7)
IOMMU Group 15 04:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Device [1002:aae0]
IOMMU Group 15 05:00.0 USB controller [0c03]: ASMedia Technology Inc. ASM1143 USB 3.1 Host Controller [1b21:1343]
IOMMU Group 15 06:00.0 Ethernet controller [0200]: Intel Corporation I211 Gigabit Network Connection [8086:1539] (rev 03)
IOMMU Group 16 08:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere [Radeon RX 470/480/570/570X/580/580X] [1002:67df] (rev ef)
IOMMU Group 16 08:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere [Radeon RX 580] [1002:aaf0]
IOMMU Group 17 09:00.0 VGA compatible controller [0300]: NVIDIA Corporation GM204 [GeForce GTX 970] [10de:13c2] (rev a1)
IOMMU Group 17 09:00.1 Audio device [0403]: NVIDIA Corporation GM204 High Definition Audio Controller [10de:0fbb] (rev a1)
IOMMU Group 18 0a:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Device [1022:145a]
IOMMU Group 19 0a:00.2 Encryption controller [1080]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Platform Security Processor [1022:1456]
IOMMU Group 1 00:01.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge [1022:1453]
IOMMU Group 20 0a:00.3 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) USB 3.0 Host Controller [1022:145c]
IOMMU Group 21 0b:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Device [1022:1455]
IOMMU Group 22 0b:00.2 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode] [1022:7901] (rev 51)
IOMMU Group 23 0b:00.3 Audio device [0403]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) HD Audio Controller [1022:1457]
IOMMU Group 2 00:01.3 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge [1022:1453]
IOMMU Group 3 00:02.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 4 00:03.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 5 00:03.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge [1022:1453]
IOMMU Group 6 00:03.2 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge [1022:1453]
IOMMU Group 7 00:04.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 8 00:07.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe Dummy Host Bridge [1022:1452]
IOMMU Group 9 00:07.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Internal PCIe GPP Bridge 0 to Bus B [1022:1454]
```
5. Identify the PCI device id's of the devices (GPU + HDMI Audio of the GPU) to be passed through the host, in my case the GPU in IOMMU group, ids: ```10de:13c2,10de:0fbb,1002:67df,1002:aaf0```. These devices will be reserved by vfio to prevent the host from using them.
6. Edit `/etc/initramfs-tools/modules` to have vfio linux driver latch on to the GPU that is passed through. Add these lines:
```
softdep nouveau pre: vfio vfio_pci
softdep amdgpu pre: vfio vfio_pci

vfio
vfio_iommu_type1
vfio_virqfd
options vfio_pci ids=10de:13c2,10de:0fbb,1002:67df,1002:aaf0
vfio_pci ids=10de:13c2,10de:0fbb,1002:67df,1002:aaf0
```
7. Edit `/etc/modules` to add these lines:
```
vfio
vfio_iommu_type1
vfio_pci ids=10de:13c2,10de:0fbb,1002:67df,1002:aaf0
```
8. Execute ```update-initramfs -u -k all``` and reboot.
9. Verify that vfio reserved the devices with ```lspci -vnn```. Find in the wall of text the GPU passed through (same for HDMI audio part), look out for the lines ```Kernel driver in use: vfio-pci```. Outpus should be something like:
```
09:00.0 VGA compatible controller [0300]: NVIDIA Corporation GM204 [GeForce GTX 970] [10de:13c2] (rev a1) (prog-if 00 [VGA controller])
	Subsystem: ASUSTeK Computer Inc. GM204 [GeForce GTX 970] [1043:8508]
	Flags: bus master, fast devsel, latency 0, IRQ 71
	Memory at f6000000 (32-bit, non-prefetchable) [size=16M]
	Memory at a0000000 (64-bit, prefetchable) [size=256M]
	Memory at b0000000 (64-bit, prefetchable) [size=32M]
	I/O ports at d000 [size=128]
	Expansion ROM at f7000000 [disabled] [size=512K]
	Capabilities: [60] Power Management version 3
	Capabilities: [68] MSI: Enable+ Count=1/1 Maskable- 64bit+
	Capabilities: [78] Express Legacy Endpoint, MSI 00
	Capabilities: [100] Virtual Channel
	Capabilities: [250] Latency Tolerance Reporting
	Capabilities: [258] L1 PM Substates
	Capabilities: [128] Power Budgeting <?>
	Capabilities: [600] Vendor Specific Information: ID=0001 Rev=1 Len=024 <?>
	Capabilities: [900] #19
	Kernel driver in use: vfio-pci
	Kernel modules: nvidiafb, nouveau
```
10. Install virtualization packages ```sudo apt-get install qemu-kvm libvirt-bin ovmf virt-manager```.
11. Depending on your needs and available storage, create the VM with corresponding storage sizes/types. I was lazy and used the virt-manager GUI to set up VMs, create storage and whatnot.
12. Set the firmware to OVMF.
13. Add the PCI devices to pass through to the VM. These are the ids following the IOMMU group number from steps 4 and 5.
14. Plug a screen on the GPU that was passed through and fire up VM. You should see the TianoCore boot splash screen on the monitor.
15. Install/Configure Windows.
16. ?????
17. Profit? Not yet. Dreaded code 43 after driver installation in the Windows VM using the GTX 970, NVidia doesn't want you to virtualize their gaming GPUs. Solution, ```virsh edit <VMName>``` and modify the XML. Important parts are  hidden state and vendor_id. They lie to the guest and trick the NVidia driver to load. 
```
<features>
    <acpi/>
    <apic/>
    <hyperv>
      <relaxed state='on'/>
      <vapic state='on'/>
      <spinlocks state='on' retries='8191'/>
      <vendor_id state='on' value='1234567890ab'/>
    </hyperv>
    <kvm>
      <hidden state='on'/>
    </kvm>
    <vmport state='off'/>
  </features>
```
18. Profit? Not yet, gmd3 doesn't fire up the host GPU (in last PCI x16 slot, shared with chipset). Remove Gmd3, replace it with LighDM. Then add GPU config in ```/usr/share/X11/xorg.conf.d``` by creating a file with (I called mine 99-hack.conf). This will let X11/gnome3/lightdm use the 3d GPU.
```
Section "Device"
    Identifier     "Device0"
    Driver         "amdgpu"
    VendorName     "AMD Corporation"
    BoardName      "AMD Tertiary"
    BusID          "PCI:4:0:0"
EndSection
```
18. Profit? Not yet, some Windows guests might not have HDMI audio out. Identify the Device Instance Path from the Details tab of the device in Device Manager.  Run regedit and find the same path under HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\  After following down the tree using the Device Instance Path information, continue to follow down through "Device Parameters" and "Interrupt Management".  Here you will find that you either have or need to create a new key named "MessageSignaledInterruptProperties".  Within that key, find or create a DWORD value named "MSISupported".  The value should be 1 enable MSI or 0 to disable. Enabling MSI should fix HDMI audio out issues.
19. Profit? Hell yea!


**Observations:**
1. BIOS updates can change IOMMU groupings, updates can mess up passthrough.
2. There is one single USB controller that is in its own IOMMU group, for other VMS, USB redirection works fine.
3. If MSI registry hack doesn't solve HDMI audio for one VM, the soundcard on motherboard can be passed through fine.
4. Using a monitor's USB hub for keyboard/mouse doesn't work after monitor goes into sleep and wakes up. Mouse is dead and keyboard events are sent to both host and guest.
5. VMs can be shut down and restarted without issues, didn't notice and PCI reset issues with the Gigabyte RX 570 nor the ASUS Strix GTX 970.
6. The ASUS Strix GTX 970 doesn't appreciate being in the first PCI x16 slot. I was unable to get it passed through even when blacklisting every possible NVidia driver using it. AMD GPUs don't seem to have these issues. Even though cooling is not optimal, I had no choice but to leave the NVidia GPU in slot 2.


**Future changes:**
1. Remove the NVME SSD, it's overkill and it's isolated in its own IOMMU group, and add a NVME to PCI x16 adapter to be able to have three fully passed through video cards. The Be Quiet! Dark Base Pro 700 allows one GPU to be mounted vertically and if picked correctly, the case can fit another GPU using the [adapter](https://www.aliexpress.com/item/Riser-PCIE-3-0-x16-To-M2-NGFF-NVMe-SSD-M-2-PCI-e-16x-Riser/32830793343.html?spm=2114.search0104.3.27.4fd972acoCIszM&ws_ab_test=searchweb0_0,searchweb201602_2_10065_10068_10130_10547_319_317_10548_10696_453_10084_454_10083_433_10618_431_10139_10307_537_536_10059_10884_10887_100031_321_322_10103,searchweb201603_51,ppcSwitch_0&algo_expid=7ef7a311-26f5-4c64-a18f-7943660471d2-4&algo_pvid=7ef7a311-26f5-4c64-a18f-7943660471d2&transAbTest=ae803_3).
2. Make it fully headless and a true server once #1 is implemented.
3. Upgrade the CPU to a 12 core Ryzen 3.


**References:**
- [VFIO tips and tricks](https://vfio.blogspot.com/2014/09/vfio-interrupts-and-how-to-coax-windows.html)
- [Mathias Hueber's awesome guide](http://mathiashueber.com/amd-ryzen-based-passthrough-setup-between-xubuntu-16-04-and-windows-10/)
- [Level1Techs' guide](https://level1techs.com/article/ryzen-gpu-passthrough-setup-guide-fedora-26-windows-gaming-linux)
- [Ubuntu Wiki](https://help.ubuntu.com/community/KVM)
- [Reddit user djgizmo](https://www.reddit.com/r/virtualization/comments/4bsnob/just_a_fyi_the_asus_maximus_iv_genez_supports_vtd/)
