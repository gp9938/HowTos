
Apple has blocked the `rclone` built-in authentication mechanism from working.  Rclone developers have been able to work around this but it is best to work around this.

After installing `rclone`, instead of using the interactive `rclone config` command where you are prompted through the setup that then uses the traditional authorization mechanism, take the following steps.


### Initial `rclone` Configuration
1. Login into `icloud.com` from your browser, making sure to **Trust** the browser when asked.   This is important because it will give you a 30 session key instead of a 30 minute session key.
2. Go to the `Drive` item displayed on the icon tray (going to `Drive` may not be strictly necessary)
3. Bring up your browser's Developer Tools (typically `CTRL-SHIFT-I`) and go to the `Network` tab shown on the top bar.
4. Refresh/reload (`F5`) the iCloud page to bring the `Network` tab up-to-date.
5. Go back to the Developer Tools and select any of the `GET` request lines that has www.icloud.com as the Domain.
6. That should bring up a sub-window showing details of the request
7. Select the `Headers` tab of the sub-window and find the area where `Request-Headers` are shown 
8. Find the `Cookie` property, which represents all the cookies on that request and copy the (long) value.  This value (without quotes) will become the value of `cookies = `  in the `rclone` configuration file.
9. Go to the `Cookies` tab of the sub-window and find the cookie named `X-APPLE-WEBAUTH-HSA-TRUST` and copy the value.  This value (without quotes) will become the value of the `trust_token = `
10. Finally 
11. Now create or update the `rclone` config file by adding in your entry to file `$HOME/.config/rclone/rclone.conf`
```
[<name-of-remote>]
type = iclouddrive
apple_id = <your-apple-id>
cookies = <the-value-of-Cookie-property-no-quotes>
trust_token = <the-value-of-HSA-TRUST-cookie-no-quotes>

```
 11. No, you do not need a `password = ` entry this this file as we are not going to try to authenticate using `rclone`, since, as stated above, `rclone` authentication may be blocked by Apple. You do not need the `apple_id = ` entry either but things get confusing if you leave this out of the configuration
 12. Now validate `rclone.conf` file using this command: `rclone listremotes`.  It should just list the iCloud remote you just created plus any other remotes you have already setup.
 13. Validate you can reach the remote using the command `rclone lsd <name-of-remote>:`  For example, if the name of your remote is just `icloud` your command would be `rclone lsd icloud:` note the `:` (colon) terminator after the name of your remote.

### Mounting with `rclone`
Once the `rclone` configuration is working for your iCloud remote, you can access it using the `rclone` commands, listed if you type `rclone` with no arguments by using `rclone` commands like `ls`, `copy`, and `delete`. 

Alternatively, you can choose to mount the filesystem so you can access the files directly.  

#### Notes
- Reliability of using iCloud as a direct mount is going to be lower than what you might get from other remote filesystems.  For the highest reliability ALWAYS use it with  `vfs-cache-mode` `full`.   This will make `rclone` keep a locally copy of all the files you access so that reads and especially writes are more reliable and allow `rclone` to retry iCloud when there are errors.
- If you `rclone` mount in the foreground, a simple `CTRL-C` will result in the filesystem being unmounted
- If you background it using the `--daemon` flag, you will need to unmount it with the `fusermount`, `fusermount3`, or, if you use the `nfsmount` features, `umount`
- Expect performance to be extremely limited and limit your use to document updates or batch copies and do not use iCloud as an applications working directory.
- Refer to `rclone` `Mounting on Linux` documentation here https://rclone.org/commands/rclone_mount/#mounting-on-linux
#### Mounting Commands
There are many options to `rclone`, be sure to look at all of them when encountering problems.

##### Recommended `rclone` mount command
```
rclone mount <name-of-remote>: <local-path-to-mount-dir> --daemon --vfs-cache-mode full 
```

For example, if the name of your `rclone` remote for iCloud is `icloud`, and you want to mount it on under your home directory in sub-directory `iCloud` :
```
rclone mount icloud: $HOME/iCloud --daemon --vfs-cache-mode full
```

To unmount use
```
fusermount3 -u <local-path-to-mount-dir>
```

For example:
```
fusermount3 -u $HOME/iCloud
```



