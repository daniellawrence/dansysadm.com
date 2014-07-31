Title: Should I change my disk scheduler to use NOOP 
Date: 2012-02-01 10:20
Category: blog

I just finished reading a white paper from Redhat (Red Hat Enterprise: Linux 5 IO Tuning Guide) and wondered if I should change my disk scheduler on my laptop that uses an SSD from CFQ( default) to NOOP.

My laptop runs Ubuntu 12.04 with a stripped down kernel and a few additions.

```sh
$ uname -a
Linux xps15z 3.4.1dan #1 SMP Fri Jun 8 12:17:12 EST 2012 x86_64 x86_64 x86_64 GNU/Linux
$ lsb_release -a
Distributor ID: Ubuntu
Description: Ubuntu 12.04 LTS
Release: 12.04
Codename: precise
```

The Schedulers to choose from:
------------------------------
There are several types of schedulers; deadline, anticipatory (as)(n/a), completely fair queueing (cfq), simple FIFO (noop)

*deadline:* As the name suggests, sets a deadline (tunable: read_expire & write_expire) for each IO operation. Once the deadline is reached the request is serviced immediately. This allows for a guarantee a response time for each request at the trade of a guarantee delay for each request.

*completely fair queueing (cfq):*  cfg the default from 2.6.18 onwards,  it works by sending all disk requests from processes to 1 of 64 queues. Then the scheduler will loop over the queues (completely fair) and take n ( tunable: quantum ) operations from a queue and processes them.

*FIFO (noop):* The NOOP scheduler adds all incoming I/O requests into queue and services them in a FIFO fashion. NOOP scheduler also implements request merging, that will group requests together that are accessing a similar area on the disk.

The paper also described NOOP to be best suited for block devices that use flash memory.

How to change a devices scheduler
---------------------------------

If you want to temporary change a single block devices scheduler, echo the name of the scheduler into /sys/block/<block-device>/queue/scheduler 

```sh
$ echo "noop" | sudo tee /sys/block/<block-device>/queue/scheduler
```

The Tests
---------

I used hdparm and a small c program called 'seeker'

*cfq: results*

```sh
$ echo "cfq" | sudo tee /sys/block/sda/queue/scheduler 
$ sudo hdparm -t /dev/sda
/dev/sda:
  Timing buffered disk reads: 1412 MB in 3.00 seconds = 470.50 MB/sec
$ sudo ./seeker /dev/sda
Seeker v2.0, 2007-01-15, http://www.linuxinsight.com/how_fast_is_your_disk.html
Benchmarking /dev/sda [244198MB], wait 30 seconds..............................
Results: 6780 seeks/second, 0.15 ms random access time
```

*noop: results*

```sh
$ echo "noop" | sudo tee /sys/block/sda/queue/scheduler
$ sudo hdparm -t /dev/sda
/dev/sda:
 Timing buffered disk reads: 1420 MB in 3.00 seconds = 472.80 MB/sec
$ sudo ./seeker /dev/sda
Seeker v2.0, 2007-01-15, http://www.linuxinsight.com/how_fast_is_your_disk.html
Benchmarking /dev/sda [244198MB], wait 30 seconds..............................
Results: 6341 seeks/second, 0.16 ms random access time
```

*deadline: results*

```sh
$ echo "deadline" | sudo tee /sys/block/sda/queue/scheduler
deadline
$ sudo hdparm -t /dev/sda
/dev/sda:
Timing buffered disk reads: 1408 MB in 3.00 seconds = 468.70 MB/sec
sudo ./seeker /dev/sda
Seeker v2.0, 2007-01-15, http://www.linuxinsight.com/how_fast_is_your_disk.html
Benchmarking /dev/sda [244198MB], wait 30 seconds..............................
Results: 6620 seeks/second, 0.15 ms random access time
Results (including deadline )
```

Results
--------

```sh
Metric	                    cfq 	noop	dead line
Amount Read in 3 seconds	1418	1420	1408
Read Speed (MB/sec)			470.50	472.80	468.70
Seeks per Second			6780	6341	6620
Seeks time (ms)				0.16	0.15	0.15
```

Overall changing the scheduler to be NOOP over CFQ has a benefit, it might be very small but it is still a benefit. I did order the laptop with an SSD purely for speed, so why not have it go that little faster.

Keeping the settings after a reboot
-----------------------------------

By changing the GRUB_CMDLINE_LINUX_DEFAULT line in the  /etc/default/grub file to the below the settings will be applied every time the system is booted.

```sh
$ sudo vi  /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash elevator=noop"
$ sudo update-grub2
```
