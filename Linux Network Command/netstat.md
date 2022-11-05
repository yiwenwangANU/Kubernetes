## `netstat` ([ Link](https://www.howtogeek.com/513003/how-to-use-netstat-on-linux/) )
`netstat -apn Used to check the ip address and port for certain process`\
The -a (all) option makes netstat show all the connected and waiting sockets. 
```sh
netstat -a | less
```
The `Active Internet` section lists the network connections that are (or will be) established to external devices.\
The `UNIX domain` section lists the connections that have been established within your computer between different applications, processes, and elements of the operating system.
```sh
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address     Foreign Address State 
tcp        0      0 localhost:domain  0.0.0.0:*       LISTEN 
tcp        0      0 0.0.0.0:ssh       0.0.0.0:*       LISTEN 
tcp        0      0 localhost:ipp     0.0.0.0:*       LISTEN 
tcp        0      0 localhost:smtp    0.0.0.0:*       LISTEN 
tcp6       0      0 [::]:ssh          [::]:*          LISTEN 
tcp6       0      0 ip6-localhost:ipp [::]:*          LISTEN 
.
.
.
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags   Type     State       I-Node  Path
unix  24     [ ]     DGRAM                12831   /run/systemd/journal/dev-log
unix  2      [ ACC ] STREAM    LISTENING  24747   @/tmp/dbus-zH6clYmvw8
unix  2      [ ]     DGRAM                26372   /run/user/1000/systemd/notify
unix  2      [ ]     DGRAM                23382   /run/user/121/systemd/notify
unix  2      [ ACC ] SEQPACKET LISTENING  12839   /run/udev/control
```
Use -n (numeric) option to see the IP addresses instead of their resolved domain and hostnames.
```sh
netstat -an | less
```

Use the -t (TCP) option to restrict the display to only show TCP sockets or -u for UDP, -x for unix
```sh
netstat -at | less
```

Use the -l (listening) option, to see the sockets that are in the listening or waiting state.
```sh
netstat -lt | less
```

Use -p to see the process using the socket.
```sh
netstat -apt
```
```sh
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address       Foreign Address   State    PID/Program name 
tcp      0        0 localhost:domain    0.0.0.0:*         LISTEN   6927/systemd-resolv 
tcp      0        0 0.0.0.0:ssh         0.0.0.0:*         LISTEN   751/sshd 
tcp      0        0 localhost:ipp       0.0.0.0:*         LISTEN   7687/cupsd 
tcp      0        0 localhost:smtp      0.0.0.0:*         LISTEN   1176/master 
tcp6     0        0 [::]:ssh            [::]:*            LISTEN   751/sshd 
tcp6     0        0 ip6-localhost:ipp   [::]:*            LISTEN   7687/cupsd 
tcp6     0        0 ip6-localhost:smtp  [::]:*            LISTEN   1176/master
```


Use -r (route) option displays the kernel routing table.
```sh
sudo netstat -r
```
Use -i (interfaces) option will display a table of the network interfaces that netstat can discover.
```sh
sudo netstat -i
```
Interface:
* `lo` interface enables processes to intercommunicate within the computer using networking protocols
* `enp0s3` interface is the network interface to the outside world
```sh
Kernel Interface table
Iface     MTU   RX-OK  RX-ERR  RX-DRP  RX-OVR    TX-OK   TX-ERR   TX-DRP   TX-OVR Flg
enp0s3   1500 4520671       0       0  0       4779773        0        0        0 BMRU
lo      65536   30175       0       0  0         30175        0        0        0 LRU
```
To see statistics for a protocol, use the -s (statistics) option and pass in the -t (TCP), -u (UDP), or -x (UNIX) options. 
```sh
netstat -st | less
```
