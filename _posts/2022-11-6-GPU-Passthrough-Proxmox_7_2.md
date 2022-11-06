---
layout: post
title: GPU Passthrough in Proxmox 7.2!
---

This is less of a blog post and more of a personal note, which I hope could help someone else out there as well. Keeping it shot and to-the-point in the hopes that this is easily searchable.

If you find any issues with the following, feel free to raise PR against this post.

Some of the specifications

```
Proxmox version: 7.2-11
Kernel version: 5.15.19-2-pve
GPU: nvidia T600 4GB (PNY VCNT600-SB)
```

Now, let's get started.

# GPU Passthrough in Proxmox 7.2-11

## 1. Passthrough modules to be loaded
> This will ensure that these modules are loaded, as these will help with the passthrough

Within `/etc/modules`, have these:
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

## 2. Blacklist the existing drivers
> This will blacklist these drivers from being loaded.

Within `/etc/modprobe.d/blacklist.cfg`.

```
blacklist nouveau
blacklist nvidia
blacklist nvidiafb
```

## 3. Allow unsafe interrupts (needed sometimes)

Within `/etc/modprobe.d/iommu_unsafe_interrupts.conf` 
```
options vfio_iommu_type1 allow_unsafe_interrupts=1
```

## 4. Add you GPU to the VFIO module config

### 4.1 Now find IDs for your GPU
- Find the main ID using `lspci`

```
09:00.0 VGA compatible controller: NVIDIA Corporation Device 1fb1 (rev a1)
^^^^^^^ This is the Main GPU
	Subsystem: NVIDIA Corporation Device 1488
	Kernel driver in use: vfio-pci
	Kernel modules: nvidiafb, nouveau
09:00.1 Audio device: NVIDIA Corporation Device 10fa (rev a1)
^^^^^^^ This is HDMI Audio
	Subsystem: NVIDIA Corporation Device 1488
	Kernel driver in use: vfio-pci
	Kernel modules: snd_hda_intel
```
- Then list device-id and vendor-id using `lspci -nk`

```
09:00.0 0300: 10de:1fb1 (rev a1)
              ^^^^^^^^^
	Subsystem: 10de:1488
	Kernel driver in use: vfio-pci
	Kernel modules: nvidiafb, nouveau
09:00.1 0403: 10de:10fa (rev a1)
              ^^^^^^^^^
	Subsystem: 10de:1488
	Kernel driver in use: vfio-pci
	Kernel modules: snd_hda_intel
```
### 4.2 Add the device ID and vendor ID within vfio module's config
- Within `/etc/modprobe.d/vfio.conf`, put these IDs
```
options vfio-pci ids=10de:1fb1,10de:10fa disable_vga=1
```

## 6. Set some more magic flags
[More details on Ignore MSRs flag](https://patchwork.kernel.org/project/kvm/patch/1250686963-8357-38-git-send-email-avi@redhat.com/)

Within `/etc/modprobe.d/kvm.conf` 
```
options kvm ignore_msrs=1
```

## 7. Disable GPU framebuffers during boot

Within `/etc/default/grub`, ensure that we have

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on nomodeset video=vesafb:off video=efifb:off video=simplefb:off"
```

Update GRUB after updating the file above

```
update-grub && update-initramfs -u
```

## 8. Reboot
```
shutdown -r 0
```

## 8. Normally attach the GPU to your choice of VM

1. Open Proxmox web UI
2. Open "Hardware" configuration of your VM where you want to attach the GPU
3. Click on "Add" on the top left, and select the GPU you want to pass through (the same one as the one in Step 4)
![Adding GPU](/images/20221106-add-pci-device.png)