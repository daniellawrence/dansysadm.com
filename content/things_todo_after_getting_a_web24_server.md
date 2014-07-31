Title: Things to do after getting a web24 server
Date: 2012-02-01 10:20
Category: blog


web24 is based in Australia which is very nice.
They also have good price for the low end VPS service.

However they like to install a heap of crap on the base ubuntu image.
Here are some tips about making the image clean, secure and SANE.


root password
-------------

Hooray yet another provider that will send you the root password via email.
The first thing you want to do is change this password. 
You will want to change it again after we complete the rest of these activities.

However its way to important too leave to the end of cleaning up the server.

```sh
$ ssh root@vmxnnnnn.hosting24.com.au
# passwd root
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
```

delete web24's back doors into your system
------------------------------------------

Remove the web24 system account from your host.
This is yet another vector that can be used to gain access to your server.


```sh
$ egrep 'vmxtest|web24support' /etc/passwd
vmxtest:x:1000:1000:vmxtest,,,:/home/vmxtest:/bin/bash
web24support:x:0:0::/home/web24support:/bin/sh
```

```sh
# userdel vmxtest 
```

As they use the uid and gid of 0, you can't delete this account vi the normal way as the root.

You need to remove the account from /etc/shadow and /etc/passwd.
```sh
vi /etc/shadow /etc/passwd
```

I am not using any backups provided by web24, so I ripped out the backup account as well

```sh
# userdel backup
```

Create yourself a user
----------------------

You now want to be able to disable the root account from SSHing into your machine.
Before turning off ssh as root, you need to have an account with sudo access to admin your machine.

```sh
# groupad admin
# useradd  -G admin -c "Jim Bob" -m -d /home/jimbob jimbob
# passwd jimbob
```

By adding a group called admin and your account into the admin group you will get full sudo access.

I set a password as well, I'll use this to setup my ssh key and then sudo acccess once on the machine.


Generate & Load you ssh keys
----------------------------

Generate your ssh on your local machine.
```sh
$ ssh-keygen -t rsa
```

Load the public key onto the remote vmxnnnn server.

```sh
$ ssh-copy-id jimbob@vmxnnnnn.hosting24.com.au
```

Test that you can login to the system without providing a password.

```sh
$ ssh jimbob@vmxnnnnn.hosting24.com.au
```

Disalbe password authenication
------------------------------

```sh
sudo vi /etc/ssh/sshd_config

# Change to no to disable tunnelled clear text passwords
PasswordAuthentication no

# Bugger off root access
PermitRootLogin no

:wq
````

restart sshd on the system

```sh
$ sudo service ssh reload
```

You will get this error message trying to connect as root to the machine
```sh
% ssh root@vmnxxxxxx.hosting24.com.au
Permission denied (publickey).
```



Update & upgrade the system
---------------------------

The image provided is 12.04.1 it is a little out of day, time to upgrade.

```sh
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
```

