#### iptables详解

基于上次redis的问题、牵涉到了防火墙，突然想起来之前遇到过虚拟机php环境，无法通过本地访问的情况，也是这个防火墙的原因，顺便、看了几篇文章，可能并不全面，也不够深入，仅仅是作为一个整理吧~~~

1. 简介：

```
由netfilter(内核空间)和iptables(用户空间)两个组件组成
netfilter是内核的一部分，由一些信息包过滤表组成，这些表包含内核用来控制信息包过滤处理的规则集
iptables是一种工具，使插入、修改和除去信息包过滤表中的规则变的更容易
```

2. 优点：

```
1）可以配置有状态的防火墙，有状态的防火墙可以指定并记住为发送或者接收信息包所建立的连接的状态，从信息包的连接跟踪状态来获取信息，在决定新的包过滤时可以增加速度、提高效率，一共有4种状态：
ESTABLISHED(已建立的连接，收发信息包，切完全有效)
INVALID(指出该信息包与任何已知的留或者连接都不关联，可能包含错误的数据或头)
NEW(该信息包已经或者将启动新的连接，或者它与尚未用于发送和接收信息包的连接相关联)
RELATED(该信息包正在启动新连接，以及它与已建立的连接相关联)
2）可以完全控制防火墙的配置和信息包过滤，只允许指定流量进入系统
3）免费~~~
```

3. 原理：

```
1. 规则：指定怎样处理一个数据包，可以指定原地址、目的地址、传输协议(TCP/UDP/ICMP等)和服务类型(如HTTP、FTP和SMTP)等，数据包和包规则匹配时，iptables就根据规则来处理包、可以accept、reject、drop(丢弃)，配置防火墙就是主要配置这些规则

2. 链(chains)就是数据包的传播路径、每一条链其实是众多规则中的一个检查清单，每条链可以有一条或者数条规则

3. 表提供特定功能，iptables内置了4张表，raw表、filter表、nat表和mangle表，分别实现包过滤、网络地址转换和包重构的功能

1）raw表
只使用在PREROUTING链和OUTPUT链上，优先级最高，从而可以对收到的数据包在连接跟踪前进行处理。一但用户使用了RAW表,在 某个链上,RAW表处理完后,将跳过NAT表和 ip_conntrack处理,即不再做地址转换和数据包的链接跟踪处理了.

2）filter表
主要用于过滤数据包，该表根据系统管理员预定义的一组规则过滤符合条件的数据包。对于防火墙而言，主要利用在filter表中指定的规则来实现对数据包的过滤。Filter表是默认的表，如果没有指定哪个表，iptables 就默认使用filter表来执行所有命令，filter表包含了INPUT链（处理进入的数据包），RORWARD链（处理转发的数据包），OUTPUT链（处理本地生成的数据包）在filter表中只能允许对数据包进行接受，丢弃的操作，而无法对数据包进行更改

3）nat表
主要用于网络地址转换NAT，该表可以实现一对一，一对多，多对多等NAT 工作，iptables就是使用该表实现共享上网的，NAT表包含了PREROUTING链（修改即将到来的数据包），POSTROUTING链（修改即将出去的数据包），OUTPUT链（修改路由之前本地生成的数据包）

4）mangle表
主要用于对指定数据包进行更改，在内核版本2.4.18 后的linux版本中该表包含的链为：INPUT链（处理进入的数据包），RORWARD链（处理转发的数据包），OUTPUT链（处理本地生成的数据包）POSTROUTING链（修改即将出去的数据包），PREROUTING链（修改即将到来的数据包）

```

3. iptables 规则

