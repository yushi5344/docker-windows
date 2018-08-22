# docker网络 #

### 创建两个容器 ###

	[vagrant@localhost ~]$ docker run --name demo1  -d yushi5344/flask-hello-world
	57889bb319a2b525508b4d5ff3fb3377e27cd23497fa09f97ee89153c6841eb5
	[vagrant@localhost ~]$ docker run --name demo2  -d yushi5344/flask-hello-world
	ca18cd2812704c97789122281b2e58b1947965dc0c36cda2ddf054df1e473211

### 进入容器，查看网络命名空间 ###
   
第一个容器

	[vagrant@localhost ~]$ ddocker exec -it ca18cd281270 /bin/sh
	# ip a
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
	    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
	       valid_lft forever preferred_lft forever
	

第二个容器

	[vagrant@localhost ~]$ docker exec -it 57889bb319a2 /bin/sh
	# ip a
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
	    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
	       valid_lft forever preferred_lft forever
	#

在第二个容器中ping第一个容器的地址

	# ping 172.17.0.3
	PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
	64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.090 ms
	64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.053 ms
	64 bytes from 172.17.0.3: icmp_seq=3 ttl=64 time=0.059 ms
	64 bytes from 172.17.0.3: icmp_seq=4 ttl=64 time=0.060 ms
	64 bytes from 172.17.0.3: icmp_seq=5 ttl=64 time=0.059 ms
	64 bytes from 172.17.0.3: icmp_seq=6 ttl=64 time=0.060 ms
	64 bytes from 172.17.0.3: icmp_seq=7 ttl=64 time=0.059 ms
	64 bytes from 172.17.0.3: icmp_seq=8 ttl=64 time=0.059 ms
	^C
	--- 172.17.0.3 ping statistics ---
	8 packets transmitted, 8 received, 0% packet loss, time 6998ms
	rtt min/avg/max/mdev = 0.053/0.062/0.090/0.012 ms
	#

### 创建一个网络命名空间 ###

	[vagrant@localhost ~]$ sudo ip netns add demo1
	[vagrant@localhost ~]$ sudo ip netns add demo2
	[vagrant@localhost ~]$ ip netns list
	demo2
	demo1

查看网络命名空间的ip地址

	[vagrant@localhost ~]$ sudo ip netns exec demo1 ip a
	1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	[vagrant@localhost ~]$ sudo ip netns exec demo1 ip link
	1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

可以看到，当前网络命名空间状态是down

设置为up 

	[vagrant@localhost ~]$ sudo ip netns exec demo1 ip link set dev lo up
	[vagrant@localhost ~]$ sudo ip netns exec demo1 ip link
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	[vagrant@localhost ~]$

单个端口是没办法up起来的，必须是一对   
可以在每个网络命名空间中创建一个 veth,然后连接起来，这个时候，两个命名空间就可以连通了。

查看当前的ip link

	[vagrant@localhost ~]$ iip link
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
	    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
	3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
	    link/ether 02:42:d4:32:31:60 brd ff:ff:ff:ff:ff:ff
	7: veth5f59380@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
	    link/ether c6:5d:ba:84:fa:61 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	9: vethbf83866@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
	    link/ether e2:2f:c8:ea:58:d2 brd ff:ff:ff:ff:ff:ff link-netnsid 1
	[vagrant@localhost ~]$


给demo1和demo2 分别添加一个veth  

	[vagrant@localhost ~]$ sudo ip link add veth-demo1 type veth peer name veth-demo2

查看当前ip link ,可以发现，多了一对

	[vagrant@localhost ~]$ ip link
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
	    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
	3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
	    link/ether 02:42:d4:32:31:60 brd ff:ff:ff:ff:ff:ff
	7: veth5f59380@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
	    link/ether c6:5d:ba:84:fa:61 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	9: vethbf83866@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
	    link/ether e2:2f:c8:ea:58:d2 brd ff:ff:ff:ff:ff:ff link-netnsid 1
	10: veth-demo2@veth-demo1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
	    link/ether 1e:0b:74:60:a4:62 brd ff:ff:ff:ff:ff:ff
	11: veth-demo1@veth-demo2: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
	    link/ether 02:7f:e5:72:e5:64 brd ff:ff:ff:ff:ff:ff
	[vagrant@localhost ~]$

