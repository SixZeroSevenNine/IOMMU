# IOMMU with ASUS Maximus IV Gene-Z on Ubuntu 18.04


I repurposed my old gaming rig to be a Windows gaming VM for the kids and a Linux server at the same time.


**Hardware:**
- ASUS Maximus IV Gene-Z motherboard (Sandy Bridge and Ivy Bridge era).
- Intel i5-3470 CPU (1 core for server, 3 for guest).
- ASUS Radeon RX 550 for host GPU.
- 20 GB Intel SLC SSD for Linux host.
- 120 GB Intel MLC SSD for Windows guest.
- 16 GB of RAM, 10 allocated to guest.
- 4 x 1 TB hard drive for NAS role (mdadm, RAID6).


**Preparation:**
1. Update the BIOS to latest version: 3603.
2. Enable virtualization in the BIOS (there is no IOMMU/VT-d option, this seems to include it).
3. Force the iGPU to be primary display.
4. Get a non K Sandy Bridge or Ivy Bridge CPU (got an i5-3470 to replace the i5-2500K).


**Installation:**
1. Install Ubuntu 18.04 LTS Server beta. I added SSH and SMB in the installation options.
2. Edit `/etc/defaults/grub` and change the **GRUB_CMDLINE_LINUX_DEFAULT** entry to:
```
quiet iommu=1 intel_iommu=on video=vesafb:off,efifb:off
```
3. Reboot.
4. Using this tiny script, get the IOMMU groups:
5. Gather IOMMU groups using this tiny script:
```
#!/bin/bash
for d in /sys/kernel/iommu_groups/*/devices/*; do
  n=${d#*/iommu_groups/*}; n=${n%%/*}
    printf 'IOMMU Group %s ' "$n"
    lspci -nns "${d##*/}"
done
```

