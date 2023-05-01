## A.1 Introduction

This article describes how to assign an NVIDIA GPU graphics card on the host machine to a virtualized guest.

## A.2 Prerequisites

-   GPU pass-through is supported on the AMD64/Intel 64 architecture only.

-   The host operating system needs to be SLES 12 SP3 or newer.
  
-   This article deals with a set of instructions based on V100/T1000 NVIDIA cards, and is meant for GPU computation purposes only.
   
-   Verify that you are using an NVIDIA Tesla product—Maxwell, Pascal, or Volta.
   
-   To be able to manage the host system, you need an additional display card on the host that you can use when configuring the GPU Pass-Through, or a functional SSH environment.   

## A.3 Configuring the host

### A.3.1 Verify the host environment

1.  Verify that the host operating system is SLES 12 SP3 or newer:

    ```
    > cat /etc/issue
    ```

2.  Verify that the host supports [VT-d](https://doc.opensuse.org/documentation/leap/virtualization/html/book-virtualization/gloss-vt-glossary.html#gloss-vt-acronym-vtd "VT-d") or SVM technology and that it is already enabled in the BIOS firmware settings:
  
    ```
    > dmesg | grep -e DMAR -e IOMMU
    ```

    *If VT-d is not enabled in the firmware, enable it and reboot the host.*

3.  Verify that the host has an extra GPU or VGA card:
    
    ```
    > lspci | grep -e "vga|VGA|nvidia|NVIDIA|amd|AMD"
    ```

### A.3.2 Enable [IOMMU](https://doc.opensuse.org/documentation/leap/virtualization/html/book-virtualization/gloss-vt-glossary.html#gloss-vt-acronym-iommu "IOMMU")

[IOMMU](https://doc.opensuse.org/documentation/leap/virtualization/html/book-virtualization/gloss-vt-glossary.html#gloss-vt-acronym-iommu "IOMMU") is disabled by default. You need to enable it at boot time in the `/etc/default/grub` configuration file.

1.  For Intel-based hosts:
    
    GRUB_CMDLINE_LINUX="intel_iommu=on iommu=pt rd.driver.pre=vfio-pci"
    
    For AMD-based hosts:
    
    GRUB_CMDLINE_LINUX="iommu=pt amd_iommu=on rd.driver.pre=vfio-pci"

2.  When you save the modified `/etc/default/grub` file, re-generate the main GRUB 2 configuration file `/boot/grub2/grub.cfg`:

    ```
    > sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

3.  Reboot the host and verify that [IOMMU](https://doc.opensuse.org/documentation/leap/virtualization/html/book-virtualization/gloss-vt-glossary.html#gloss-vt-acronym-iommu "IOMMU") is enabled:

    ```
    > dmesg | grep -e DMAR -e IOMMU
    ```


### A.3.3 Blacklist the Nouveau driver

To assign the NVIDIA card to a VM guest, we need to prevent the host OS from loading the built-in `nouveau` driver for NVIDIA GPUs. Create the file `/etc/modprobe.d/60-blacklist-nouveau.conf` with the following content:

```
blacklist nouveau
```

### A.3.4 Configure [VFIO](https://doc.opensuse.org/documentation/leap/virtualization/html/book-virtualization/gloss-vt-glossary.html#gloss-vt-acronym-vfio "VFIO") and isolate the GPU used for pass-through

1.  Find the card vendor and model IDs. Utilize the bus number identified in [Section A.3.1, “Verify the host environment”](https://doc.opensuse.org/documentation/leap/virtualization/html/book-virtualization/app-gpu-passthru.html#gpu-passthru-verify-host "A.3.1. Verify the host environment"), for example `03:00.0`:
 
    ```
    > lspci -nn | grep 03:00.0
    ```

2.  Create the file `/etc/modprobe.d/vfio.conf` with the following content:

    `options vfio-pci ids=10de:1db4`
    
    *Verify that your card does not need an extra `ids=` parameter. For some cards, you must specify the audio device too, so that device's ID must also be added to the list, otherwise you will not be able to use the card.*

### A.3.5 Load the [VFIO](https://doc.opensuse.org/documentation/leap/virtualization/html/book-virtualization/gloss-vt-glossary.html#gloss-vt-acronym-vfio "VFIO") driver

There are four ways you can load the [VFIO](https://doc.opensuse.org/documentation/leap/virtualization/html/book-virtualization/gloss-vt-glossary.html#gloss-vt-acronym-vfio "VFIO") driver.

#### A.3.5.1 Including the driver in the initrd file

1.  Create the file `/etc/dracut.conf.d/gpu-passthrough.conf` and add the following content (mind the leading whitespace):

	`add_drivers+=" vfio vfio_iommu_type1 vfio_pci vfio_virqfd"`

1.  Re-generate the initrd file:

    ```
    > sudo dracut --force /boot/initrd $(uname -r)
    ```

#### A.3.5.2 Adding the driver to the list of auto-loaded modules

Create the file `/etc/modules-load.d/vfio-pci.conf` and add the following content:

```
vfio
vfio_iommu_type1
vfio_pci
kvm
kvm_intel
```

#### A.3.5.3 Loading the driver manually

To load the driver manually at runtime, execute the following command:

```
> sudo modprobe vfio-pci
```

#### A.3.5.4 Dynamically load from GRUB2 (preferred)

This culminates all previous steps into a one-liner (i.e. you do not need to make any of the suggested file changes) that can be disabled from the boot menu by editting the boot entry. Add the following (specific to my ROG Zephyrus only) to `/etc/default/grub` and reload GRUB2 with `sudo grub2-mkconfig -o /boot/grub2/grub.cfg`:

```
...
GRUB_CMDLINE_LINUX_DEFAULT="splash=silent modprobe.blacklist=nouveau,snd_hda_intel iommu=pt amd_iommu=on rd.driver.pre=vfio-pci vfio-pci.ids=10de:1f9d,10de:10fa mitigations=auto security=apparmor"
...
```

### A.3.6 Disable MSR for Microsoft Windows guests

 [](https://bugzilla.opensuse.org/enter_bug.cgi?&amp;product=openSUSE%20Distribution&amp;component=Documentation&amp;short_desc=%5Bdoc%5D%20Issue%20in%20%22Disable%20MSR%20for%20Microsoft%20Windows%20guests%22&amp;comment=Disable%20MSR%20for%20Microsoft%20Windows%20guests%3A%0A%0Ahttps%3A%2F%2Fdoc.opensuse.org%2Fdocumentation%2Fleap%2Fvirtualization%2Fhtml%2Fbook-virtualization%2Fapp-gpu-passthru.html%23gpu-passthru-disable-msr&amp;assigned_to=fs%40suse.com&amp;version=Leap%2015.4 "Report an issue") [](https://github.com/SUSE/doc-sle/edit/main/xml/gpu_passthru.xml "Edit source document")

For Microsoft Windows guests, we recommend disabling MSR (model-specific register) to avoid the guest crashing. Create the file `/etc/modprobe.d/kvm.conf` and add the following content:

options kvm ignore_msrs=1

COPY

### A.3.7 Install UEFI firmware

 [](https://bugzilla.opensuse.org/enter_bug.cgi?&amp;product=openSUSE%20Distribution&amp;component=Documentation&amp;short_desc=%5Bdoc%5D%20Issue%20in%20%22Install%20UEFI%20firmware%22&amp;comment=Install%20UEFI%20firmware%3A%0A%0Ahttps%3A%2F%2Fdoc.opensuse.org%2Fdocumentation%2Fleap%2Fvirtualization%2Fhtml%2Fbook-virtualization%2Fapp-gpu-passthru.html%23gpu-passthru-ovmf&amp;assigned_to=fs%40suse.com&amp;version=Leap%2015.4 "Report an issue") [](https://github.com/SUSE/doc-sle/edit/main/xml/gpu_passthru.xml "Edit source document")

For proper GPU Pass-Through functionality, the host needs to boot using UEFI firmware (that is, not using a legacy-style BIOS boot sequence). Install the qemu-ovmf package if not already installed:

```
> 
```

COPY

### A.3.8 Reboot the host machine

 [](https://bugzilla.opensuse.org/enter_bug.cgi?&amp;product=openSUSE%20Distribution&amp;component=Documentation&amp;short_desc=%5Bdoc%5D%20Issue%20in%20%22Reboot%20the%20host%20machine%22&amp;comment=Reboot%20the%20host%20machine%3A%0A%0Ahttps%3A%2F%2Fdoc.opensuse.org%2Fdocumentation%2Fleap%2Fvirtualization%2Fhtml%2Fbook-virtualization%2Fapp-gpu-passthru.html%23gpu-passthru-reboot&amp;assigned_to=fs%40suse.com&amp;version=Leap%2015.4 "Report an issue") [](https://github.com/SUSE/doc-sle/edit/main/xml/gpu_passthru.xml "Edit source document")

For most of the changes in the above steps to take effect, you need to reboot the host machine:

```
> 
```

COPY

## A.4 Configuring the guest

 [](https://bugzilla.opensuse.org/enter_bug.cgi?&amp;product=openSUSE%20Distribution&amp;component=Documentation&amp;short_desc=%5Bdoc%5D%20Issue%20in%20%22Configuring%20the%20guest%22&amp;comment=Configuring%20the%20guest%3A%0A%0Ahttps%3A%2F%2Fdoc.opensuse.org%2Fdocumentation%2Fleap%2Fvirtualization%2Fhtml%2Fbook-virtualization%2Fapp-gpu-passthru.html%23gpu-passthru-guest&amp;assigned_to=fs%40suse.com&amp;version=Leap%2015.4 "Report an issue") [](https://github.com/SUSE/doc-sle/edit/main/xml/gpu_passthru.xml "Edit source document")

This section describes how to configure the guest virtual machine so that it can use the host's NVIDIA GPU. Use Virtual Machine Manager or `virt-install` to install the guest VM. Find more details in [Chapter 9, _Guest installation_](https://doc.opensuse.org/documentation/leap/virtualization/html/book-virtualization/cha-kvm-inst.html "Chapter 9. Guest installation").

### A.4.1 Requirements for the guest configuration

 [](https://bugzilla.opensuse.org/enter_bug.cgi?&amp;product=openSUSE%20Distribution&amp;component=Documentation&amp;short_desc=%5Bdoc%5D%20Issue%20in%20%22Requirements%20for%20the%20guest%20configuration%22&amp;comment=Requirements%20for%20the%20guest%20configuration%3A%0A%0Ahttps%3A%2F%2Fdoc.opensuse.org%2Fdocumentation%2Fleap%2Fvirtualization%2Fhtml%2Fbook-virtualization%2Fapp-gpu-passthru.html%23gpu-passthru-guest-general&amp;assigned_to=fs%40suse.com&amp;version=Leap%2015.4 "Report an issue") [](https://github.com/SUSE/doc-sle/edit/main/xml/gpu_passthru.xml "Edit source document")

During the guest VM installation, select Customize configuration before install and configure the following devices:

-   Use Q35 chipset if possible.
    
-   Install the guest VM using UEFI firmware.
    
-   Add the following emulated devices:
    
    Graphic: Spice or VNC
    
    Device: qxl, VGA, or Virtio
    
    Find more information in [Section 13.6, “Video”](https://doc.opensuse.org/documentation/leap/virtualization/html/book-virtualization/cha-libvirt-config-gui.html#sec-libvirt-config-video "13.6. Video").
    
-   Add the host PCI device (`03:00.0` in our example) to the guest. Find more information in [Section 13.12, “Assigning a host PCI device to a VM Guest”](https://doc.opensuse.org/documentation/leap/virtualization/html/book-virtualization/cha-libvirt-config-gui.html#sec-libvirt-config-pci "13.12. Assigning a host PCI device to a VM Guest").
    
-   For best performance, we recommend using virtio drivers for the network card and storage.
    

### A.4.2 Install the graphic card driver

 [](https://bugzilla.opensuse.org/enter_bug.cgi?&amp;product=openSUSE%20Distribution&amp;component=Documentation&amp;short_desc=%5Bdoc%5D%20Issue%20in%20%22Install%20the%20graphic%20card%20driver%22&amp;comment=Install%20the%20graphic%20card%20driver%3A%0A%0Ahttps%3A%2F%2Fdoc.opensuse.org%2Fdocumentation%2Fleap%2Fvirtualization%2Fhtml%2Fbook-virtualization%2Fapp-gpu-passthru.html%23gpu-passthru-guest-driver&amp;assigned_to=fs%40suse.com&amp;version=Leap%2015.4 "Report an issue") [](https://github.com/SUSE/doc-sle/edit/main/xml/gpu_passthru.xml "Edit source document")

#### A.4.2.1 Linux guest

 [](https://bugzilla.opensuse.org/enter_bug.cgi?&amp;product=openSUSE%20Distribution&amp;component=Documentation&amp;short_desc=%5Bdoc%5D%20Issue%20in%20%22Linux%20guest%22&amp;comment=Linux%20guest%3A%0A%0Ahttps%3A%2F%2Fdoc.opensuse.org%2Fdocumentation%2Fleap%2Fvirtualization%2Fhtml%2Fbook-virtualization%2Fapp-gpu-passthru.html%23gpu-passthru-guest-driver-linux-rpm&amp;assigned_to=fs%40suse.com&amp;version=Leap%2015.4 "Report an issue") [](https://github.com/SUSE/doc-sle/edit/main/xml/gpu_passthru.xml "Edit source document")

PROCEDURE A.1: RPM-BASED DISTRIBUTIONS

 [](https://bugzilla.opensuse.org/enter_bug.cgi?&amp;product=openSUSE%20Distribution&amp;component=Documentation&amp;short_desc=%5Bdoc%5D%20Issue%20in%20%22RPM-BASED%20DISTRIBUTIONS%22&amp;comment=RPM-BASED%20DISTRIBUTIONS%3A%0A%0Ahttps%3A%2F%2Fdoc.opensuse.org%2Fdocumentation%2Fleap%2Fvirtualization%2Fhtml%2Fbook-virtualization%2Fapp-gpu-passthru.html%23id-1.8.10.5.4.2.2&amp;assigned_to=fs%40suse.com&amp;version=Leap%2015.4 "Report an issue") [](https://github.com/SUSE/doc-sle/edit/main/xml/gpu_passthru.xml "Edit source document")

1.  Download the driver RPM package from [http://www.nvidia.com/download/driverResults.aspx/131159/en-us](http://www.nvidia.com/download/driverResults.aspx/131159/en-us).
    
2.  Install the downloaded RPM package:
    
    ```
    > 
    ```
    
    COPY
    
3.  Refresh repositories and install cuda-drivers. This step will be different for non-SUSE distributions:
    
    ```
    > 
    ```
    
    COPY
    
4.  Reboot the guest VM:
    
    ```
    > 
    ```
    
    COPY
    

PROCEDURE A.2: GENERIC INSTALLER

 [](https://bugzilla.opensuse.org/enter_bug.cgi?&amp;product=openSUSE%20Distribution&amp;component=Documentation&amp;short_desc=%5Bdoc%5D%20Issue%20in%20%22GENERIC%20INSTALLER%22&amp;comment=GENERIC%20INSTALLER%3A%0A%0Ahttps%3A%2F%2Fdoc.opensuse.org%2Fdocumentation%2Fleap%2Fvirtualization%2Fhtml%2Fbook-virtualization%2Fapp-gpu-passthru.html%23id-1.8.10.5.4.2.3&amp;assigned_to=fs%40suse.com&amp;version=Leap%2015.4 "Report an issue") [](https://github.com/SUSE/doc-sle/edit/main/xml/gpu_passthru.xml "Edit source document")

1.  Because the installer needs to compile the NVIDIA driver modules, install the gcc-c++ and kernel-devel packages.
    
2.  Disable Secure Boot on the guest, because NVIDIA's driver modules are unsigned. On SUSE distributions, you can use the YaST GRUB 2 module to disable Secure Boot. Find more information in Book “_Reference_”, Chapter 14 “UEFI (Unified Extensible Firmware Interface)”, Section 14.1.1 “Implementation on openSUSE Leap”.
    
3.  Download the driver installation script from [https://www.nvidia.com/Download/index.aspx?lang=en-us](https://www.nvidia.com/Download/index.aspx?lang=en-us), make it executable, and run it to complete the driver installation:
    
    ```
    > 
    ```
    
    COPY
    
4.  Download CUDA drivers from [https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=SLES&target_version=15&target_type=rpmlocal](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=SLES&target_version=15&target_type=rpmlocal) and install following the on-screen instructions.
    

![Note](https://doc.opensuse.org/documentation/leap/virtualization/html/book-virtualization/static/images/icon-note.svg "Note")

Note: Display issues

After you have installed the NVIDIA drivers, the Virtual Machine Manager display will lose its connection to the guest OS. To access the guest VM, you must either login via `ssh`, change to the console interface, or install a dedicated VNC server in the guest. To avoid a flickering screen, stop and disable the display manager:

```
> 
```

COPY

PROCEDURE A.3: TESTING THE LINUX DRIVER INSTALLATION

 [](https://bugzilla.opensuse.org/enter_bug.cgi?&amp;product=openSUSE%20Distribution&amp;component=Documentation&amp;short_desc=%5Bdoc%5D%20Issue%20in%20%22TESTING%20THE%20LINUX%20DRIVER%20INSTALLATION%22&amp;comment=TESTING%20THE%20LINUX%20DRIVER%20INSTALLATION%3A%0A%0Ahttps%3A%2F%2Fdoc.opensuse.org%2Fdocumentation%2Fleap%2Fvirtualization%2Fhtml%2Fbook-virtualization%2Fapp-gpu-passthru.html%23id-1.8.10.5.4.2.5&amp;assigned_to=fs%40suse.com&amp;version=Leap%2015.4 "Report an issue") [](https://github.com/SUSE/doc-sle/edit/main/xml/gpu_passthru.xml "Edit source document")

1.  Change the directory to the CUDA sample templates:
    
    ```
    > 
    ```
    
    COPY
    
2.  Compile and run the `simpleTemplates` file:
    
    ```
    > 
    ```
    
    COPY
    

#### A.4.2.2 Microsoft Windows guest

 [](https://bugzilla.opensuse.org/enter_bug.cgi?&amp;product=openSUSE%20Distribution&amp;component=Documentation&amp;short_desc=%5Bdoc%5D%20Issue%20in%20%22Microsoft%20Windows%20guest%22&amp;comment=Microsoft%20Windows%20guest%3A%0A%0Ahttps%3A%2F%2Fdoc.opensuse.org%2Fdocumentation%2Fleap%2Fvirtualization%2Fhtml%2Fbook-virtualization%2Fapp-gpu-passthru.html%23gpu-passthru-guest-driver-windows&amp;assigned_to=fs%40suse.com&amp;version=Leap%2015.4 "Report an issue") [](https://github.com/SUSE/doc-sle/edit/main/xml/gpu_passthru.xml "Edit source document")

![Important](https://doc.opensuse.org/documentation/leap/virtualization/html/book-virtualization/static/images/icon-important.svg "Important")

Important

Before you install the NVIDIA drivers, you need to hide the hypervisor from the drivers by using the `<hidden state='on'/>` directive in the guest's `libvirt` definition, for example:

<features>
 <acpi/>
 <apic/>
 <kvm>
  <hidden state='on'/>
 </kvm>
</features>

COPY

1.  Download and install the NVIDIA driver from [https://www.nvidia.com/Download/index.aspx](https://www.nvidia.com/Download/index.aspx).
    
2.  Download and install the CUDA toolkit from [https://developer.nvidia.com/cuda-downloads?target_os=Windows&target_arch=x86_64](https://developer.nvidia.com/cuda-downloads?target_os=Windows&target_arch=x86_64).
    
3.  Find some NVIDIA demo samples in the directory `Program Files\Nvidia GPU Computing Toolkit\CUDA\v10.2\extras\demo_suite` on the guest.