将veth-demo1整个link加到demo1这个网络命名空间中

	[vagrant@localhost ~]$ sudo ip link set veth-demo1 netns demo1
	[vagrant@localhost ~]$ sudo ip netns exec demo1 ip link
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	11: veth-demo1@if10: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
	    link/ether 02:7f:e5:72:e5:64 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	[vagrant@localhost ~]$

同理，将veth-demo2这个link加到demo2这个网络命名空间中

	[vagrant@localhost ~]$ sudo ip link set veth-demo2 netns demo2
	[vagrant@localhost ~]$ sudo ip netns exec demo2 ip link
	1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	10: veth-demo2@if11: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
	    link/ether 1e:0b:74:60:a4:62 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	[vagrant@localhost ~]$


这个时候，在容器外部，ip link ,可以发现，多出来的两个link 不见了。

	[vagrant@localhost ~]$ ip link
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
	    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
	3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
	    link/ether 02:42:d4:32:31:60 brd ff:ff:ff:ff:ff:ff
	7: veth5f59380@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
	    link/ether c6:5d:ba:84:fa:61 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	9: vethbf83866@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
	    link/ether e2:2f:c8:ea:58:d2 brd ff:ff:ff:ff:ff:ff link-netnsid 1
	[vagrant@localhost ~]$

分别给这两个端口配上ip地址


	[vagrant@localhost ~]$ sudo ip netns exec demo1 ip addr add 192.168.1.1/24 dev veth-demo1
	[vagrant@localhost ~]$ sudo ip netns exec demo2 ip addr add 192.168.1.2/24 dev veth-demo2

分别启动这两个端口

	[vagrant@localhost ~]$ sudo ip netns exec demo1 ip link set dev veth-demo1 up
	[vagrant@localhost ~]$ sudo ip netns exec demo2 ip link set dev veth-demo2 up

查看设置的端口是否开启，ip是否正确


	[vagrant@localhost ~]$ sudo ip netns exec demo1 ip link
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	11: veth-demo1@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
	    link/ether 02:7f:e5:72:e5:64 brd ff:ff:ff:ff:ff:ff link-netnsid 1


可以看到，已经是up状态


	[vagrant@localhost ~]$ sudo ip netns exec demo1 ip a
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	    inet6 ::1/128 scope host
	       valid_lft forever preferred_lft forever
	11: veth-demo1@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
	    link/ether 02:7f:e5:72:e5:64 brd ff:ff:ff:ff:ff:ff link-netnsid 1
	    inet 192.168.1.1/24 scope global veth-demo1
	       valid_lft forever preferred_lft forever
	    inet6 fe80::7f:e5ff:fe72:e564/64 scope link
	       valid_lft forever preferred_lft forever
	[vagrant@localhost ~]$

demo1的ip地址为192.158.1.1：24

可以通过ping来判断两个namespace是否相通

在demo1中ping demo2的ip地址  

	[vagrant@localhost ~]$ sudo ip netns exec demo1 ping 192.168.1.2
	PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
	64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=0.060 ms
	64 bytes from 192.168.1.2: icmp_seq=2 ttl=64 time=0.041 ms
	64 bytes from 192.168.1.2: icmp_seq=3 ttl=64 time=0.041 ms
	^C
	--- 192.168.1.2 ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 2000ms
	rtt min/avg/max/mdev = 0.041/0.047/0.060/0.010 ms

在demo2中ping demo1的ip地址  

	[vagrant@localhost ~]$ sudo ip netns exec demo2 ping 192.168.1.1
	PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
	64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=0.044 ms
	64 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=0.042 ms
	64 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=0.040 ms
	^C
	--- 192.168.1.1 ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 2000ms
	rtt min/avg/max/mdev = 0.040/0.042/0.044/0.001 ms
	[vagrant@localhost ~]$

