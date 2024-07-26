---
title: Docker Network
tags:
  - Interview
category:
  - Docker
author: bsyonline
lede: 没有摘要
date: 2020-01-21 11:09:23
thumbnail:
---


Linux 内核支持 6 中命名空间：UTS，User，PID，IPC，Mount，Network 。

docker 默认创建有 3 种网络，使用命令可以查看。
```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
c9277adb7c8b        bridge              bridge              local
705c67c36bba        host                host                local
fd36f2cec0f3        none                null                local
```

当我们启动一个容器，默认使用 bridge 网络。

```
$ docker run --name busybox1 -it --rm busybox:latest
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:13 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:998 (998.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

进入容器使用 ifconfig 可以看到 eth0 是 172.17 网段。这种网络是 docker 使用了一个虚拟网络设备，将容器内部网络和宿主机网络连接起来，从而可以实现容器和宿主机之间通信，并且可以将宿主机当作 dns 服务器和外部通信。
```
/ # nslookup -type=A www.baidu.com
Server:         192.168.65.1
Address:        192.168.65.1:53

Non-authoritative answer:
www.baidu.com   canonical name = www.a.shifen.com
Name:   www.a.shifen.com
Address: 61.135.169.125
Name:   www.a.shifen.com
Address: 61.135.169.121
```

当然可以指定 dns 服务器。

```
$ docker run -it --rm --dns 8.8.8.8 --name busybox1 busybox
/ # nslookup -type=A www.baidu.com
Server:         8.8.8.8
Address:        8.8.8.8:53

Non-authoritative answer:
www.baidu.com   canonical name = www.a.shifen.com
Name:   www.a.shifen.com
Address: 61.135.169.121
Name:   www.a.shifen.com
Address: 61.135.169.125
```

如果我们使用 - -network 指定容器启动的网络为 bridge ，效果是一样的。

如果我们指定使用 none 网络，那么就只有 lo 网络，是一种封闭式容器。
```
$ docker run --name busybox1 -it --rm --network none busybox:latest
/ # ifconfig                                                       
lo        Link encap:Local Loopback                                
          inet addr:127.0.0.1  Mask:255.0.0.0                      
          UP LOOPBACK RUNNING  MTU:65536  Metric:1                 
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0       
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0     
          collisions:0 txqueuelen:1                                
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)          
```

我们还可以指定 network 为 host 。
```
$ docker run -it --rm --network host --name busybox1 busybox               
/ # ifconfig                                                               
docker0   Link encap:Ethernet  HWaddr 02:42:98:75:6B:44                    
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0     
          inet6 addr: fe80::42:98ff:fe75:6b44/64 Scope:Link                
          UP BROADCAST MULTICAST  MTU:1500  Metric:1                       
          RX packets:77548 errors:0 dropped:0 overruns:0 frame:0           
          TX packets:174258 errors:0 dropped:0 overruns:0 carrier:0        
          collisions:0 txqueuelen:0                                        
          RX bytes:3130282 (2.9 MiB)  TX bytes:250653674 (239.0 MiB)       
                                                                           		  
eth0      Link encap:Ethernet  HWaddr 02:50:00:00:00:01                    
          inet addr:192.168.65.3  Bcast:192.168.65.15  Mask:255.255.255.240
          inet6 addr: fe80::50:ff:fe00:1/64 Scope:Link                     
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1               
          RX packets:174978 errors:0 dropped:0 overruns:0 frame:0          
          TX packets:78102 errors:0 dropped:0 overruns:0 carrier:0         
          collisions:0 txqueuelen:1000                                     
          RX bytes:251511916 (239.8 MiB)  TX bytes:4265623 (4.0 MiB)       
                                                                           
hvint0    Link encap:Ethernet  HWaddr 00:15:5D:00:67:17                    
          inet addr:10.0.75.2  Bcast:0.0.0.0  Mask:255.255.255.240         
          inet6 addr: fe80::215:5dff:fe00:6717/64 Scope:Link               
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1               
          RX packets:57690 errors:0 dropped:1 overruns:0 frame:0           
          TX packets:29914 errors:0 dropped:0 overruns:0 carrier:0         
          collisions:0 txqueuelen:1000                                     
          RX bytes:7303259 (6.9 MiB)  TX bytes:2630900 (2.5 MiB)           
                                                                           
lo        Link encap:Local Loopback                                        
          inet addr:127.0.0.1  Mask:255.0.0.0                              
          inet6 addr: ::1/128 Scope:Host                                   
          UP LOOPBACK RUNNING  MTU:65536  Metric:1                         
          RX packets:18 errors:0 dropped:0 overruns:0 frame:0              
          TX packets:18 errors:0 dropped:0 overruns:0 carrier:0            
          collisions:0 txqueuelen:1                                        
          RX bytes:1484 (1.4 KiB)  TX bytes:1484 (1.4 KiB)         
```

可以看到容器的 IP 地址就是宿主机的 IP 地址，说明容器共享了宿主机的 network 命名空间。

当我们启动多个容器的时候，如果我们使用 bridge ，多个容器的 ip 地址是不同的。
```
$ docker run -it --rm --network bridge --name busybox1 busybox
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:688 (688.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

```
$ docker run -it --rm --network bridge --name busybox2 busybox
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:688 (688.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

如果我们修改一下 busybox2 的 network 为 --network container:busybox1 。
```
$ docker run -it --rm --network container:busybox1 --name busybox2 busybox   
/ # ifconfig                                                                 
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02                      
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0       
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1                 
          RX packets:13 errors:0 dropped:0 overruns:0 frame:0                
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0               
          collisions:0 txqueuelen:0                                          
          RX bytes:1038 (1.0 KiB)  TX bytes:0 (0.0 B)                        
                                                                             
lo        Link encap:Local Loopback                                          
          inet addr:127.0.0.1  Mask:255.0.0.0                                
          UP LOOPBACK RUNNING  MTU:65536  Metric:1                           
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0                 
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0               
          collisions:0 txqueuelen:1                                          
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)    
```
这时，两个容器的 IP 是相同的，这种网络模式叫做联盟式网络。联盟式网络的容器共享同一个 network 命名空间，可以彼此之间进行通信，类似同一个主机上的两个进程。

以上就是 docker 支持的 4 种内置的网络模式。
除此之外，docker 还可以创建自定义的网络。通过 docker info 可以查看 docker 支持的网络类型。
```
$ docker info
Client:
 Debug Mode: false

Server:
 ...
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
  ...
```

通过插件，可以支持多种网络。使用命令可创建自定义网络。
```
$ docker network create --driver bridge mynet
```

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
c9277adb7c8b        bridge              bridge              local
705c67c36bba        host                host                local
798d854b9d67        mynet               bridge              local
fd36f2cec0f3        none                null                local
```
我们自己创建的 bridge 网络和默认的 bridge 网络是类似，适合规模较小的网络。如果需要创建较大规模的网络，可以使用 overlay 网络。
