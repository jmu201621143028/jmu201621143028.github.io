---
title: 使用Netlink监测linux网络状态
date: 2023-02-24 09:44:43
tags:
---
## 参考链接 
 [monitoring-linux-networking-state-using-netlink](https://olegkutkov.me/2018/02/14/monitoring-linux-networking-state-using-netlink/)  

## python重写
```python
import struct
import socket
import os

nlmsghdr_format = "=IHHII"
ifinfomsg_format = "=BBHiII"
ifaddrmsg_format = "=BBBBI"
rtattr_format = "=HH"
nlmsghdr_size = 16
ifinfomsg_size = 16
ifaddrmsg_size = 8
rtattr_size = 4

RTM_NEWROUTE = 24
RTM_DELROUTE = 25
IFA_LOCAL = 2
IFLA_IFNAME = 3
IFLA_MAX = 51
RTMGRP_LINK = 1
RTMGRP_IPV4_IFADDR = 0x10
RTMGRP_IPV4_ROUTE = 0x40
IFF_UP = 0x1
IFF_RUNNING = 0x40
RTM_DELADDR = 21
RTM_DELLINK = 17
RTM_NEWLINK = 16
RTM_NEWADDR = 20


sock = socket.socket(socket.AF_NETLINK, socket.SOCK_RAW, socket.NETLINK_ROUTE)
sock.bind((os.getpid(), RTMGRP_LINK | RTMGRP_IPV4_IFADDR | RTMGRP_IPV4_ROUTE))

def parseRtattr(rta, msg_len, ifla_rta_ix):
    RTA_ALIGNTO = 4
    RTA_ALIGN = lambda Length : (Length + RTA_ALIGNTO - 1) & ~(RTA_ALIGNTO - 1)
    tb = [None for _ in range(IFLA_MAX + 1)]
    point, size = 0, len(rta)
    rta_len, rta_type = struct.unpack(rtattr_format, rta[point : rtattr_size])
    while msg_len >= rtattr_size and rtattr_size <= rta_len <= msg_len and point + rtattr_size <= size:
        rta_len, rta_type = struct.unpack(rtattr_format, rta[point : point + rtattr_size])
        print("point is {}".format(point))
        if rta_type <= IFLA_MAX:
            print("rta_type is {}, rta_len is {}, rta is {}".format(rta_type, rta_len, ifla_rta_ix + point))
            tb[rta_type] = ifla_rta_ix + point + rtattr_size
        point += RTA_ALIGN(rta_len)
        msg_len -= RTA_ALIGN(rta_len)

    return tb


while True:
    data = sock.recv(8192)
    nlmsghdr = data[0:nlmsghdr_size]

    msg_len, msg_type, flags, seq, pid = struct.unpack(nlmsghdr_format, nlmsghdr)
    if msg_type == RTM_NEWROUTE or msg_type == RTM_DELROUTE:
        print("Routing table was changed")
        continue
    # print("msg_len={}\t msg_type={}\t flags={}\t seq={}\t pid={}".format(msg_len, msg_type, flags, seq, pid))
    # print(len(data))
    # parser struct ifinfomsg
    # unsigned really is a shorthand for unsigned int, and so defined in standard C
    # https://stackoverflow.com/questions/1171839/what-is-the-unsigned-datatype
    ifinfomsg = data[nlmsghdr_size : nlmsghdr_size + ifinfomsg_size]
    ifi_family, __ifi_pad, ifi_type, ifi_index, ifi_flags, ifi_change = struct.unpack(ifinfomsg_format, ifinfomsg)
    # print("ifi_family={}\t__ifi_pad={}\tifi_type={}\tifi_index={}\tifi_flags={}\tifi_change={}".format(ifi_family, __ifi_pad, ifi_type, ifi_index, ifi_flags, ifi_change))
    
    ifla_rta = data[nlmsghdr_size + ifinfomsg_size :]
    tb = parseRtattr(ifla_rta, msg_len, nlmsghdr_size + ifinfomsg_size)

    if tb[IFLA_IFNAME]:
        # may 
        for ix in range(tb[IFLA_IFNAME], len(data)):
            if data[ix] == '\x00': break
        ifName_format = "=" + "c" * (ix - tb[IFLA_IFNAME])
        ifName = ''.join(struct.unpack(ifName_format, data[tb[IFLA_IFNAME] : ix]))
        print("ifName is {}".format(ifName))
    if ifi_flags & IFF_UP:
        ifUpp = "UP"
    else:
        ifUpp = "DOWN"
    print(ifUpp)
    if ifi_flags & IFF_RUNNING:
        ifRunn = "RUNNING"
    else:
        ifRunn = "NOT RUNNING"
    print(ifRunn)
    ifaddrmsg = data[nlmsghdr_size : nlmsghdr_size + ifaddrmsg_size]
    ifa_family, ifa_prefixlen, ifa_flags, ifa_scope, ifa_index = struct.unpack(ifaddrmsg_format, ifaddrmsg)
    ifa = data[nlmsghdr_size + ifaddrmsg_size :]
    tba = parseRtattr(ifa, msg_len, nlmsghdr_size + ifaddrmsg_size)
    if tba[IFA_LOCAL]:
        for i in data[tba[IFA_LOCAL] : tba[IFA_LOCAL] + 4]:
            print(ord(i))
        ifAddress = '.'.join(str(i) for i in struct.unpack('=BBBB', data[tba[IFA_LOCAL] : tba[IFA_LOCAL] + 4 ]))
    if msg_type == RTM_DELADDR:
        print("Interface {}: address was removed".format(ifName))
        pass
    elif msg_type == RTM_DELLINK:
        print("Network interface {} was removed".format(ifName))
        pass
    elif msg_type == RTM_NEWLINK:
        print("New network interface {}, state: {} {}".format(ifName, ifUpp, ifRunn))
        pass
    elif msg_type == RTM_NEWADDR:
        print("Interface {}: new address was assigned: {}".format(ifName, ifAddress))
        pass

close(sock)
```
## 
