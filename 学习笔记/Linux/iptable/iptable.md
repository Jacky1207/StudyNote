## Iptable

### 内核支持

    Enable IPTABLES support in Linux Kernel

    You need to recompile kernel to enable IPTABLES support. I am giving the steps to enable IPTABLES support during kernel recompilation.

    Get into the kernel source directory:

    # make menuconfig

    Select the following option (not as a loadable module)

    Networking >> Networking options >> Network packet filtering (replaces ipchains) >> Core Netfilter Configuration >> Netfilter Xtables support (required for ip_tables) and select the all following options as modules.

    Networking >> Networking options >> Network packet filtering (replaces ipchains) >> IP: Net Filter configurationS >> IP Tables support

    # make
    # make modules
    # make modules_install
    # make install
### 配置规则
~~~sh
[root@tp ~]# iptables -L -n
Chain INPUT (policy ACCEPT)
target       prot opt source                 destination         
Chain FORWARD (policy ACCEPT)
target       prot opt source                 destination         
Chain OUTPUT (policy ACCEPT)
target       prot opt source                 destination  
~~~
默认什么规则都没有.

#### 清除规则
~~~sh
[root@tp ~]# iptables -F        清除预设表filter中的所有规则链的规则
[root@tp ~]# iptables -X        清除预设表filter中使用者自定链中的规则
~~~

这些配置就像用命令配置IP一样,重启就会失去作用),怎么保存.
~~~sh
[root@tp ~]# /etc/rc.d/init.d/iptables save
[root@tp ~]# service iptables restart
~~~
这样就可以写到/etc/sysconfig/iptables文件里了.写入后记得把防火墙重起一下,才能起作用.

#### 规则配置

~~~sh
[root@tp ~]# iptables -p INPUT DROP
[root@tp ~]# iptables -p OUTPUT ACCEPT
[root@tp ~]# iptables -p FORWARD DROP
~~~
上面的意思是,当超出了IPTABLES里filter表里的两个链规则(INPUT,FORWARD)时,不在这两个规则里的数据包怎么处理呢,那就是DROP(放弃).应该说这样配置是很安全的.我们要控制流入数据包
而对于OUTPUT链,也就是流出的包我们不用做太多限制,而是采取ACCEPT,也就是说,不在着个规则里的包怎么办呢,那就是通过.
可以看出INPUT,FORWARD两个链采用的是允许什么包通过,而OUTPUT链采用的是不允许什么包通过.
这样设置还是挺合理的,当然你也可以三个链都DROP,但这样做我认为是没有必要的,而且要写的规则就会增加.但如果你只想要有限的几个规则是,如只做WEB服务器.还是推荐三个链都是DROP.

