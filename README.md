# (Status: WORK IN PROGRESS) FreeBSD disk encryption with zfs and remote-boot functionality
##This fork uses only one drive (instead two mirrord drives) and should run on FreeBSD 13.2 and 14.0

This repository contains some scripts to help you set up a nearly full-disk-encrypted FreeBSD where you can enter the password via ssh.

This is accompished by having a small ufs partition consisting of the kernel, modules and a statically linked dropbear. It enables networking, and starts dropbear to allow you to log on. Once you entered the password, the boot switches the root filesystem and boot resumes from the zfs same as a normal unencrypted boot.

If you are at the console, you can hit ^c to kill the dropbear and you will be dropped into a shell where you can enter the password at the console and resume booting.



# Usage[Setup]:

Boot a FreeBSD install (either memstick or disc1 is fine). And at the first window (after Boot-Menu) select "shell"

In the shell:

* [OPTIONAL] set new keyboard layout, if your non-english with

`kbdmap`



* get your network up and running, e.g. with

`dhclient re0`

* change to a writeable path:

`cd /tmp`

* then fetch the files from this repo. For example directly from github:

```
fetch https://raw.githubusercontent.com/oxfox22/freebsd-remote-crypto-single-drive/master/CRYPT
fetch https://raw.githubusercontent.com/oxfox22/freebsd-remote-crypto-single-drive/master/PREBOOT
```

* Check the name of your drive.
```
dmesg|grep sectors
```

If it is not "ada0", you will need to change the `geom0=` line at the top of the `CRYPT` script.
`ee CRYPT`

Now run the script

```
sh CRYPT
```

It will ask you for your crypt passphrase (only once, so type carefully), later throw you into `ee` to edit a minimal `/etc/rc.conf` (set the hostname and IP) and after you quit the vi it should end with "All ok"

After that, you can remove your installation media and reboot the machine. It will come up with a shell. Run "sh DWIM" and enter your passphrase to boot to the final system.

# Static dropbear:
We need a static dropbear binary for this to work properly.
Make sure you have the ports tree on your system:
```
portsnap fetch
portsnap extract
```
To compile and install it,
```
pkg install gmake indexinfo gettext-runtime
cd /usr/ports/security/dropbear
make OPTIONS_SET="STATIC ECDSA DSA BLOWFISH" BATCH=YES install clean
pkg lock -y dropbear
```

The last line "lock"s the dropbear so "pkg upgrade" will not replace it with a dynamically linked version.

After this installation, run the `PREBOOT update` as described below at least once to make sure your preboot environment is properly setup


# Maintenance:

After each system/kernel update (and root password change), run the `PREBOOT` script.

```
sh PREBOOT update
```
This updates your preboot environment and makes sure everything stays secure and bug-free :-)

Also periodically check if your dropbear is up to date with 
```
pkg audit
```

and if necessary, update it.

```
cd /usr/ports/security/dropbear
pkg unlock -y dropbear
pkg delete -y dropbear
make OPTIONS_SET="STATIC ECDSA DSA BLOWFISH" BATCH=YES install clean
pkg lock -y dropbear
```

don't forget to run the `PREBOOT` script as above after this.

# Usage[boot]:

Either ssh to the box as root, or hit ^c on the console to get a shell. Run the provided script

```
sh DWIM
```

and you will be asked for the crypto passphrase (twice), after which the boot will automatically continue. If you are logged in via ssh, your connection will die. Give it a few moments to come up properly, then you should be able to re-login

# Additional scripts

The `SSH` script is a template on how to enable SSH from the setup shell. This might be a useful starting point if you want to replicate this setup on a remote machine without console access.

If you have a dropbear binary in `bin/` during the setup run of `CRYPT`, it will already copy it to the correct place.

# Help

If you run into problems, please open an issue.
