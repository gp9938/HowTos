Hyper-V virtual switches allow you to create a network for your Hyper-V VMs.  My only use right now is to allow direct IP access to my VMs, mostly via ssh.

1. From the Hyper-V Manager, go to the Action pulldown menu and select Virtual Switch Manager...
2. On the upper-left of the window, the menu item "New virtual network switch" should already be selected
3. On the upper-right of the window, the menu item "External" for "Create virtual switch" should already be selected
4. Click the button "Create Virtual Switch"
5. Under Name, choose a descriptive name as in `ExternalVirtualSwitch` or something like that.
6. Under Connection type, select the network adapter  (a.k.a network card) you are using that provides the appropriate external network access. 
7. Make sure "Allow management operation system to share this network adapter" is checked unless you are able to dedicate a network adapter to Hyper-V
8. Click the Ok button and read the warning that applying network changes will temporarily interrupt your host's network access.

## Updating Networking on an Existing Hyper-V VM
This network adapter can now be selected when creating a new VM in Hyper-V and is also to be added to any of the existing VMs as well.

1. Be sure the state of the existing VM is  "Off" and not "Saved," "Running," etc.  Network changes are disabled unless the VM is Off.  If the state is Saved, you can start the VM, wait for the VM to restore from the saved state, and then right-click  and select "Shutdown ..."
	1. Alternatively, if you are okay with losing state and potentially causing an OS issue for the saved VM, you can right-click and select "Delete Saved State..." to bring the State to off.  This can damage the OS or running application state for that system, so be careful.
2. Now right-click and select Settings....
3. One the left side bar of the window, select "Network Adapter"
4. On the right side of the window,  where it says "Virtual switch", click and select the switch you just created.   
5. Click Okay and start the VM to check if it worked
6. Note that the Network Adapter section in Settings has a section for Hardware Acceleration and Advanced Features that are worth reviewing and understanding for your specific situation.   The defaults are probably okay for most typical, non-enterprise VM uses.

