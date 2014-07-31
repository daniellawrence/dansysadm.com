Title: Solaris netstat tcp protocol statistics explained
Date: 2012-02-01 10:20
Category: blog

The netstat command on Solaris is a fantastic gateway to all your TCP statistics of your system.

You would have used the netstat -an command in the past.
However the netstat -s -P tcp can give you up-to-date statistics about what your TCP stack has been up to...

```sh
 $ nestat -s -P tcp
 TCP
    tcpRtoAlgorithm     =     4     tcpRtoMin           =   400
    tcpRtoMax           = 60000     tcpMaxConn          =    -1
    tcpActiveOpens      =7624114    tcpPassiveOpens     =7084624
    tcpAttemptFails     =1896763    tcpEstabResets      =193326
    tcpCurrEstab        =    74     tcpOutSegs          =21843389688
    tcpOutDataSegs      =3328351751 tcpOutDataBytes     =3235412917
    tcpRetransSegs      =41967918   tcpRetransBytes     =2212890976
    tcpOutAck           =853704065  tcpOutAckDelayed    =247961090
    tcpOutUrg           =     1     tcpOutWinUpdate     =477772
    tcpOutWinProbe      = 12412     tcpOutControl       =33045410
    tcpOutRsts          =5285917    tcpOutFastRetrans   =  8210
    tcpInSegs           =11491393189
    tcpInAckSegs        =1158661729 tcpInAckBytes       =1102332654
    tcpInDupAck         =142544351  tcpInAckUnsent      =     0
    tcpInInorderSegs    =1884725886 tcpInInorderBytes   =1286627563
    tcpInUnorderSegs    =1912668    tcpInUnorderBytes   =2409325298
    tcpInDupSegs        =34780066   tcpInDupBytes       =1415828491
    tcpInPartDupSegs    =  3626     tcpInPartDupBytes   =1693311
    tcpInPastWinSegs    =2269167    tcpInPastWinBytes   =126796354
    tcpInWinProbe       =  1057     tcpInWinUpdate      = 11758
    tcpInClosed         =3426232    tcpRttNoUpdate      =445365299
    tcpRttUpdate        =706469685  tcpTimRetrans       =1305409
    tcpTimRetransDrop   =  2328     tcpTimKeepalive     = 75510
    tcpTimKeepaliveProbe= 28677     tcpTimKeepaliveDrop =   193
    tcpListenDrop       =    11     tcpListenDropQ0     =     0
    tcpHalfOpenDrop     =     0     tcpOutSackRetrans   =40485027
```
As you can see this is a head of data that you now have at your finger tips, however the man pages have no explanation what each of the items are in this list.

```sh
 $ man netstat
  -s
         Show per-protocol statistics.
```

The source code has a little bit more data (http://src.opensolaris.org/source/xref/onnv/onnv-gate/usr/src/uts/common/inet/mib2.h#mib2_tcp) to get to started...

Sockets
-------

__tcpRtoAlgorithm:__ algorithm used for transmit timeout value

__tcpRtoMin:__ minimum retransmit timeout (ms)

__tcpRtoMax:__ maximum retransmit timeout (ms)

__tcpMaxConn:__ maximum # of connections supported

__tcpActiveOpens:__ # of direct transitions CLOSED -> SYN-SENT

__tcpPassiveOpens:__ # of direct transitions LISTEN -> SYN-RCVD

__tcpAttemptFails:__ # of direct SIN-SENT/RCVD -> CLOSED/LISTEN

__tcpEstabResets:__ # of direct ESTABLISHED/CLOSE-WAIT -> CLOSED

__tcpCurrEstab:__ # of connections ESTABLISHED or CLOSE-WAIT

__tcpInSegs:__ total # of segments recv'd

__tcpOutSegs:__ total # of segments sent

__tcpRetransSegs:__ total # of segments retransmitted

__tcpConnTableSize:__ Size of tcpConnEntry_t in ip

__tcpOutRsts:__ # of segments sent with RST flag

Sender
------

__tcpOutDataSegs:__ total # of data segments sent

__tcpOutDataBytes:__ total # of bytes in data segments sent

__tcpRetransBytes:__ total # of bytes in segments retransmitted

__tcpOutAck:__ total # of acks sent

__tcpOutAckDelayed:__ total # of delayed acks sent

__tcpOutUrg:__ total # of segments sent with the urg flag on

__tcpOutWinUpdate:__ total # of window updates sent

__tcpOutWinProbe:__ total # of zero window probes sent

__tcpOutControl:__ total # of control segments sent (syn, fin, rst)

__tcpOutFastRetrans:__ total # of segments sent due to "fast retransmit"


Receiver
---------

__tcpInAckSegs:__ total # of ack segments received

__tcpInAckBytes:__ total # of bytes acked

__tcpInDupAck:__ total # of duplicate acks

__tcpInAckUnsent:__ total # of acks acking unsent data

__tcpInDataInorderSegs:__ total # of data segments received in order

__tcpInDataInorderBytes:__ total # of data bytes received in order

__tcpInDataUnorderSegs:__ total # of data segments received out-of-order

__tcpInDataUnorderBytes:__ total # of data bytes received out-of-order

__tcpInDataDupSegs:__ total # of complete duplicate data segments received

__tcpInDataDupBytes:__ total # of bytes in the complete duplicate data segments received

__tcpInDataPartDupSegs:__ total # of partial duplicate data segments received

__tcpInDataPartDupBytes:__ total # of bytes in the partial duplicate data segments received

__tcpInDataPastWinSegs:__ total # of data segments received past the window

__tcpInDataPastWinBytes:__ total # of data bytes received part the window

__tcpInWinProbe:__ total # of zero window probes received

__tcpInWinUpdate:__ total # of window updates received

__tcpInClosed:__ total # of data segments received after the connection has closed

Others
-------

__tcpRttNoUpdate:__ total # of failed attempts to update the rtt estimate

__tcpRttUpdate:__ total # of successful attempts to update the rtt estimate

__tcpTimRetrans:__ total # of retransmit timeouts

__tcpTimRetransDrop:__ total # of retransmit timeouts dropping the connection

__tcpTimKeepalive:__ total # of keepalive timeouts

__tcpTimKeepaliveProbe:__ total # of keepalive timeouts sending a probe

__tcpTimKeepaliveDrop:__ total # of keepalive timeouts dropping the connection

__tcpListenDrop:__ total # of connections refused due to backlog full on listen

__tcpListenDropQ0:__ total # of connections refused due to half-open queue (q0) full

__tcpHalfOpenDrop:__ total # of connections dropped from a full half-open queue (q0)

__tcpOutSackRetransSegs:__ total # of retransmitted segments by SACK retransmission

__tcp6ConnTableSize:__ Size of tcp6ConnEntry_t


Getting Netstat output into Graphite
-------------------------------------

What would blog post able statistics be if you can't graph them into graphite using python... ?

```python
from sysadm import graph
data={}
for line in os.popen("/usr/bin/netstat -s -P tcp").readlines():
    netstat = line.replace('=','').replace('TCP','').split()
    if len(netstat) == 2:
        (k,v) = netstat
        data["tcp.%s" % k] = float(v)
        continue
    if len(netstat) == 4:
        (k,v,k2,v2) = netstat
        data["tcp.%s" % k2] = float(v2)
        data["tcp.%s" % k] = float(v)
        continue
graph.logBulkData(data)
```