Output should be something like this (If you don't see IOMMU Group ? ??:??:?? lines, then you got configuration issues in previous steps):
```
IOMMU Group 0 00:00.0 Host bridge [0600]: Intel Corporation Xeon E3-1200 v2/3rd Gen Core processor DRAM Controller [8086:0150] (rev 09)
IOMMU Group 10 00:1c.6 PCI bridge [0604]: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 7 [8086:1c1c] (rev b5)
IOMMU Group 11 00:1d.0 USB controller [0c03]: Intel Corporation 6 Series/C200 Series Chipset Family USB Enhanced Host Controller #1 [8086:1c26] (rev 05)
IOMMU Group 12 00:1f.0 ISA bridge [0601]: Intel Corporation Z68 Express Chipset Family LPC Controller [8086:1c44] (rev 05)
IOMMU Group 12 00:1f.2 SATA controller [0106]: Intel Corporation 6 Series/C200 Series Chipset Family SATA AHCI Controller [8086:1c02] (rev 05)
IOMMU Group 12 00:1f.3 SMBus [0c05]: Intel Corporation 6 Series/C200 Series Chipset Family SMBus Controller [8086:1c22] (rev 05)
IOMMU Group 13 03:00.0 USB controller [0c03]: ASMedia Technology Inc. ASM1042 SuperSpeed USB Host Controller [1b21:1042]
IOMMU Group 14 04:00.0 USB controller [0c03]: ASMedia Technology Inc. ASM1042 SuperSpeed USB Host Controller [1b21:1042]
IOMMU Group 15 05:00.0 SATA controller [0106]: JMicron Technology Corp. JMB362 SATA Controller [197b:2362] (rev 10)
IOMMU Group 1 00:01.0 PCI bridge [0604]: Intel Corporation Xeon E3-1200 v2/3rd Gen Core processor PCI Express Root Port [8086:0151] (rev 09)
IOMMU Group 1 01:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Polaris12 [1002:699f] (rev c7)
IOMMU Group 1 01:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Device [1002:aae0]
IOMMU Group 2 00:02.0 VGA compatible controller [0300]: Intel Corporation Xeon E3-1200 v2/3rd Gen Core processor Graphics Controller [8086:0152] (rev 09)
IOMMU Group 3 00:16.0 Communication controller [0780]: Intel Corporation 6 Series/C200 Series Chipset Family MEI Controller #1 [8086:1c3a] (rev 04)
IOMMU Group 4 00:19.0 Ethernet controller [0200]: Intel Corporation 82579V Gigabit Network Connection [8086:1503] (rev 05)
IOMMU Group 5 00:1a.0 USB controller [0c03]: Intel Corporation 6 Series/C200 Series Chipset Family USB Enhanced Host Controller #2 [8086:1c2d] (rev 05)
IOMMU Group 6 00:1b.0 Audio device [0403]: Intel Corporation 6 Series/C200 Series Chipset Family High Definition Audio Controller [8086:1c20] (rev 05)
IOMMU Group 7 00:1c.0 PCI bridge [0604]: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 1 [8086:1c10] (rev b5)
IOMMU Group 8 00:1c.4 PCI bridge [0604]: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 5 [8086:1c18] (rev b5)
IOMMU Group 9 00:1c.5 PCI bridge [0604]: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 6 [8086:1c1a] (rev b5)
```
6. Identify the PCI device id's of the devices (GPU + HDMI Audio of the GPU) to be passed through the host, in my case the GPU in IOMMU group, ids: ```1002:699f, 1002:aae0```. These devices will be reserved by vfio to prevent the host from using them.
7. Edit `/etc/initramfs-tools/modules` to have vfio linux driver latch on to the GPU that is passed through. Add these lines:
```
softdep amdgpu pre: vfio vfio_pci

vfio
vfio_iommu_type1
vfio_virqfd
options vfio_pci ids=1002:699f,1002:aae0
vfio_pci ids=1002:699f,1002:aae0
```
8. Edit `/etc/modules` to add these lines:
```
vfio
vfio_iommu_type1
vfio_pci ids=1002:699f,1002:aae0
```
9. Reboot.
10. Verify that vfio reserved the devices with ```lspci -vnn```. Find in the wall of text the GPU passed through (same for HDMI audio part), look out for the lines ```Kernel driver in use: vfio-pci```. Outpus should be something like:
```
01:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Polaris12 [1002:699f] (rev c7) (prog-if 00 [VGA controller])
        Subsystem: ASUSTeK Computer Inc. Lexa PRO [Radeon RX 550] [1043:0511]
        Flags: bus master, fast devsel, latency 0, IRQ 41
        Memory at e0000000 (64-bit, prefetchable) [size=256M]
        Memory at f0000000 (64-bit, prefetchable) [size=2M]
        I/O ports at e000 [size=256]
        Memory at f7b00000 (32-bit, non-prefetchable) [size=256K]
        Expansion ROM at f7b40000 [disabled] [size=128K]
        Capabilities: [48] Vendor Specific Information: Len=08 <?>
        Capabilities: [50] Power Management version 3
        Capabilities: [58] Express Legacy Endpoint, MSI 00
        Capabilities: [a0] MSI: Enable+ Count=1/1 Maskable- 64bit+
        Capabilities: [100] Vendor Specific Information: ID=0001 Rev=1 Len=010 <?>
        Capabilities: [150] Advanced Error Reporting
        Capabilities: [200] #15
        Capabilities: [270] #19
        Capabilities: [2b0] Address Translation Service (ATS)
        Capabilities: [2c0] Page Request Interface (PRI)
        Capabilities: [2d0] Process Address Space ID (PASID)
        Capabilities: [320] Latency Tolerance Reporting
        Capabilities: [328] Alternative Routing-ID Interpretation (ARI)
        Capabilities: [370] L1 PM Substates
        Kernel driver in use: vfio-pci
        Kernel modules: amdgpu
```
11. Install virtualization packages ```sudo apt-get install qemu-kvm libvirt-bin ovmf```.
12. Depending on your needs/storage create the VM. I was lazy and used the virt-manager GUI from my Ubuntu workstation.
13. Set the firmware to OVMF.
14. Add the PCI devices to pass through to the VM. These are the ids following the IOMMU group number from step 5, in my case GPU: ```01:00.0``` and GPU Audio: ```01:00.1```. For some reasons, there is a PCI Bridge in the same IOMMU group that can't be passed through and KVM isn't complaining about it. I also passed through the Intel USB controller in Group 5, id: ```00:1a.0```, this is the row of USB ports next to the USB3/eSata ports on the io panel of the motherboard.
15. Plug a screen on the GPU passed through and fire up VM. You should see the TianoCore boot splash screen on the monitor.
16. Install/Configure Windows.
16. ?????
17. Profit!


**Observations:***
1. BIOS doesn't mention a word about IOMMU/VT-d, yet enabling Virtualization in the CPU tab seems to enable it.
2. Putting the GPU in the second PCIex 16 slot for passthrough will need ACS. The GPU ends up grouped with a bunch of chipset devices.
3. ASMedia controllers can be passed through to give the VM a USB3 controller.


**References:**
- [Mathias Hueber](http://mathiashueber.com/amd-ryzen-based-passthrough-setup-between-xubuntu-16-04-and-windows-10/)
- [Level1Techs](https://level1techs.com/article/ryzen-gpu-passthrough-setup-guide-fedora-26-windows-gaming-linux)
- [Ubuntu Wiki](https://help.ubuntu.com/community/KVM)
- [Reddit user djgizmo](https://www.reddit.com/r/virtualization/comments/4bsnob/just_a_fyi_the_asus_maximus_iv_genez_supports_vtd/)
