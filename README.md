# IPVS CA

get ip vs(fullnat) client addr 

由taobao/toa修改，可作为独立模块编译安装, 支持tcp/udp

支持 centos6.6(linux 2.6.32-220) / centos7.2(linux 3.10.0-237.4.5) / ubuntu14.04(linux 3.13.0-77-generic)

对应内核在[github.com/yubo/LVS](https://github.com/yubo/LVS/tree/lvs_v2),兼容[taobao/LVS(lvs_v2)](https://github.com/alibaba/LVS/tree/lvs_v2)

支持taobao/lvs_v2版本的tcp opt报文格式，新加入了icmp echo报文(payload),实现了tcp/udp local - client 地址对应关系的通告

## Feature
  - [x] Build as a module
  - [x] Support TCP
  - [x] Support UDP
  - [x] Support centos 6.6
  - [x] Support centos 7.2 rpmbuild
  - [x] Support ubuntu 14.04(trusty) dpkg

## Demo

lvs(fullnat) client address TCP
[![TCP](https://asciinema.org/a/7e1qyj3ovn8yfe6a3srfcj104.png)](https://asciinema.org/a/7e1qyj3ovn8yfe6a3srfcj104?autoplay=1)

lvs(fullnat) client address UDP
[![UDP](https://asciinema.org/a/c0q9u1jhr367qay237azaep5e.png)](https://asciinema.org/a/c0q9u1jhr367qay237azaep5e?autoplay=1)

## Install

#### build kmod
```shell
cd src
make -f linux.mk
insmod ./ip_vs_ca.ko
```

### build rpm/deb 
```shell
cd build
cmake ..
#cmake -DENABLE_ICMP=1 -DENABLE_DEBUG=1 ..
make package
rpm -ivh ip_vs_ca-`uname -r`-0.1.0.x86_64.rpm
#or
dpkg -i ip_vs_ca-`uname -r`-0.1.0.x86_64.deb
modprobe ip_vs_ca
```

### proc sys ctl
可以通过修改以下文件来设置连接超时回收的时间
   - /proc/sys/net/ca/tcp_timeout (defualt 90s)
   - /proc/sys/net/ca/udp_timeout (defualt 180s)

## Udpd example

[udpd.c](udpd.c)

```c
	char recvbuf[1024] = {0};
	struct sockaddr_in peeraddr[2];
	socklen_t peerlen;
	int n;

	peerlen = sizeof(peeraddr);
	n = recvfrom(sock, recvbuf, sizeof(recvbuf), 0,
			(struct sockaddr *)peeraddr, &peerlen);
	if(peerlen == sizeof(struct sockaddr_in)){
		printf("recv %d %s:%d\n", peerlen,
			inet_ntoa(peeraddr[0].sin_addr),
			ntohs(peeraddr[0].sin_port));
	}else if(peerlen == sizeof(peeraddr)){
		printf("recv %d %s:%d", peerlen,
			inet_ntoa(peeraddr[0].sin_addr),
			ntohs(peeraddr[0].sin_port));
		printf("(%s:%d)\n",
			inet_ntoa(peeraddr[1].sin_addr),
			ntohs(peeraddr[1].sin_port));
	}
```

#注
该project直接fork了https://github.com/yubo/ip\_vs\_ca，在其上做了一点修改已解决遇到的问题，感谢作者