~~~sh
#远程SSH登陆,需要开启22端口.
[root@tp ~]# iptables -A INPUT -p tcp --dport 22 -j ACCEPT
[root@tp ~]# iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT
~~~
(注:这个规则,如果你把OUTPUT 设置成DROP的就要写上这一部,好多人都是望了写这一部规则导致,始终无法SSH.其他的端口也一样,如果开启了web服务器,OUTPUT设置成DROP的话,同样也要添加一条链:
[root@tp ~]# iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT ,其他同理.)

~~~sh
#WEB服务器,开启80端口.
[root@tp ~]# iptables -A INPUT -p tcp --dport 80 -j ACCEPT
#邮件服务器,开启25,110端口.
[root@tp ~]# iptables -A INPUT -p tcp --dport 110 -j ACCEPT
[root@tp ~]# iptables -A INPUT -p tcp --dport 25 -j ACCEPT
#FTP服务器,开启21端口
[root@tp ~]# iptables -A INPUT -p tcp --dport 21 -j ACCEPT
[root@tp ~]# iptables -A INPUT -p tcp --dport 20 -j ACCEPT
#DNS服务器,开启53端口
[root@tp ~]# iptables -A INPUT -p tcp --dport 53 -j ACCEPT
#允许icmp包通过,也就是允许ping,
[root@tp ~]# iptables -A OUTPUT -p icmp -j ACCEPT (OUTPUT设置成DROP的话)
[root@tp ~]# iptables -A INPUT -p icmp -j ACCEPT    (INPUT设置成DROP的话)
#允许loopback!(不然会导致DNS无法正常关闭等问题)
IPTABLES -A INPUT -i lo -p all -j ACCEPT (如果是INPUT DROP)
IPTABLES -A OUTPUT -o lo -p all -j ACCEPT(如果是OUTPUT DROP)
#下面写OUTPUT链,OUTPUT链默认规则是ACCEPT,所以我们就写需要DROP(放弃)的链.
#减少不安全的端口连接
[root@tp ~]# iptables -A OUTPUT -p tcp --sport 31337 -j DROP
[root@tp ~]# iptables -A OUTPUT -p tcp --dport 31337 -j DROP

#只允许192.168.0.3的机器进行SSH连接
[root@tp ~]# iptables -A INPUT -s 192.168.0.3 -p tcp --dport 22 -j ACCEPT
#如果要允许,或限制一段IP地址可用 192.168.0.0/24 表示192.168.0.1-255端的所有IP.
#24表示子网掩码数.但要记得把 /etc/sysconfig/iptables 里的这一行删了.
#-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT 因为它表示所有地址都可以登陆.或采用命令方式:
[root@tp ~]# iptables -D INPUT -p tcp --dport 22 -j ACCEPT
~~~

#### FORWARD链
FORWARD链的默认规则是DROP,所以就写需要ACCETP(通过)的链,对正在转发链的监控.
开启转发功能,(在做NAT时,FORWARD默认规则是DROP时,必须做)
~~~sh
[root@tp ~]# iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
[root@tp ~]# iptables -A FORWARD -i eth1 -o eh0 -j ACCEPT
#丢弃坏的TCP包
[root@tp ~]#iptables -A FORWARD -p TCP ! --syn -m state --state NEW -j DROP
#处理IP碎片数量,防止攻击,允许每秒100个
[root@tp ~]#iptables -A FORWARD -f -m limit --limit 100/s --limit-burst 100 -j ACCEPT
#设置ICMP包过滤,允许每秒1个包,限制触发条件是10个包.
[root@tp ~]#iptables -A FORWARD -p icmp -m limit --limit 1/s --limit-burst 10 -j ACCEPT
#在前面只所以允许ICMP包通过,就是因为在这里有限制.
~~~

#### NAT表
~~~sh
#查看本机关于NAT的设置情况
[root@tp rc.d]# iptables -t nat -L
Chain PREROUTING (policy ACCEPT)
target       prot opt source                 destination         
Chain POSTROUTING (policy ACCEPT)
target       prot opt source                 destination         
SNAT         all    --    192.168.0.0/24         anywhere              to:211.101.46.235
Chain OUTPUT (policy ACCEPT)
target       prot opt source                 destination    
#清除NAT命令
[root@tp ~]# iptables -F -t nat
[root@tp ~]# iptables -X -t nat
[root@tp ~]# iptables -Z -t nat
#添加规则
#添加基本的NAT地址转换,防止外网用内网IP欺骗
[root@tp sysconfig]# iptables -t nat -A PREROUTING -i eth0 -s 10.0.0.0/8 -j DROP
[root@tp sysconfig]# iptables -t nat -A PREROUTING -i eth0 -s 172.16.0.0/12 -j DROP
[root@tp sysconfig]# iptables -t nat -A PREROUTING -i eth0 -s 192.168.0.0/16 -j DROP

#禁止与211.101.46.253的所有连接
[root@tp ~]# iptables -t nat -A PREROUTING    -d 211.101.46.253 -j DROP
#禁用FTP(21)端口
[root@tp ~]# iptables -t nat -A PREROUTING -p tcp --dport 21 -j DROP
#这样写范围太大了,可以更精确的定义.
[root@tp ~]# iptables -t nat -A PREROUTING    -p tcp --dport 21 -d 211.101.46.253 -j DROP
#这样只禁用211.101.46.253地址的FTP连接,其他连接还可以.如web(80端口)连接.
#drop非法连接
[root@tp ~]# iptables -A INPUT     -m state --state INVALID -j DROP
[root@tp ~]# iptables -A OUTPUT    -m state --state INVALID -j DROP
[root@tp ~]# iptables-A FORWARD -m state --state INVALID -j DROP
#允许所有已经建立的和相关的连接
[root@tp ~]# iptables-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
[root@tp ~]# iptables-A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
~~~