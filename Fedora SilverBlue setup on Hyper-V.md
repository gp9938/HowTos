1. Create a virtual switch (unless it exists) in Hyper-V with "External network:" access to the external NIC for the host machine    
2. Download Fedora SilverBlue ISO
3. Create Hyper-V Generation 2 virtual machine
    1. Defaullt memory okay
    2. Use the virtual switch created above
    3. Set the Fedora ISO as the install image on the virtual CD/DVD drive
    4. DISABLE SECURE BOOT
4. Before starting the new virtual machine, connect to it (Virtual Machine Connection in Hyper-V)
5. The new Virutal Machine Connection window should have "Start" button in the middle
6. Click on the start button and prepare to hit down-arrow to select the last entry (probably the third entry), UEFI Firmware Settings
7. From that menu select Install Fedora with Basic Video Driver
    1. Without that selection, there will be no GUI and no way to fully setup the machine -- Silverblue does _not_ suppoort Ignition
8. Edit /etc/default grub and add `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash video-hyperv_fb :3840x2160"`
9. Run grub2-mkconfig -o /etc/grub2-efi.cfg
10. Shutdown the VM
11. From a pwsh admin shell, run `set-vmvideo <hyper-v-vm-name> -horizontalresolution:3840 -verticalresolution:2160 -resolutiontype single`
12. And: `set-vm <hyper-v-vm-name> -EnhancedSessionTransportType HVSocket`    
13. Upon reboot, the "Displays" section of "Settings" will allow you choose from a variety of resolutions within 3840x2160 res
14. Enable proper mouse pointer tracking in Hyper-V client: When in the os -- as root, edit /etc/environment MUTTER_DEBUG_FORCE_KMS_MODE=simple