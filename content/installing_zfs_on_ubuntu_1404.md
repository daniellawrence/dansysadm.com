Title: Installing zfs on Ubuntu server 14.04
Date: 2014-07-01 10:20
Category: blog
Tags: ubuntu, zfs

This is a quick guide to get zfs up and running on Ubuntu server, it would also work on Ubuntu desktop.

From Ubuntu server I needed to install python-software-properties, so that i could run add-apt-repository
( You don't need to complete this first step from Ubuntu desktop or Linux mint  )

```sh
$ sudo apt-get install python-software-properties
```

Then install the zfs-native ppa, so that you can download and compile native zfs goodness

```sh
$ sudo add-apt-repository ppa:zfs-native/stable
```

Update your repo list, so it includes the new zfs-native ppa

```sh
$ sudo apt-get update
```

Make sure the the zfs packages in the repo

```sh
$ apt-cache search zfs
```

Install, Note: this might take some time as it compiles the kernel module for your kernel

```sh
$ sudo apt-get install ubuntu-zfs
```

Run the zfs commands to make sure it works

```sh
$ sudo zfs
$ sudo zpool
```

Create a pool

```sh
$ sudo zpool create data /dev/sda
```

Check the mounts

```sh
$ df -h -F zfs
Filesystem      Size  Used Avail Use% Mounted on
data            229G     0  229G   0% /data
```
