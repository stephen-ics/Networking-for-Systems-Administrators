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