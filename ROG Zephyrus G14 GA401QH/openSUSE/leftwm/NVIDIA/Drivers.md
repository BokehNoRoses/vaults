## Open GPU kernel modules versus Proprietary drivers[](https://en.opensuse.org/SDB:NVIDIA_drivers#Open_GPU_kernel_modules_versus_Proprietary_drivers)

The following article is about installing NVIDIA's Proprietary drivers. For more information about the Open GPU kernel modules, that NVIDIA released in May 2022, read [this openSUSE Blog article](https://sndirsch.github.io/nvidia/2022/06/07/nvidia-opengpu.html).

## Situation[](https://en.opensuse.org/SDB:NVIDIA_drivers#Situation)

Installing the official NVIDIA drivers using [ZYpp](https://en.opensuse.org/Portal:Libzypp) (YaST or Zypper) is desired.

For information regarding NVIDIA's official linux .run file see ["the hard way"](https://en.opensuse.org/SDB:NVIDIA_the_hard_way) page.

## Procedure[](https://en.opensuse.org/SDB:NVIDIA_drivers#Procedure)

Installing with YaST or Zypper requires root privileges.

### Special Requirements[](https://en.opensuse.org/SDB:NVIDIA_drivers#Special_Requirements)

You need to have multiversion feature for kernel packages enabled, i.e. have the following entry in /etc/zypp/zypp.conf:

multiversion = provides:multiversion(kernel)

### Add the Nvidia Repository[](https://en.opensuse.org/SDB:NVIDIA_drivers#Add_the_Nvidia_Repository)

The NVIDIA drivers can not be included with openSUSE because of their [license](https://en.opensuse.org/Restricted_formats#NVIDIA_graphics_drivers "Restricted formats"). Conveniently, NVIDIA has an openSUSE repository that can be added and downloaded from.

#### YaST[](https://en.opensuse.org/SDB:NVIDIA_drivers#YaST)

1.  Open YaST, then click Software Repositories.
2.  Click Add (in the bottom left), then select Community Repositories.
3.  Select NVIDIA Graphics Drivers, then click OK.

#### Zypper[](https://en.opensuse.org/SDB:NVIDIA_drivers#Zypper)

##### Leap[](https://en.opensuse.org/SDB:NVIDIA_drivers#Leap)

# zypper addrepo --refresh '[https://download.nvidia.com/opensuse/leap/$releasever'](https://download.nvidia.com/opensuse/leap/$releasever') NVIDIA

##### Tumbleweed[](https://en.opensuse.org/SDB:NVIDIA_drivers#Tumbleweed)

# zypper addrepo --refresh [https://download.nvidia.com/opensuse/tumbleweed](https://download.nvidia.com/opensuse/tumbleweed) NVIDIA

### Get the hardware information[](https://en.opensuse.org/SDB:NVIDIA_drivers#Get_the_hardware_information)

In a terminal:

# lspci | grep VGA
# lscpu | grep Arch

Alternatively:

# hwinfo --gfxcard | grep Model
# hwinfo --arch

Using inxi utility:

# inxi -G
# inxi -Ga

This information is also available in the Display subsection at **YaST Hardware Information**.

### Install[](https://en.opensuse.org/SDB:NVIDIA_drivers#Install)

One way to determine the appropriate driver is to input your hardware information into [Nvidia's driver search engine](https://www.nvidia.com/Download/index.aspx). Beware: Nvidia's site may declare shorter support period with the same chip for mobile graphics compared to desktop video cards. This is not true for Linux drivers (more info [here](https://forums.opensuse.org/showthread.php/571628-Finding-compatible-drivers-for-Nvidia-mobile-videocards)). openSUSE provides [unified Linux driver](https://bugzilla.opensuse.org/show_bug.cgi?id=1195885#c81) for all flavours: GeForce and Quadro, desktop and mobile. The driver decides during startup whether it supports this GPU or not.

Note: Specific video cards are mapped to the following naming convention listed below. You will need this information when you are ready to install via commandline/zypper.

-   G03 = driver v340 = legacy driver for GT8xxx/9xxx devices
-   G04 = driver v390 = legacy driver for GTX4xx/5xx Fermi devices
-   G05 = current driver for current devices
-   G06 = covers all cards GT700 and up.

  
Alternatively, YaST Software or the following commands can be used to check the available packages:

# zypper se x11-video-nvidiaG0* nvidia-video-G06*
S | Name                | Summary                                                 | Type
--+---------------------+---------------------------------------------------------+--------
  | nvidia-video-G06    | NVIDIA graphics driver for GeForce 700 series and newer | package
  | x11-video-nvidiaG04 | NVIDIA graphics driver for GeForce 400 series and newer | package
  | x11-video-nvidiaG05 | NVIDIA graphics driver for GeForce 600 series and newer | package

  

# zypper se -s x11-video-nvidiaG0* nvidia-video-G06*
S | Name                | Type    | Version         | Arch   | Repository
--+---------------------+---------+-----------------+--------+------------------------
  | nvidia-video-G06    | package | 525.89.02-5.2   | x86_64 | nVidia Graphics Drivers
  | x11-video-nvidiaG04 | package | 390.157-19.1    | x86_64 | nVidia Graphics Drivers
  | x11-video-nvidiaG05 | package | 470.161.03-59.2 | x86_64 | nVidia Graphics Drivers

  
If you are going to install CUDA later, then you must use nvidiaG05 version for compatibility (as of Jan 18, 2022).

To take advantage of OpenGL acceleration you must install an additional package, choosing the one that corresponds to the driver:

# zypper se nvidia-gl*G0*
S | Name          | Summary                                         | Type
--+---------------+-------------------------------------------------+--------
  | nvidia-gl-G06 | NVIDIA OpenGL libraries for OpenGL acceleration | package
  | nvidia-glG04  | NVIDIA OpenGL libraries for OpenGL acceleration | package
  | nvidia-glG05  | NVIDIA OpenGL libraries for OpenGL acceleration | package

  
Patched G03 drivers were provided by community user: [https://software.opensuse.org/search?baseproject=ALL&q=g03](https://software.opensuse.org/search?baseproject=ALL&q=g03). You may install them after adding required repository.

  

#### YaST[](https://en.opensuse.org/SDB:NVIDIA_drivers#YaST_2)

1.  Go to the YaST Control Center and click Software Management.
2.  View > Repositories > NVIDIA
3.  Choose the appropriate driver, e.g. x11-video-nvidiaG04 or x11-video-nvidiaG05 or nvidia-video-G06
4.  Optionally choose the corresponding OpenGL acceleration libraries; nvidia-glG04 or nvidia-glG05 or nvidia-gl-G06.
5.  Press Accept.
6.  Restart your computer.

#### Zypper[](https://en.opensuse.org/SDB:NVIDIA_drivers#Zypper_2)

Once you know which product is mapped to the appropriate driver command mapping (e.g. G04 or G05 or G06), you can use zypper to install.

# zypper in <x11-video-nvidiaG04 or x11-video-nvidiaG05 or nvidia-video-G06>
# zypper in <nvidia-glG04 or nvidia-glG05 or nvidia-gl-G06>

Restart your computer.

#### Secureboot[](https://en.opensuse.org/SDB:NVIDIA_drivers#Secureboot)

Kernels in Leap 15.2 or higher (and Tumbleweed since Kernel 6.2.1) will, by default, refuse to load any unsigned kernel modules on machines with [secure boot](https://www.suse.com/c/uefi-secure-boot-overview/) enabled.

During the NVIDIA driver installation on a secureboot system a MOK keypair is being created and the kernel modules been signed with the created private key. The created certificate (public key) remains on the storage below /var/lib/nvidia-pubkeys, but it also needs to be imported to the list of to be enrolled MOK pubkeys.

After the first reboot this certificate can easily be enrolled to the [MOK database](https://www.suse.com/c/uefi-secure-boot-details/). The EFI tool for this (mokutil) is automatically started: inside the tool select "Enroll MOK", then "Continue", then "Yes". Use your root password (US keyboard layout!) when prompted for a password. The certificate is now added to the MOK database and is considered trusted, which will allow kernel modules with matching signatures to load. To finish, select "Reboot".

[![Nvidia-secureboot-enrollKey.jpg](https://en.opensuse.org/images/6/66/Nvidia-secureboot-enrollKey.jpg)](https://en.opensuse.org/File:Nvidia-secureboot-enrollKey.jpg)

In case you miss the timeout for certificate enrollment after first reboot, you can easily import again the certificate by running the following command:

# mokutil --import /var/lib/nvidia-pubkeys/MOK-nvidia-gfxG0<X>-<driver_version>-<kernel_flavor>.der --root-pw

Then reboot the machine and enroll the certificate as described before.

**As the last resource**, in case you are having problems with secure boot, you can, at your own risk, disable validation for kernel modules:

# mokutil --disable-validation

##### Driver Update[](https://en.opensuse.org/SDB:NVIDIA_drivers#Driver_Update)

During a driver update the old and no longer being used public key is being registered to be deleted from the MOK data base. So in addition to the "Enroll MOK" menu entry a "Delete MOK" entry will appear in the EFI tool once you reboot the machine. In order to finally remove it from the MOK data base, select "Delete MOK", then "Continue", then "Yes". Again use your root password (US keyboard layout!) when prompted for a password. You can show the certificate/description of the public key when selecting "View Key X" in order not to delete the wrong key. Press any key to continue from there.

[![Nvidia-secureboot-deleteKey.jpg](https://en.opensuse.org/images/f/f5/Nvidia-secureboot-deleteKey.jpg)](https://en.opensuse.org/File:Nvidia-secureboot-deleteKey.jpg)

## Uninstalling the NVIDIA drivers[](https://en.opensuse.org/SDB:NVIDIA_drivers#Uninstalling_the_NVIDIA_drivers)

### YaST[](https://en.opensuse.org/SDB:NVIDIA_drivers#YaST_3)

1.  Start YaST, go to: Software -> Software Management
2.  Change the 'Filter' to filter by software repositories
3.  Select the respective NVIDIA repository
4.  Mark any installed package from this repository for deletion and press 'Accept'. You may be prompted for conflicts, please ignore any conflicts and chose to break dependencies.
5.  Now in YaST select: Software -> Software Repositories
6.  Chose the respective NVIDIA repository and mark it 'disabled' - don't delete it as it will return enabled the next time the repositories are synced with the server.

Uninstalling the proprietary drivers will restore the previous X configuration file `/etc/X11/xorg.conf` if one existed. If the hardware has changed in the mean time it may be necessary to manually edit this file.

### Zypper[](https://en.opensuse.org/SDB:NVIDIA_drivers#Zypper_3)

 # zypper rm <x11-video-nvidiaG04 or x11-video-nvidiaG05>

If that doesn't delete all the packages you can find the other names with

 # zypper se -ir NVIDIA

or

 # zypper lr
 # zypper se -ir <the repo number>

Also, the installer for the NVIDIA driver might have added nouveau to the blocklist; to be able to run the modesetting DDX driver or nouveau DDX driver again make sure there are no files containing the words `blacklist nouveau` at /etc/modprobe.d/, as the installer might fail to remove these.

After uninstalling the packages, you might need to recreate the initrd by running:

 # dracut -f

## Troubleshooting[](https://en.opensuse.org/SDB:NVIDIA_drivers#Troubleshooting)

-   [https://en.opensuse.org/SDB:NVIDIA_troubleshooting](https://en.opensuse.org/SDB:NVIDIA_troubleshooting)
-   If your computer freezes before the login screen after installing the propietary drivers and you are using GDM (usually the case if you are using GNOME), try adding _**WaylandEnable=false**_ in /etc/gdm/custom.conf.
-   KDE may not work with Wayland protocol.
-   You can verify the driver was actually loaded by running `lsmod | grep nvidia` in the terminal. The output should be like:

nvidia_drm             57344  2
nvidia_modeset       1187840  3 nvidia_drm
nvidia_uvm           1110016  0
nvidia              19771392  81 nvidia_uvm,nvidia_modeset
drm_kms_helper        229376  2 nvidia_drm,i915
drm                   544768  13 drm_kms_helper,nvidia_drm,i915

The numbers in the middle column do not need to be the same. If the driver is loaded the problem relies elsewhere, since that means it was installed successfully.

-   As stated before in this guide, if you are using secure boot make sure you accept the MOK, else the module won't load. One way to know see if secure boot could be blocking the module is looking at the output of `dmesg` and search for warnings like the following:

Lockdown: modprobe: unsigned module loading is restricted; see man kernel_lockdown.7
modprobe: ERROR: could not insert 'nvidia': Required key not available

## See also[](https://en.opensuse.org/SDB:NVIDIA_drivers#See_also)

### Optimus[](https://en.opensuse.org/SDB:NVIDIA_drivers#Optimus)

Users on hardware configurations with [NVIDIA Optimus](https://www.nvidia.com/en-us/geforce/technologies/optimus/) (usually the case on laptops) are advised to read [SDB: NVIDIA SUSE Prime](https://en.opensuse.org/SDB:NVIDIA_SUSE_Prime). Offloading to the dedicated GPU is possible using the instructions at [PRIME Render Offload](https://download.nvidia.com/XFree86/Linux-x86_64/455.38/README/primerenderoffload.html)

### CUDA[](https://en.opensuse.org/SDB:NVIDIA_drivers#CUDA)

Developers and users involved in High Performance Computing applications may want to install CUDA libraries. Additional instructions are provided at the [CUDA Toolkit Documentation](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html) and download links at [NVIDIA Developer](https://developer.nvidia.com/cuda-downloads)