### 查看当前的网络 ###

	[vagrant@localhost ~]$ docker network ls
	NETWORK ID          NAME                DRIVER              SCOPE
	eb916d13a834        bridge              bridge              local
	8b1a430dd42b        host                host                local
	a1352ea570fd        none                null                local
	[vagrant@localhost ~]$

## docker中容器网络连接原理 ##

	[vagrant@localhost ~]$ ip a
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	    inet6 ::1/128 scope host
	       valid_lft forever preferred_lft forever
	2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
	    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
	    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
	       valid_lft 82976sec preferred_lft 82976sec
	    inet6 fe80::5054:ff:fec9:c704/64 scope link
	       valid_lft forever preferred_lft forever
	3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
	    link/ether 02:42:d4:32:31:60 brd ff:ff:ff:ff:ff:ff
	    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
	       valid_lft forever preferred_lft forever
	    inet6 fe80::42:d4ff:fe32:3160/64 scope link
	       valid_lft forever preferred_lft forever
	7: veth5f59380@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
	    link/ether c6:5d:ba:84:fa:61 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	    inet6 fe80::c45d:baff:fe84:fa61/64 scope link
	       valid_lft forever preferred_lft forever



通过ip a 可以看到，docker0是本机，也就是linux的网络地址，还有一个veth网络地址，veth网络接口负责连接到docker0上面.   
demo1中

 
	[vagrant@localhost ~]$ docker exec demo1 ip a
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
	    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
	       valid_lft forever preferred_lft forever
	[vagrant@localhost ~]$

eth0@if7这个网络接口和外面的veth5f59380@if6是一对，容器通过这一对veth-pair 连接到了主机上面，也就是docker0上面 。可以通过bridge-utils来验证。

先安装bridge-utils

	[vagrant@localhost ~]$ ssudo yum install bridge-utils

查看连接

	[vagrant@localhost ~]$ brctl show
	bridge name     bridge id               STP enabled     interfaces
	docker0         8000.0242d4323160       no              veth5f59380

可以看到，docker0连接到了 veth5f59380这个网络接口上面。


如果一个容器要访问另一个容器，但是另一个容器的ip地址不固定，这个时候，可以根据它的link名称访问

在创建第二个容器的时候加上参数--link

	[vagrant@localhost ~]$ docker run -d --name demo2 --link demo1 yushi5344/flask-hello-world

查看demo1的ip地址

	[vagrant@localhost ~]$ ssudo docker exec -it demo1 ip a
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	4: eth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
	    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
	       valid_lft forever preferred_lft forever
	[vagrant@localhost ~]$

进入demo2 ,查看demo2的ip地址

	[vagrant@localhost ~]$ sudo docker exec -it demo2 /bin/sh
	# ip a
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
	    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
	    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
	       valid_lft forever preferred_lft forever
	#

在demo2中pingdemo1的ip地址

	# ping 172.17.0.2
	PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
	64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.121 ms
	64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.063 ms
	^C
	--- 172.17.0.2 ping statistics ---
	2 packets transmitted, 2 received, 0% packet loss, time 999ms
	rtt min/avg/max/mdev = 0.063/0.092/0.121/0.029 ms
	#
在demo2中ping link的名称 

	# ping demo1
	PING demo1 (172.17.0.2) 56(84) bytes of data.
	64 bytes from demo1 (172.17.0.2): icmp_seq=1 ttl=64 time=0.060 ms
	64 bytes from demo1 (172.17.0.2): icmp_seq=2 ttl=64 time=0.096 ms
	^C
	--- demo1 ping statistics ---
	2 packets transmitted, 2 received, 0% packet loss, time 999ms
	rtt min/avg/max/mdev = 0.060/0.078/0.096/0.018 ms
	#