```
reject直接拦截数据包、返回数据包通知对方，可能是ICMP port-unreachable、ICMP echo-reply 或者tcp-reset，不再比对其它规则，直接中断过滤程序
iptables -A INPUT -p TCP --dprot 22 -j REJECT --reject-with ICMP echo-reply
在iptables的规则之上，添加一条规则：接受来自22端口的数据包，其它端口直接拒绝、响应对方消息为reply

drop直接丢弃不处理

redirect 将数据包转发到另一个端口 pnat，可用来做透明代理，或保护web服务器
iptables -t nat -A prerouting -p tcp --dport 80 -j REDIRECT --to-ports 8081
把来自80端口的数据包转发到8081端口

改写封包来源ip为防火墙i，可以指定port对应的返回，与snat略有不同，进行ip伪装是，无需指定伪装成哪一个ip
iptables -t nat -A POSTROUTING -p TCP -j MASQUERADE --to-ports 21000-31000

LOG 将数据包相关信息纪录在 /var/log 中
iptables -A INPUT -p tcp -j LOG --log-prefix "input packet"

SNAT 改写封包来源IP为某特定IP或IP范围，可以指定port对应的范围
iptables -t nat -A POSTROUTING -p tcp-o eth0 -j SNAT --to-source 192.168.10.15-192.168.10.160:2100-3200

MIRROR  镜像数据包，也就是将来源 IP与目的地IP对调后，将数据包返回，进行完此处理动作后，将会中断过滤程序。

QUEUE   中断过滤程序，将封包放入队列，交给其它程序处理。透过自行开发的处理程序，可以进行其它应用，例如：计算联机费用.......等。

RETURN  结束在目前规则链中的过滤程序，返回主规则链继续过滤，如果把自订规则炼看成是一个子程序，那么这个动作，就相当于提早结束子程序并返回到主程序中

MARK 将封包标上某个代号，以便提供作为后续过滤的条件判断依据
```

4. 保存规则：

```
使用iptables程序建立的规则只会保存在内存中，通常我们在修改了iptables的规则重启 iptables 后，之前修改的规则又消失了。那么如何保存新建立的规则呢？

1.对于RHEL和ceontos系统可以使用service iptables save将当前内存中的规则保存到/etc/sysconfig/iptables文件中
2. 修改/etc/sysconfig/iptables-config 将里面的IPTABLES_SAVE_ON_STOP="no", 这一句的"no"改为"yes"这样每次服务在停止之前会自动将现有的规则保存在 /etc/sysconfig/iptables 这个文件中去。
```

5. 常用命令：

```
1. -A --append添加新的规则 iptables -A INPUT -p tcp --dport 80 -j ACCEPT
2. -D --delete 删除规则 iptables -D INPUT -p tcp --dport 80 -j ACCEPT
   或者直接指定建立的规则的编号、iptables -D INPUT 1
3. -R --replace 取代现行第一条规则，被取代后不改变顺序
4. -I --insert 在第一条规则前插入一条规则
5. -L --list 列出input链中所有的规则
6. -F --flush删除input链中所有的规则
7. -Z --zero将input链中的数据包计数器归0
8. -N --new-chain 定义新的规则链
9. -X, --delete-chain  删除某个规则链
10.  -P, --policy 定义默认的过滤策略
11.  -E, --rename-chain 修改某自订规则链的名称 eg iptables -E denied disallowed
```

6. 常用封包比对参数：

```
-p --protocol知道协议 iptables -A INPUT -p tcp 是否是tcp协议包 可以使用 !tcp或者all
-s, --src, --source指定来源
-d, --dst, --destination用来比对封包的目的地 IP，设定方式同上
-i, --in-interface 对比数据包从哪个网卡进入
-o, --out-interface 对比数据包的目的端口
--sport, --source-port 指定源端口 --sport 22:80表示22到80端口都可以 ! 取反
--dport, --destination-port 目的端口
-m multiport --source-port 用来比对不连续的多个来源端口号，一次最多可以比对 15 个端口
-m limit --limit 用来比对某段时间内数据包的平均流量 
 eg iptables -A INPUT -m limit --limit 3/hour
    每小时平均流量是否超过一次3个数据包
-m mac --mac-source 用来比对数据包来源网络接口的硬件地址
-m owner --gid-owner 用来比对来自本机的数据包，是否为某特定使用者群组所产生
-m state --state  用来比对联机状态，联机状态共有四种：INVALID、ESTABLISHED、NEW 和 RELATED
iptables -L -n -v查看计数器
```

6. 常用的处理动作：

```
-j 参数用来指定要进行的处理动作，常用的处理动作包括：ACCEPT、REJECT、DROP、REDIRECT、MASQUERADE、LOG、DNAT、SNAT、MIRROR、QUEUE、RETURN、MARK
```

