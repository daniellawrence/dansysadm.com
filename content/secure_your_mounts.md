Title: Security: Mount /tmp, /var/tmp & /dev/shm with nodev, nosuid and noexec
Date: 2014-08-03 10:36
Category: blog
Tags: linux, security, mount, ext4

The goal of this will be to make it harder for your system to be exploited by removing
a very common path taken by script kiddies.

/tmp, /var/tmp/ and /dev/shm are all world-writable, so every process (& therefore user) is able to write files into
these directories.
Many malicious scripts take advantage of this are is used as a temporary storage space for unwanted executables


We are going to lock down these directories removing the dev, suid and exec options from the mounts.

*nodev*
Do not interpret character or block special devices on the file system.

*noexec*
Block the execution of any files.

*nosuid*
Block set-user-identifier & set-group-identifier bits on files and directories.


Getting /tmp and /var/tmp on their own file-system
---------------------------------------------------------

Create a file on your root partition that will be used as a loopback mount for /tmp

	$ sudo dd if=/dev/zero of=/tmp.bin bs=1 count=0 seek=1G
	
Check the file has been created.

	$ ls -l /tmp.bin
	-rw-r--r-- 1 root root 1.0G Aug  3 10:45 /tmp.bin	

Make this file into a ext4 file-system

	$ sudo mkfs.ext4 /tmp.bin

Check the file-system has been created.

	$ tune2fs -l /tmpfile.bin 
	tune2fs 1.42.11 (09-Jul-2014)
	Filesystem volume name:   <none>
	Last mounted on:          <none>
	...
	Filesystem magic number:  0xEF53
	Filesystem revision #:    1 (dynamic)
	Journal inode:            8
	First orphan inode:       14
	Default directory hash:   half_md4
	Journal backup:           inode blocks

Mount your new file-system with the nodev, nosuid and nonexec flags

	$ sudo mount -o loop,rw,nodev,nosuid,noexec /tmp.bin /tmp

Check the mount, /tmp should now have the file-system of */dev/loop0*

	$ df -h /tmp
	Filesystem      Size  Used Avail Use% Mounted on
	/dev/loop0      976M  1.4M  908M   1% /tmp
	

We will set the *restricted deletion flag* or *sticky bit* on the file-system.
This will prevents unprivileged users from removing or renaming a file in the
directory unless they own the file or the directory.

	$ sudo chmod 1777 /tmp

Confirm the permissions

	$ ls -ld /tmp
	drwxrwxrwt 3 root root 4096 Aug  3 10:53 /tmp

Now we will *bind* the /var/tmp mount to /tmp.
This will ensure that will are using the same file-system for /tmp and /var/tmp

	$ sudo mount -o rw,noexec,nosuid,nodev,bind /tmp /var/tmp

Confirm the bind and mount, /var/tmp adn /tmp should have the same file-system of */dev/loop0*

	$ df -h  /var/tmp /tmp
	Filesystem      Size  Used Avail Use% Mounted on
	/dev/loop0      976M  1.4M  908M   1% /var/tmp
	/dev/loop0      976M  1.4M  908M   1% /tmp

You can also confirm all of this via the *mount* command

	$ mount |egrep '/tmp|/var/tmp|/dev/shm'
	tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,noexec,relatime)
	/tmp.bin on /tmp type ext4 (rw,nosuid,nodev,noexec,relatime,data=ordered)
	/tmp.bin on /var/tmp type ext4 (rw,nosuid,nodev,noexec,relatime,data=ordered)
	
	
Update the /etc/fstab, else we will lose all of the changes on reboot

	$ cat << EOF | sudo tee -a /etc/fstab
	/tmpfile.bin   /tmp     ext4  rw,noexec,nosuid,nodev       0 0
	/tmp           /var/tmp none  rw,noexec,nosuid,nodev,bind  0 0
	tmpfs          /dev/shm tmpfs defaults,nodev,nosuid,noexec 0 0
	EOF

Make sure /etc/fstab updated

	$ grep tmp /etc/fstab
	/tmpfile.bin   /tmp     ext4  rw,noexec,nosuid,nodev       0 0
	/tmp           /var/tmp none  rw,noexec,nosuid,nodev,bind  0 0
	tmpfs          /dev/shm tmpfs defaults,nodev,nosuid,noexec 0 0


Testing
---------

Now we have completed all this work, lets find out the effects.
We will start by taking a copy of the ls binary into /tmp and trying to execute /bin/ls.

	$ cp /bin/ls /tmp/ls
	$ ./ls
	zsh: permission denied: ./ls

Validate that /tmp/ls has execution rights

	$ ls -l /tmp/ls
	-rwxr-xr-x 1 dannyla dannyla 109992 Aug  3 10:57 /tmp/l


All done!
-----------