在实际的使用中，link的方式其实用的并不多。可以使用下面介绍的bridge方式。 

## 创建bridge ##

	[vagrant@localhost ~]$ sudo docker network create -d bridge my-bridge
	ee8fd65bda3297f01ee3dafc2634ac8c41edd56a7ef1ed5c04037ccf2e52fe50
	[vagrant@localhost ~]$ docker network ls
	NETWORK ID          NAME                DRIVER              SCOPE
	51bd9ea36625        bridge              bridge              local
	8b1a430dd42b        host                host                local
	ee8fd65bda32        my-bridge           bridge              local
	a1352ea570fd        none                null                local
	
	[vagrant@localhost ~]$ brctl show
	bridge name     bridge id               STP enabled     interfaces
	br-ee8fd65bda32         8000.024218604f16       no
	docker0         8000.0242c9e769f9       no              vethae8cabe

创建container的时候，如果不指定network参数，会默认链接到docker0这个网络。

	[vagrant@localhost ~]$ docker run -d  --name demo3 --network my-bridge yushi5344/flask-hello-world
	c16c0142695d28e2bb6ca1e7b00ad57eafd97e1d113cb00e1c6661dc28ad62c6

通过brctl show查看 ，可以发现，已经有了一个连接接口

	[vagrant@localhost ~]$ brctl show
	bridge name     bridge id               STP enabled     interfaces
	br-ee8fd65bda32         8000.024218604f16       no              veth7eac634
	docker0         8000.0242c9e769f9       no              vethae8cabe
	[vagrant@localhost ~]$

对于之前已经建好的container，可以使用下面的命令，重新指定网络连接。

	[vagrant@localhost ~]$ docker network connect my-bridge demo1

可以通过下面命令查看bridge连接的容器

	[vagrant@localhost ~]$ docker network inspect my-bridge

结果如下   

	"Containers": {
	            "57889bb319a2b525508b4d5ff3fb3377e27cd23497fa09f97ee89153c6841eb5": {
	                "Name": "demo1",
	                "EndpointID": "d4bee33bde93341a402ca418db96c36f332051fe52fa7a5237745a4edc0ecf2c",
	                "MacAddress": "02:42:ac:12:00:03",
	                "IPv4Address": "172.18.0.3/16",
	                "IPv6Address": ""
	            },
	            "c16c0142695d28e2bb6ca1e7b00ad57eafd97e1d113cb00e1c6661dc28ad62c6": {
	                "Name": "demo3",
	                "EndpointID": "33274bf95cb510f904341fa988aaf41a77fa5f5176e6447ca35b8e30a07d3d92",
	                "MacAddress": "02:42:ac:12:00:02",
	                "IPv4Address": "172.18.0.2/16",
	                "IPv6Address": ""
	            }
	        },


查看默认的bridge的容器连接情况

	[vagrant@localhost ~]$  docker network inspect bridge

结果如下 

	 "Containers": {
	            "57889bb319a2b525508b4d5ff3fb3377e27cd23497fa09f97ee89153c6841eb5": {
	                "Name": "demo1",
	                "EndpointID": "4fad531b45b4bb423def160fa5fa6e9c836bc570d4cb472bfb36c9eabd1d33d2",
	                "MacAddress": "02:42:ac:11:00:02",
	                "IPv4Address": "172.17.0.2/16",
	                "IPv6Address": ""
	            }
	        },


可以发现 demo1既连接着默认的bridge，又连接着新建的my-bridge.   

进入demo3,在demo3中ping my-bridge下demo1的ip地址

	# ping 172.18.0.3
	PING 172.18.0.3 (172.18.0.3) 56(84) bytes of data.
	64 bytes from 172.18.0.3: icmp_seq=1 ttl=64 time=0.085 ms
	64 bytes from 172.18.0.3: icmp_seq=2 ttl=64 time=0.057 ms
	^C
	--- 172.18.0.3 ping statistics ---
	2 packets transmitted, 2 received, 0% packet loss, time 999ms
	rtt min/avg/max/mdev = 0.057/0.071/0.085/0.014 ms
	#
