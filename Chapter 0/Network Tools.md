# Network Tools
If you look around you'll find innumerable server-side tools for analyzing the network, many of these tools work only on very specific operating systems or have limited utility

## ifconfig, route, and ipconfig
A host's network configuration includes its IP addresses and gateway

On Unix, use the *route* and *ifconfig* commands to view the system's network configuration

Windows systems put all of these in the *ifconfig* command, while different operating systems have different versions of these commands, you can sort out the information you need from any of them

Microsoft has added networking functions to PowerShell, while it is recommended to learn and understand PowerShell, it will not be covered largely in this book

Similarly, some Linux variants have started to move away from ifconfig, but it will be available and supported for some time to come, and it's the command most Unixes use for network configuration

### ifconfig Deep Dive
`ifconfig` (short for "interface configuration) shows details about your computer's network connection, it tells you your **IP address**, **network interface** (Ethernet or Wi-Fi), and whether your connection is up or down

Command:
```sh
ifconfig
```

Sample Output:
```sh
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.100  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::1a2b:3c4d:5e6f:7g8h  prefixlen 64  scopeid 0x20<link>
        ether 00:1a:2b:3c:4d:5e  txqueuelen 1000  (Ethernet)
        RX packets 34567  bytes 4567890 (4.5 MB)
        TX packets 12345  bytes 2345678 (2.3 MB)

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 1000  bytes 123456 (123.4 KB)
        TX packets 1000  bytes 123456 (123.4 KB)
```

Let's dive into a bit more detail on this
Explanation:
- `eth0`: THis is the network interface name (a "port" on your computer that connects to the network)
    - `flags=4163<UP,BROADCAST,RUNNING,MULTICAST>`: UP means the connection is active
    - `inet`: This is the IP address (your computer's unique network ID)
    - `netmask`: This is a bit-mask that defines which part of the Ip is the network and which part is the device, perform a bitwise AND operation on your `inet`!
    - `broadcast`: The address used to send messages to all devices on the network

### route Deep Dive
`route` shows where the network traffic is sent, it tells you where your computer sends data, your gateway, and how data is routed inside your network

Command:
```sh
route -n
```

Sample Output:
```sh
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    100    0        0 eth0
192.168.1.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0
127.0.0.0       0.0.0.0         255.0.0.0       U     100    0        0 lo
```

Will do a deep dive into this too!
- Destination: Where the data is being sent
- Gateway: The "next stop" for sending data
- Genmask: A bit-mask that defines the network size
- Iface (interface): The network interface used (Wi-Fi or Ethernet) such as `eth0`

If your computer isn't connecting to a network, these commands help check if your network interfaces are up and have the correct IP addresses, you can see quickly if a network cable is disconnected or if your Wi-Fi isn't working properly

## grep and findstr
Many network-related commands product far more output than you want to read

*grep* (Unix) and *findstr* (Windows) commands let you search for a specific string within a pile of output

I do encourage Windows sysadmins to install one of the many versions of *grep* on your systems, as it's far more flexible than *findstr*

`|` is the pipe operator, it takes output from the left command and passes it as input to the right command

### grep
*grep*: A command-line utility available on Unix/Linux systems, it searches through files or command output for lines that match a specific **pattern** (using simple text matching or regular expressions)

For example, say you only want to see lines containing the word "inet", which shows IP Addresses, you can do

```sh
ifconfig | grep inet
```

By default, grep is case sensitive, you can use the `-i` flag to ignore case

The exact same can be done with *findstr*, though in findstr, the `/I` option makes the search case-insensitive

```sh
ipconfig | findstr /I IPv4
```

*grep* allows for complex regular expressions to match patterns, *findstr* supports basic regex, though it's less powerful an less intuitive than grep

## netstat
The *netstat* command displays a system's established network connections, what connections the system can receive, and network statistics

Some operating systems, like Solaris, use *netstat* to show the routing table, many operating systems offer most of their visibility into their network using *netstat*

To clarify, *ifconfig* focuses on individual network interfaces while *netstat* looks at a broader view of network activity such as TCP and UDP sockets, and details like the local and foreign addresses, port numbers, and the state of each connection

You should use *ifconfig* to verify that your interface is up, has an IP address, and the correct configuration, and you should use *netstat* when monitoring network traffic, identifying which connections are active, which ports are open, and that your system is sending data through the correct gateway

*route* is specifically designed to display the routing table while netstat is a more comprehensive tool though `netstat -rn` displays the same routing table information as `route`

You can use the `-rn` flag to display numeric addresses and avoid DNS lookups

Command:
```sh
netstat
```

Sample Output:
```sh
Active Internet connections
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
tcp        0      0  192.168.1.100:443      93.184.216.34:52345    ESTABLISHED
tcp        0      0  192.168.1.100:22       192.168.1.50:57632     ESTABLISHED
tcp6       0      0  ::1:631                ::1:51234             LISTEN
udp        0      0  192.168.1.100:68      *:*                    
```
It displays the protocol used by the connection, for example `tcp` means a TCP connection over IPv4

- `Recv-Q`: The amount of data in bytes that has been received and queued waiting to be read
- `Send-Q`: The amount of data (in bytes) waiting in the send queue to be transmitted
- `Local Address`: The address on your machine
- `Foreign Address`: The address of the remote machine that your computer is connected to
`ESTABLISHED`: This means there is a fully active connection where data can flow

## lsof
The Unix command *lsof* stands for *list Open Files*, it shows you a list of all files that are open by active processes on your system

Since in Unix-like systems "everything is a file" (files, directories, network sockets, pipes, devices, etc), lsof is a versatile for diagnosing system behaviour and debugging

Each open file is associated with a process, lsof shows which process (PID) has the file open, what user owns that process, and how the file is being used (read, write, etc)

*lsof* interfaces with the kernel to retrieve a snapshot of open file descriptors, every process maintains a table of file descriptors, which lsof reads to present the information

Command:
```sh
lsof
```

Sample Output:
```sh
COMMAND     PID    USER   FD      TYPE     DEVICE    SIZE/OFF     NODE NAME
bash       1234   alice  cwd     DIR      8,1       4096         56789 /home/alice
bash       1234   alice  txt     REG      8,1       1037528      123456 /bin/bash
chrome     2345   alice  mem     REG      8,1       2345678      234567 /usr/lib/chrome/libfoo.so
chrome     2345   alice  123u    IPv4     0t0        0            34567 TCP 192.168.1.100:56789->93.184.216.34:http (ESTABLISHED)
```

- `COMMAND`: The name of the command (or process) that opened the file
- `PID`: The process ID which uniquely identifies the process, it's a unique number assigned by the operating system to each process when it starts
- `USER`: The owner under which the process is running
- `FD`: The descriptor number or type of file, sometimes appended with a letter
        - *cwd*: Current working directory
        - *txt*: Program text
        - *mem*: Memory-mapped file
- `TYPE`: The type of file
        - *DIR*: Directory
        - *REG*: Regular file
        - *IPv4/IPv6*: Network sockets
        - *CHR*: Character device

- `DEVICE`: The device number or filesystem identifier
- `SIZE/OFF`: For regular files, this shows the size in bytes, for other types it might show the current file offset
- `NODE`: The inode number of the file system, useful for low-level troubleshooting
- `NAME`: The full path to the file, for network sockets the network endpoint

`-c [name]`: Lists only files opened by processes whose command begins with set name
`-p [pid]`: Lists files opened by the process ID
`-i :80`: Shows processes using port 80 (HTTP)

A "process" is a running program, when you execute a program, the operating system creates a process, each process is an individual instance of that program running in memory

## tcpdump and Wireshark
The *tcpdump* command displays traffic to and from a server, even when the server rejects that traffic, it is the fastest way to view network activity

For more complicated analysis, an option is Wireshark

Many operating systems include their own traffic sniffing program such as *snoop* on Solaris and Microsoft's Network Monitor and Message Analyzer

There's nothing wrong with these tools, but expertise in them doesn't carry over into other operating systems

Most of them use syntax copied from *tcpdump*, however an understanding of *tcpdump* makes using platform-specific tools much easier

## netcat
The *netcat* program lets you listen to the network on a specific port, and lets you send arbitrary network traffic

It's a great way to verify that the network will let you send and receive traffic without configuring a specific daemon or service for that purpose, not all operating systems include *netcat* by default

To use netcat as a listener:
```sh
nc -l -p 1234
```
`-l` tells the netcat to listen for an incoming connection
`-p 1234` specifies the port

The terminal waits for someone to connect to port 1234, any data sent by the connecting client will be displayed in this terminal

Netcat can be used to transfer files, for example
```sh
nc -l -p 1234 > received_file.txt
```
This command listens on port 1234 and redirects incoming data to received_file.txt