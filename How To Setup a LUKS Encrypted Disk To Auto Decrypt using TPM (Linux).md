
From https://www.reddit.com/r/Fedora/comments/szlvwd/psa_if_you_have_a_luks_encrypted_system_and_a/

First of all, you need to check if you actually have a TPM2 chip, just to make sure. To do that, run `cat /sys/class/tpm/tpm0/device/description` or `cat /sys/class/tpm/tpm0/tpm_version_major`. You'll get a result showing your TPM2 chip if you have one working. So, if you do, time to use it.

After that, run `lsblk` to make sure which partition houses your LUKS container.

- Run `systemd-cryptenroll --tpm2-device=auto /dev/$DEVICE`, where $DEVICE is your partition where your LUKS encryption rests (in my case /dev/sda3).
    
- After that, edit `/etc/crypttab`, making it look something like this:
    
`luks-$UUID UUID=$UUID - tpm2-device=auto,discard`

I'll reference it as **(part1) (part2) (part3) (part4)** for an easier understanding of the file's contents

The changes you'll do in this case is changing **part3** to `-` and adding `tpm2-device=auto` to **part4**.

After that, make `dracut` aware of the tpm2 by creating the `/etc/dracut.conf.d/tss2.conf` file, with its contents being the following:

`install_optional_items+=" /usr/lib64/libtss2* /usr/lib64/libfido2.so.* "`

After that, recreate the `initramfs` by running `sudo dracut -f` and reboot your system. If everything went correctly, your system should automatically boot without the need to input your LUKS password.

----
Additional notes:
- You may need to run `systemd-cryptenroll --password <device>` to enter in the correct password if not properly prompted or if the prompt on the first use of the command is ambiguous.
- Note that `tang` servers can be used to for network based key distribution to decrypt volumes .  There is lots of documentation online about LUKS and using `tang` with `clevis` to manage this
- `tang` is not really for laptop computers