直接通过container名称ping,也是可以ping通。

	# ping demo1
	PING demo1 (172.18.0.3) 56(84) bytes of data.
	64 bytes from demo1.my-bridge (172.18.0.3): icmp_seq=1 ttl=64 time=0.043 ms
	64 bytes from demo1.my-bridge (172.18.0.3): icmp_seq=2 ttl=64 time=0.059 ms
	64 bytes from demo1.my-bridge (172.18.0.3): icmp_seq=3 ttl=64 time=0.060 ms
	^C
	--- demo1 ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 2001ms
	rtt min/avg/max/mdev = 0.043/0.054/0.060/0.007 ms
	#

虽然在新建容器的时候，没有指定network，但是因为他们都连着我们自己新建的bridge，因此是可以ping通的。


### 端口映射 ###


将容器内的80端口映射到外部8090端口

	[vagrant@localhost ~]$ docker run --name web -d -p 8090:80 nginx


### None网络 ###

	[vagrant@localhost ~]$ docker run -d --name demo1 --network none yushi5344/flask-hello-world

查看

	[vagrant@localhost ~]$ docker network inspect none


可以发现，none网络ip地址都是空的


	 "Containers": {
	            "e2f2ba4eabfa4aad42e1572ec8c57641b86b3afe02c6be68e44d0e23af037cc6": {
	                "Name": "demo1",
	                "EndpointID": "c22e6ce512f1758233481322af8372680a672037fb0937142e566dfc5ca2fd3a",
	                "MacAddress": "",
	                "IPv4Address": "",
	                "IPv6Address": ""
	            }
	        }

查看demo1网络信息

	[vagrant@localhost ~]$ docker exec -it demo1  ip a
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	[vagrant@localhost ~]$

可以发现，没有任何对外的网络端口，也就是说，它是孤立的，除了exec访问，外部是无法访问的。这个容器的安全性可能比较高，用来存储密码之类的敏感信息。


### host网络 ###

创建一个container链接到host网络

	[vagrant@localhost ~]$ docker run -d --name demo2 --network host yushi5344/flask-hello-world

查看host网络信息

	[vagrant@localhost ~]$ docker network inspect host
	 "Containers": {
	            "fa712c985f3b91860b44af2d27cce152c0d28320822618ee60ad3d4a900d9e09": {
	                "Name": "demo2",
	                "EndpointID": "31602b3ffda327ed7607796bfb911a93d28d0851fa7f40e01bf81973a4643e64",
	                "MacAddress": "",
	                "IPv4Address": "",
	                "IPv6Address": ""
	            }
	        },

可以看到，demo2容器没有自己的ip地址和mac地址。
进入demo2查看

		[vagrant@localhost ~]$ docker exec -it demo2  ip a
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	    inet6 ::1/128 scope host
	       valid_lft forever preferred_lft forever
	2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
	    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
	    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
	       valid_lft 80956sec preferred_lft 80956sec
	    inet6 fe80::5054:ff:fec9:c704/64 scope link
	       valid_lft forever preferred_lft forever
	3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
	    link/ether 02:42:c9:e7:69:f9 brd ff:ff:ff:ff:ff:ff
	    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
	       valid_lft forever preferred_lft forever
	    inet6 fe80::42:c9ff:fee7:69f9/64 scope link
	       valid_lft forever preferred_lft forever
	8: br-ee8fd65bda32: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
	    link/ether 02:42:18:60:4f:16 brd ff:ff:ff:ff:ff:ff
	    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-ee8fd65bda32
	       valid_lft forever preferred_lft forever
	    inet6 fe80::42:18ff:fe60:4f16/64 scope link
	       valid_lft forever preferred_lft forever
	[vagrant@localhost ~]$

可以发现 demo2的ip信息和当前的主机ip信息是一样的，也就是说，链接到host的container是没有自己独立的namespace的，跟主机namespace共享。