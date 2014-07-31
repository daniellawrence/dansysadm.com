Title: Sending data to graphite from python
Date: 2012-02-01 10:20
Category: blog
Tags: graphite, python

This is an overview about how you can send data to a remote graphite server using python.

This overview will take advantage of an opensource python module called [graphitesend](https://github.com/daniellawrence/graphitesend), that you can find on [github.com](https://github.com/daniellawrence/graphitesend) or via [pypi.python.org](https://pypi.python.org/pypi/graphitesend/).


The goal
----------------

We are going to take the [example that comes with the graphite install about sending the load avg.](https://github.com/graphite-project/graphite-web/blob/master/examples/example-client.py) from a linux.

The example code does not use any any extra python modules, which is nice when your getting started. However pulling in the [graphitesend](https://github.com/daniellawrence/graphitesend) library will make it all a bit easier.


Installing the graphitesend module
------------------------------------
You can install the module via pip and [pypi.python.org](https://pypi.python.org/pypi/graphitesend/)

```sh
$ sudo pip install graphitesend
```

You can also install from the source from [github.com](https://github.com/daniellawrence/graphitesend).

```sh
$ git clone https://github.com/daniellawrence/graphitesend
$ cd graphitesend
$ sudo python ./setup.py install
```

Gather the data
-----------------

Before you can send the data to graphite, you need to be able to gather the data in the first place.
Open up the /proc/loadavg file and grab the first 3 columns. 

```python
>>> (la1, la5, la15 ) = open('/proc/loadavg').read().strip().split()[:3]
>>> print la1, la5, la15
0.35 0.34 0.30
```

Now we have 3 variables that contain the loadavg data for 1,5 and 15 minute avg.

Sending the data to graphite
-----------------------------

In this example we are going to assume that your graphite server is called **graphite** and its listening on port **2003**.

```python
>>> import graphitesend
>>> g = graphitesend.init(group='loadavg_')
'sent 51 long message: systems.<hostname>.loadavg_1min 0.470000 1365154443\n '
>>> g.send('5min', la5)
'sent 51 long message: systems.<hostname>.loadavg_5min 0.280000 1365154469\n '
>>> g.send('15min', la15)
'sent 52 long message: systems.<hostname>.loadavg_15min 0.280000 1365154474\n '
```

If your graphite instance is not called **graphite** or maybe not listening on port **2003**. 
Then you can just change it in the init().

```python
>>> graphitesend.init(graphite_server='graphite.prod.example.com', graphite_port=1234)
```

Now throw loadavg. gather into a while loop, like the example from the graphite project.

We are also using the suffix keyword arg in the init() function to suffix all the metric names with 'min'.

The **send_dict()** lets us send a dict of data points as long as they are simple key,value pairs.

```python
#!/usr/bin/env python
import time
import graphitesend

g = graphitesend.init(group='loadavg_', suffix='min')
while True:
    (la1, la5, la15 ) = open('/proc/loadavg').read().strip().split()[:3]
    print g.send_dict( {'1': la1, '5': la5, '15': la15} )
    time.sleep(delay)
```

This will send 3 metrics to graphite with the following names.

```sh
systems.<hostname>.loadavg_1min
systems.<hostname>.loadavg_15min
systems.<hostname>.loadavg_5min
```



More advanced example
---------------------

Lets take a file with some alot more data the /proc/loadavg, we can take the /proc/net/netstat file.

```sh
cat /proc/net/netstat
TcpExt: SyncookiesSent SyncookiesRecv SyncookiesFailed EmbryonicRsts PruneCalled RcvPruned OfoPruned OutOfWindowIcmps LockDroppedIcmps ArpFilter TW TWRecycled TWKilled PAWSPassive PAWSActive PAWSEstab DelayedACKs DelayedACKLocked DelayedACKLost ListenOverflows ListenDrops TCPPrequeued TCPDirectCopyFromBacklog TCPDirectCopyFromPrequeue TCPPrequeueDropped TCPHPHits TCPHPHitsToUser TCPPureAcks TCPHPAcks TCPRenoRecovery TCPSackRecovery TCPSACKReneging TCPFACKReorder TCPSACKReorder TCPRenoReorder TCPTSReorder TCPFullUndo TCPPartialUndo TCPDSACKUndo TCPLossUndo TCPLostRetransmit TCPRenoFailures TCPSackFailures TCPLossFailures TCPFastRetrans TCPForwardRetrans TCPSlowStartRetrans TCPTimeouts TCPRenoRecoveryFail TCPSackRecoveryFail TCPSchedulerFailed TCPRcvCollapsed TCPDSACKOldSent TCPDSACKOfoSent TCPDSACKRecv TCPDSACKOfoRecv TCPAbortOnSyn TCPAbortOnData TCPAbortOnClose TCPAbortOnMemory TCPAbortOnTimeout TCPAbortOnLinger TCPAbortFailed TCPMemoryPressures TCPSACKDiscard TCPDSACKIgnoredOld TCPDSACKIgnoredNoUndo TCPSpuriousRTOs TCPMD5NotFound TCPMD5Unexpected TCPSackShifted TCPSackMerged TCPSackShiftFallback TCPBacklogDrop TCPMinTTLDrop TCPDeferAcceptDrop IPReversePathFilter TCPTimeWaitOverflow TCPReqQFullDoCookies TCPReqQFullDrop TCPRetransFail TCPRcvCoalesce
TcpExt: 0 0 0 0 0 0 0 0 0 0 241 0 0 0 0 5 1828 1 315 0 0 40838 81 38684878 0 141783 22891 3477 2855 0 2 0 0 0 0 0 0 0 0 57 0 0 0 0 2 0 14 92 0 0 1 0 357 10 47 0 0 53 60 0 4 0 0 0 0 0 6 0 0 0 0 0 32 0 0 16 0 0 0 0 0 41419
IpExt: InNoRoutes InTruncatedPkts InMcastPkts OutMcastPkts InBcastPkts OutBcastPkts InOctets OutOctets InMcastOctets OutMcastOctets InBcastOctets OutBcastOctets
IpExt: 2 0 123 125 172 12 315907582 12881887 62007 60884 44576 548
```

Graph the data then sent it to graphite
----------------------------------------

To turn the above mess into a graphite data, its going to take 9 lines of code!

```python
#!/usr/bin/env python

import graphitesend

lines = open('/proc/net/netstat').readlines()

# Throw away the first colum as it has a text value
tcp_metrics = lines[0].split()[1:]
tcp_values = lines[1].split()[1:]
ip_metrics = lines[2].split()[1:]
ip_values = lines[3].split()[1:]

# zip the 2 lists into 1 list.
data_list = zip( tcp_metrics + ip_metrics, tcp_values + ip_values )

g = graphitesend.init(group='netstat.')
g.send_list( data_list )
```

The above script generates metrics over to graphite that look like this.

```sh
systems.<hostname>.netstat.InBcastPkts 211.000000 1365156828
systems.<hostname>.netstat.OutBcastPkts 12.000000 1365156828
systems.<hostname>.netstat.InOctets 329841457.000000 1365156828
systems.<hostname>.netstat.OutOctets 13542993.000000 1365156828
systems.<hostname>.netstat.InMcastOctets 62007.000000 1365156828
systems.<hostname>.netstat.OutMcastOctets 60884.000000 1365156828
systems.<hostname>.netstat.InBcastOctets 55275.000000 1365156828
```

If you did not like the fact that is mixed case, then you can just throw this in the init() function

```python
g = graphitesend.init(lowercase_metric_names=True)
```

conclusion
----------

Graphite is awesome! 

Hopefully graphitesend lets you take advantage of putting more metrics into graphite.

