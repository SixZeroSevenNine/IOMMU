# IOMMU on ASUS Maximus IV Gene-Z on Ubuntu 18.04


I repurposed my old gaming rig to be a Windows gaming VM for the kids and a Linux server at the same time.


**Hardware:**
- ASUS Maximus IV Gene-Z motherboard.
- Intel i5-3470 CPU (1 core for server, 3 for guest).
- ASUS Radeon RX 550 for host GPU.
- 20 GB Intel SLC SSD for Linux host.
- 120 GB Intel MLC SSD for Windows guest.
- 16 GB of RAM, 10 allocated to guest.
- 4 x 1 TB hard drive for NAS role.


**Preparation:**
1. Update the BIOS to latest version: 3603.
2. Enable virtualization in the BIOS (there is no IOMMU/VT-d option, this seems to include it).
3. Depending on your setup, might have to force the iGPU to be primary display unless you pass through the GPU in second PCIex 16 slot.
4. Get a non K Sandy Bridge or Ivy Bridge CPU (got an i5-3470 to replace the 2500K).

**Linux:**
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
6. Identify the PCI device id's of the devices to be passed through the host, in my case the GPU (group 1, ids: ```1002:699f, 1002:aae0```. These devices will be reserved by vfio to prevent the host from using them.
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
