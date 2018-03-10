# IOMMU on ASUS Maximus IV Gene-Z on Ubuntu 18.04

I repurposed my old gaming rig to be a Windows gaming VM for the kids and a Linux server at the same time.

**Preparation:**
1- Update the BIOS to latest version: 3603.

2- Enable virtualization in the BIOS (there is no IOMMU/VT-d option, this seems to include it).

3- Depending on your setup, might have to force the iGPU to be primary display.

4- Get a non K Sandy Bridge or Ivy Bridge CPU (got an i5-3470 to replace the 2500K).


**Linux:**
1- Install Ubuntu 18.04 LTS Server beta. I added SSH and SMB in the installation options.

2- Edit /etc/defaults/grub and change the **GRUB_CMDLINE_LINUX_DEFAULT** entry to:

```
quiet iommu=1 intel_iommu=on video=vesafb:off,efifb:off
```

