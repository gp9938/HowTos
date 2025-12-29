How To Install Fedora Silverblue on Hyper-V
=
# Overview
Fedora Silverblue is only available to install via a Hybrid ISO image which is not bootable by Hyper-V or VirtualBox.  Hybrid ISOs are designed to be burned onto USB drives for subsequent booting.  The solution is user Rufus to burn a virtual hard disk (VHD or VHDX) with the Silverblue ISO and then make that the boot disk of a new Hyper-V VM.  The Hyper-V VM is also created with a secondary blank virtual disk.

When the VM boots for the first time, it will kick off the Silverblue installer which you need to click through.  That's it.
# Steps

The steps below are the specifics needed to get this to happen:

1. Download the lastest Silverblue iso from   https://fedoraproject.org/atomic-desktops/silverblue/download
2. Create a VHDX on Windows of 127 GB or more
	1. Use the Action pulldown menu from Windows Disk Management utility to do this
	2. Size of 16 GB should be more than enough
	3. You will get the option for VHD or VHDX -- VHDX is safest
	4. Windows will auto attach this new virtual disk after it is created
3. Use latest Rufus utility to install Fedora HybridISO (a.k.a USB iso image) onto VHDX
	1. Download from https://rufus.ie/
	2. Once Rufus starts up, you should see the first line list the new 16 GB disk name as Rufus will see it as the obvious target  for image creation.  If not, click on the label and find 16 GB drive you just created
	3. On the second line, click the the "Select" button to select the Silverblue iso downloaded in step 1.
	4. Select option Choose GPT over MBR
	5. If there is a warning about an unknown version of Grub, just allow it to try to download the correct one (it should have no issue doing so)
	6. Create the image.  Note the _strange_ user interface of Rufus will not make it obvious things have succeeded.  It is _best_ to look at Rufus while it creates the disk image, it does not take long.
4. Now detach (or eject) the virtual disk from step 2.
5. Create Hyper-V machine 
	1. Use a type 2 VM
	2. Set the  boot disk as VHDX
	3. Give the VM a blank VHDX of at least 127 GB (Hyper-V can do this creation itself)
	4. Be sure to setup the VM to use a Hyper-V Virtual Switch that has access to the Internet (an external virtual switch).   To create one:
		1. Go to the Hyper-V righthand-side pane titled  `Actions` and select item `Virtual Switch Manager...`. 
		2. The sub-window that pops up will have `New virtual network switch` selected in the lefthand-side pane.  On the righthand-side, select `External` and then push the button `Create Virtual Switch`.
		3. Now name the new virtual switch something meaningful like `ExternalVirtualSwitch` and select `Connection type` with `External network:` and select an **active** network interface for your host.
		4. Ensure the `Allow management operating system to share this network adapter` is checked unless you know it does not need to be checked.
		5. Click the `OK` button
	5. Click `OK`
6. Update the new Hyper-V machine by selecting `Settings...` for it.
	1. Under `Security` **uncheck** `Enable Secure Boot` unless you know how to set this up.  Without checking this, Hyper-V will not be able to start the boot process for your machine.
	2.  Also under Security, check `Enable Trusted Platform Module` if you prefer -- this may not be necessary for your use case.
	3. Under `Integration Services` optionally enable  `Guest Services` in Hyper-V so that you can more easily share resources with your VM. 
	4. Click the `OK` button
7. Connect to the VM
8. Start the VM and wait for the Fedora installer to start
9. Install Fedora
10. Done.

## Optional Display Setup
To unlock more display options for your VM you can take the following steps:
1. Start a `pwsh` admin shell, 
2. Run these commands making sure to update the resolution to the appropriate one for your own monitor (4k monitor resolution shown):
```
set-vmvideo <hyper-v-vm-name> -horizontalresolution:3840 - \ verticalresolution:2160 -resolutiontype single
set-vm <hyper-v-vm-name> -EnhancedSessionTransportType HVSocket
```

# To Work Around a Blank Screen
If the boot seems to start but the install screen isn't being shown, this can be a display incompatibility.  You can work around this by forcing the installer to use the `Basic Video Driver`

1. Before starting the new virtual machine, connect to it  using Virtual Machine Connection in Hyper-V
2. The new Virtual Machine Connection window should have a  `Start` button in the center
3. Click on the `Start` button and prepare to hit down-arrow to select the last entry (probably the third entry) in the displayed text menu titled `UEFI Firmware Settings`
4. Next, select `Install Fedora with Basic Video Driver`.  This step will force the installer to avoid the full video driver that is not fully working.
5. You will now get a simple Linux command line where you will edit `/etc/default grub`.   To `/tec/default/grub`,  add `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash video-hyperv_fb :3840x2160`. Be sure to update the resolution to the appropriate one for your monitor.
6. Save the file and run the command `grub2-mkconfig -o /etc/grub2-efi.cfg`
7. `reboot` the machine
