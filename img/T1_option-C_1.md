# T1 option-C(1)

## 拓扑

![image-20220421132442944](T1_option-C_1.assets/image-20220421132442944.png)

## 一、L2 + VRRP（16分）

### 1.1 链路聚合（2分）

#### ① 假设S1不支持LACP，S1和S2互联的接口需要捆绑成一个二层逻辑接口。逻辑接口根据源目的MAC进行负载分担（2分）

##### 解法：

SW1、SW2：

~~~
#
interface Eth-Trunk 12    //进入已经存在的Eth-Trunk接口，或创建并进入Eth-Trunk接口
#
interface GigabitEthernet0/0/23    //进入指定的以太网接口或子接口视图
 eth-trunk 12    //用来将当前接口加入到指定Eth-Trunk中
#
interface GigabitEthernet0/0/24
 eth-trunk 12
#
interface Eth-Trunk12
 load-balance src-dst-mac    //配置Eth-Trunk接口的负载分担模式
#
~~~

##### 验证

SW1：

![image-20220421132507092](T1_option-C_1.assets/image-20220421132507092.png)

SW2：

![image-20220421132513043](T1_option-C_1.assets/image-20220421132513043.png)

### 1.2 Link-type（7分）

#### ① S1、S2、S3、S4互连接口的链路类型为Trunk，允许除VLAN 1以外的所有VLAN通过

##### 解法：

SW1、SW2：

~~~
#
interface GigabitEthernet0/0/10
 port link-type trunk    //配置接口的链路类型
 undo port trunk allow-pass vlan 1    //删除Trunk类型接口加入的VLAN
 port trunk allow-pass vlan 2 to 4094    //配置Trunk类型接口加入的VLAN
#
interface GigabitEthernet0/0/12
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 2 to 4094
#
interface Eth-Trunk12
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 2 to 4094
 load-balance src-dst-mac
#
~~~

SW3、SW4：

~~~
#
interface GigabitEthernet0/0/2
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 2 to 4094
#
interface GigabitEthernet0/0/10
 port link-type trunk
 undo port trunk allow-pass vlan 1
 port trunk allow-pass vlan 2 to 4094
#
~~~

##### 验证：

SW1：

![image-20220421132523788](T1_option-C_1.assets/image-20220421132523788.png)

SW2：

![image-20220421132530664](T1_option-C_1.assets/image-20220421132530664.png)

SW3：

![image-20220421132537532](T1_option-C_1.assets/image-20220421132537532.png)

SW4：

![image-20220421132542479](T1_option-C_1.assets/image-20220421132542479.png)

#### ② CE 1、CE2的VRRP虚拟IP地址10.3.1.254，为PC1的网关。CE1会周期性发送sender ip为10.3.1.254，源MAC为00-00-5E-00-01-01的免费ARP。PC1与网关之间的数据包封装在VLAN 10中（PC1收发untag帧）

#### ③ CE1、CE2的VRRP虚拟IP地址10.3.2.254，为Server1的网关。CE2会周期性发送sender ip为10.3.2.254，源MAC为00-00-5E-00-01-02的免费ARP。Server1与网关之间的数据包封装在VLAN 20中（Server1收发untag帧）

#### ④ VRRP的master设备重启时，在Ge0/0/2变为UP 1分钟后，才能重新成为master。（4分）

##### 解法：

SW1、SW2：

~~~
#
vlan batch 10 20    //批量创建多个VLAN
#
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 10 20
#
~~~

SW3、SW4：

~~~
#
vlan batch 10 20
#
interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 10
#
~~~

CE1：

~~~
#
interface GigabitEthernet0/0/2.10
 dot1q termination vid 10    //配置封装方式为dot1q，通过的报文外层Tag为10
 ip address 10.3.1.1 255.255.255.0 
 vrrp vrid 1 virtual-ip 10.3.1.254    //用来创建VRRP备份组并为备份组指定虚拟IP地址
 vrrp vrid 1 priority 105    //配置设备在VRRP备份组中的优先级
 vrrp vrid 1 preempt-mode timer delay 60    //配置VRRP备份组中路由器的抢占延迟时间
 arp broadcast enable    //使能设备子接口的ARP广播功能
#
interface GigabitEthernet0/0/2.20
 dot1q termination vid 20
 ip address 10.3.2.1 255.255.255.0 
 vrrp vrid 2 virtual-ip 10.3.2.254
 arp broadcast enable
#
~~~

CE2：

~~~
#
interface GigabitEthernet0/0/2.10
 dot1q termination vid 10
 ip address 10.3.1.2 255.255.255.0 
 vrrp vrid 1 virtual-ip 10.3.1.254
 arp broadcast enable
#
interface GigabitEthernet0/0/2.20
 dot1q termination vid 20
 ip address 10.3.2.2 255.255.255.0 
 vrrp vrid 2 virtual-ip 10.3.2.254
 vrrp vrid 2 priority 105
 vrrp vrid 2 preempt-mode timer delay 60
 arp broadcast enable
#
~~~

##### 验证：

CE1：

![image-20220421132553259](T1_option-C_1.assets/image-20220421132553259.png)

CE2：

![image-20220421132558990](T1_option-C_1.assets/image-20220421132558990.png)

PC1：

![image-20220421132604741](T1_option-C_1.assets/image-20220421132604741.png)

Server1：

![image-20220421132609770](T1_option-C_1.assets/image-20220421132609770.png)

### 1.3 MSTP（5分）

#### ① S1、S2、S3、S4都运行MSTP，VLAN 10 在instance 10中，S1为primary root，S2为secondary root。VLAN 20在instance 20中，S1为secondary root，S2为primary root。Region-name为HUAWEI，revision-level 为12。（3分）

#### ② 除了交换机互连的接口，其他接口要确保不参与MSTP计算，由disable变为forwarding状态。（2分）

##### 解法：

SW1：

~~~
#
stp region-configuration    //进入MST域视图
 region-name HUAWEI    //配置交换设备的MST域名
 revision-level 12    //配置交换设备的MSTP修订级别
 instance 10 vlan 10    //将VLAN 10映射到生成树实例10上
 instance 20 vlan 20
 active region-configuration    //用来激活MST域配置
#
stp instance 10 root primary    //配置当前设备为生成树实例10的根桥
stp instance 20 root secondary    //指定当前设备为生成树实例20的备份根桥
#
interface GigabitEthernet0/0/2
 stp edged-port enable    //配置当前端口为边缘端口
#
~~~

SW2：

~~~
#
stp region-configuration
 region-name HUAWEI
 revision-level 12
 instance 10 vlan 10
 instance 20 vlan 20
 active region-configuration
#
stp instance 10 root secondary
stp instance 20 root primary
#
#
interface GigabitEthernet0/0/2
 stp edged-port enable
#
~~~

SW3、SW4：

~~~
#
stp region-configuration
 region-name HUAWEI
 revision-level 12
 instance 10 vlan 10
 instance 20 vlan 20
 active region-configuration
#
interface GigabitEthernet0/0/1
 stp edged-port enable
#
~~~

##### 验证：

SW1：

![image-20220421132618901](T1_option-C_1.assets/image-20220421132618901.png)

![image-20220421132723147](T1_option-C_1.assets/image-20220421132723147.png)

![image-20220421132742727](T1_option-C_1.assets/image-20220421132742727.png)

![image-20220421132748349](T1_option-C_1.assets/image-20220421132748349.png)

SW2：

![image-20220421132752269](T1_option-C_1.assets/image-20220421132752269.png)

![image-20220421132756349](T1_option-C_1.assets/image-20220421132756349.png)

![image-20220421132803254](T1_option-C_1.assets/image-20220421132803254.png)![image-20220421132809282](T1_option-C_1.assets/image-20220421132809282.png)

SW3：

![image-20220421132813736](T1_option-C_1.assets/image-20220421132813736.png)

SW4：

![image-20220421132817813](T1_option-C_1.assets/image-20220421132817813.png)

### 1.4 WAN（2分）

#### ① PE1-RR1互连的Ser接口，捆绑为一个逻辑接口，成员采用HDLC。逻辑接口的IPv4、IPv6地址，按图配置。

![image-20220421132822118](T1_option-C_1.assets/image-20220421132822118.png)

![image-20220421132825219](T1_option-C_1.assets/image-20220421132825219.png)

##### 解法

PE1：

~~~
#
interface Ip-Trunk1    //创建一个IP-Trunk接口，并进入IP-Trunk接口视图
 ipv6 enable
 ip address 10.1.13.1 255.255.255.252
 ipv6 address 2000:EAD8:99EF:C03F:B2AD:9EFF:32DD:DC10/127
#
interface Serial0/0/0
 link-protocol hdlc    //配置接口的链路层协议为HDLC
 ip-trunk 1    //配置当前POS接口加入指定IP-Trunk
#
interface Serial0/0/1
 link-protocol hdlc
 ip-trunk 1
#
~~~

RR1

~~~
#
interface Ip-Trunk1
 ipv6 enable
 ip address 10.1.13.2 255.255.255.252
 ipv6 address 2000:EAD8:99EF:C03F:B2AD:9EFF:32DD:DC11/127
#
interface Serial0/0/0
 link-protocol hdlc
 ip-trunk 1
#
interface Serial0/0/1
 link-protocol hdlc
 ip-trunk 1
#
~~~

##### 验证

PE1：

![image-20220421132832697](T1_option-C_1.assets/image-20220421132832697.png)

![image-20220421132835736](T1_option-C_1.assets/image-20220421132835736.png)

![image-20220421132843746](T1_option-C_1.assets/image-20220421132843746.png)

![image-20220421132847850](T1_option-C_1.assets/image-20220421132847850.png)

RR1：

![image-20220421132852592](T1_option-C_1.assets/image-20220421132852592.png)

![image-20220421132856577](T1_option-C_1.assets/image-20220421132856577.png)

#### ② PE3-CE3互连的POS接口，绑定为一个逻辑接口，成员采用PPP，逻辑接口的IPv4地址，按图配置。注：是P4/0/0和P6/0/0

![image-20220421132901344](T1_option-C_1.assets/image-20220421132901344.png)

##### 解法：

PE3：

~~~
#
interface Mp-group0/0/1    //创建MP-Group接口并进入MP-Group接口视图
 ip address 10.2.33.2 255.255.255.252 
#
interface Pos4/0/0
 link-protocol ppp    //配置POS接口的链路层协议
 ppp mp Mp-group 0/0/1    //将接口加入指定的MP-group，使该接口工作在MP方式
#
interface Pos6/0/0
 link-protocol ppp
 ppp mp Mp-group 0/0/1
#
~~~

CE3：

~~~
#
interface Mp-group0/0/1
 ip address 10.2.33.1 255.255.255.252 
#
interface Pos4/0/0
 link-protocol ppp
 ppp mp Mp-group 0/0/1
#
interface Pos6/0/0
 link-protocol ppp
 ppp mp Mp-group 0/0/1
#
~~~

##### 验证：

PE3：

![image-20220421132907342](T1_option-C_1.assets/image-20220421132907342.png)

![image-20220421132913375](T1_option-C_1.assets/image-20220421132913375.png)

![image-20220421132917853](T1_option-C_1.assets/image-20220421132917853.png)

CE3：

![image-20220421132923052](T1_option-C_1.assets/image-20220421132923052.png)

![image-20220421132931804](T1_option-C_1.assets/image-20220421132931804.png)



## 二、IPv4 IGP

### 2.1 基础配置

### 2.2 OSPF（6分）

#### ① CE1和CE2之间的链路，及该两台设备的loopback 0，通告入OSPF区域（已配）

#### ② CE1的Ge0/0/2.10和Ge0/0/2.20，CE2的Ge0/0/2.10和Ge0/0/2.20直连网断通告入OSPF区域0，但这些接口不能转发OSPF报文。（2分）

##### 解法：

CE1：

~~~
#
ospf 1 router-id 172.17.1.1    //用来创建并运行OSPF进程，配置Router ID
 silent-interface GigabitEthernet0/0/2.10    //禁止接口接收和发送OSPF报文
 silent-interface GigabitEthernet0/0/2.20
 area 0.0.0.0    //创建OSPF区域，并进入OSPF区域视图
  network 10.2.12.1 0.0.0.0    //指定运行OSPF协议的接口
  network 10.3.1.1 0.0.0.0 
  network 10.3.2.1 0.0.0.0 
  network 172.17.1.1 0.0.0.0 
#
~~~

CE2：

~~~
#
ospf 1 router-id 172.17.1.2 
 silent-interface GigabitEthernet0/0/2.10
 silent-interface GigabitEthernet0/0/2.20
 area 0.0.0.0 
  network 10.2.12.2 0.0.0.0 
  network 10.3.1.2 0.0.0.0 
  network 10.3.2.2 0.0.0.0 
  network 172.17.1.2 0.0.0.0 
#
~~~

验证：

CE1：

![image-20220421132946364](T1_option-C_1.assets/image-20220421132946364.png)

CE2：

![image-20220421132950706](T1_option-C_1.assets/image-20220421132950706.png)

#### ③ RR2、P2、PE3、PE4在OSPF区域0中，cost如图配置（已配）

![image-20220421132955107](T1_option-C_1.assets/image-20220421132955107.png)

#### ④ PE3-PE4的OSPF链路类型为P2P。（1分）

##### 解法：

PE3：

~~~、
#
interface GigabitEthernet0/0/0
 ip address 10.1.112.1 255.255.255.252 
 ospf cost 20    //配置接口上运行OSPF协议所需的开销
 ospf network-type p2p    //设置OSPF接口的网络类型
 mpls
 mpls ldp
#
~~~

PE4：

~~~
#
interface GigabitEthernet0/0/0
 ip address 10.1.112.2 255.255.255.252 
 ospf cost 20
 ospf network-type p2p
 mpls
 mpls ldp
#
~~~

##### 验证：

PE3：

![image-20220421133004123](T1_option-C_1.assets/image-20220421133004123.png)

PE4：

![image-20220421133007473](T1_option-C_1.assets/image-20220421133007473.png)

#### ⑤ PE4上将loopback 0地址引入OSPF。在AS 200中，各网元到PE4 loopback 0的路由要包含内部cost。（3分）

##### 解法：

PE4：

~~~
#
ip ip-prefix lo0 index 10 permit 172.16.1.2 32    //配置名为lo0的地址前缀列表，路由172.16.1.2/32被Permit
#
route-policy lo0 permit node 10    //配置路由策略lo0，其节点号为10，匹配模式为允许
 if-match ip-prefix lo0    //创建一个基于IP地址前缀列表lo0的匹配规则
#
ospf 1 
 import-route direct type 1 tag 100 route-policy lo0    //配置只能引入符合指定路由策略的直连路由，路由标记为100，外部路由的类型为第一类外部路由
 area 0.0.0.0 
  network 10.1.102.2 0.0.0.0 
  network 10.1.112.2 0.0.0.0 
#
~~~

##### 验证：

RR2：

![image-20220421133012720](T1_option-C_1.assets/image-20220421133012720.png)

P2：

![image-20220421133020707](T1_option-C_1.assets/image-20220421133020707.png)

PE3：

![image-20220421133024560](T1_option-C_1.assets/image-20220421133024560.png)

### 2.3 IS-IS（12分）

#### ① 在AS 100内，loopback 0和互连接口全部开启IS-IS协议，其中PE1、PE2路由类型L1，区域号为49.0001；RR1、P1的路由类型L1-2，区域号为49.0001；ASBR1、ASBR2路由类型为L2，区域号为49.0002；各网元system-ID唯一，cost-style为wide；cost值如图配置（除PE1-RR1之间的逻辑链路，其余已配）（1分）

![image-20220421133033947](T1_option-C_1.assets/image-20220421133033947.png)

##### 解法：

PE1：

~~~
#
interface Ip-Trunk1
 ipv6 enable
 ip address 10.1.13.1 255.255.255.252
 ipv6 address 2000:EAD8:99EF:C03F:B2AD:9EFF:32DD:DC10/127
 isis enable 1    //在接口上使能IS-IS功能并指定要关联的IS-IS进程号
 isis cost 3000    //配置IS-IS接口的链路开销值
#
~~~

RR1：

~~~
#
interface Ip-Trunk1
 ipv6 enable
 ip address 10.1.13.2 255.255.255.252
 ipv6 address 2000:EAD8:99EF:C03F:B2AD:9EFF:32DD:DC11/127
 isis enable 1
 isis cost 3000
#
~~~

##### 验证：

PE1：

![image-20220421133039907](T1_option-C_1.assets/image-20220421133039907.png)

RR1：

![image-20220421133043674](T1_option-C_1.assets/image-20220421133043674.png)

#### ② RR2-P2的IS-IS链路类型P2P（1分）

##### 解法：

RR2：

~~~
#
interface GigabitEthernet0/0/0
 ip address 10.1.91.1 255.255.255.252 
 isis enable 1
 isis circuit-type p2p    //将IS-IS广播网接口的网络类型模拟为P2P类型
 isis cost 50
 ospf cost 50
 mpls
 mpls ldp
#
~~~

P2：

~~~
#
interface GigabitEthernet0/0/0
 ip address 10.1.91.2 255.255.255.252 
 isis enable 1
 isis circuit-type p2p
 isis cost 50
 ospf cost 50
 mpls
 mpls ldp
#
~~~

验证：

RR2：

![image-20220421133056230](T1_option-C_1.assets/image-20220421133056230.png)

P2：

![image-20220421133100386](T1_option-C_1.assets/image-20220421133100386.png)

#### ③在RR2，P2上，IS-IS和OSPF双向引入前缀为172.16.0.0/16的主机路由，被引入协议的cost要继承到后引入的协议中，P2和PE4的loopback 0互访走最优路径。配置要求有最好的扩展性。（注：本题考试时可能会转移到MPLS部分）（8分）

##### 解法：

RR2：

~~~
#
ip ip-prefix host index 10 permit 172.16.0.0 16 greater-equal 32 less-equal 32    //配置名为host的地址前缀列表，只允许172.16.0.0/16网段内，掩码长度在32到32之间的路由通过
#
route-policy o2i deny node 10    //先拒绝，防止P2中IS-IS引入到OSPF的路由在RR2上被重新引入到IS-IS中
 if-match tag 1099    //定义一个规则，用来匹配标记字段为1099的路由信息
#
route-policy o2i permit node 20 
 if-match ip-prefix host    //创建一个基于IP地址前缀列表host的匹配规则
 apply tag 9910    //设置路由信息的标记为9910
#
route-policy i2o deny node 10    //先拒绝，防止P2中OSPF引入到IS-IS的路由在RR2上被重新引入到OSPF中
 if-match tag 109
#
route-policy i2o permit node 20 
 if-match ip-prefix host 
 apply tag 910
#
route-policy pe4 permit node 10 
 if-match tag 100
 apply preference 8    //为路由指定优先级
#
ospf 1 
 default cost inherit-metric    //引入路由的开销值为路由自带的cost值（此为IS-IS协议cost继承到OSPF中）
 import-route isis 1 route-policy i2o    //配置只能引入符合指定路由策略的IS-IS路由，引入IS-IS路由协议的进程号为1
 preference ase route-policy pe4 150    //设置AS-External路由的优先级，通过策略pe4的路由优先级被设定为8，未通过策略pe4的路由优先级被设定为150
 area 0.0.0.0 
  network 10.1.91.1 0.0.0.0 
  network 10.1.119.1 0.0.0.0 
#
isis 1
 is-level level-2    //配置IS-IS路由器的级别
 cost-style wide    //设置IS-IS设备接收和发送路由的开销类型
 network-entity 49.0002.1720.1600.1009.00    //配置IS-IS进程的网络实体名称（NET）
 import-route ospf 1 inherit-cost route-policy o2i    //配置只能引入符合指定路由策略的OSPF路由，引入OSPF路由协议的进程号为1，保留路由的原有开销值（此为OSPF协议cost继承到IS-IS）
#
~~~

P2：

~~~
#
ip ip-prefix host index 10 permit 172.16.0.0 16 greater-equal 32 less-equal 32
#
route-policy o2i deny node 10 
 if-match tag 910
#
route-policy o2i permit node 20 
 if-match ip-prefix host 
 apply tag 109 
#
route-policy i2o deny node 10 
 if-match tag 9910
#
route-policy i2o permit node 20 
 if-match ip-prefix host 
 apply tag 1099 
#
route-policy pe4 permit node 10 
 if-match tag 100
 apply preference 8 
#
ospf 1 
 default cost inherit-metric
 import-route isis 1 route-policy i2o
 preference ase route-policy pe4 150 
 area 0.0.0.0 
  network 10.1.91.2 0.0.0.0 
  network 10.1.102.1 0.0.0.0 
#
isis 1
 is-level level-2
 cost-style wide
 network-entity 49.0002.1720.1600.1010.00
 import-route ospf 1 inherit-cost route-policy o2i 
#
~~~

##### 验证：

RR2：

![image-20220421133107349](T1_option-C_1.assets/image-20220421133107349.png)

P2：

![image-20220421133111427](T1_option-C_1.assets/image-20220421133111427.png)

PE3：

![image-20220421133116210](T1_option-C_1.assets/image-20220421133116210.png)

PE4：

![image-20220421133120970](T1_option-C_1.assets/image-20220421133120970.png)

ASBR3：

![image-20220421133124994](T1_option-C_1.assets/image-20220421133124994.png)

ASBR4：

![image-20220421133210493](T1_option-C_1.assets/image-20220421133210493.png)

#### ④ P1的IS-IS进程：产生LSP的最大延迟时间是1S，初始延迟为50ms，递增时间为50ms，使能LSP的快速扩散特性；SPF计算最大延迟时间是1S，初始延迟为100ms，递增时间为100ms。（2分）

##### 解法：

P1：

~~~
#
isis 1
 cost-style wide
 timer lsp-generation 1 50 50 level-1    //设置产生LSP的最大延迟为1秒，初始延迟为50毫秒，递增延迟时间为50毫秒。
 timer lsp-generation 1 50 50 level-2
 flash-flood level-1    //使能LSP快速扩散特性，以便加快IS-IS网络的收敛速度
 flash-flood level-2
 network-entity 49.0001.1720.1600.1004.00
 timer spf 1 100 100    //设置SPF计算最大延迟为1秒，初始延迟为100毫秒，递增时间为100毫秒。
#
~~~

##### 验证：

可在P1上模拟链路DOWN，查看收敛情况

## 三、MPLS VPN（35分）

#### ① CE1、CE2为VPN 1的Hub-CE，PE1、PE2为Hub-PE，CE3、CE4为VPN的spoke站点；PE3、PE4为SPOKE-PE

#### ② CE4为Multi-VPN-instance CE，CE4的VPN实例1，通过Ge0/0/1连接PE4

##### 解法：

CE4：

~~~
#
ip vpn-instance 1    //创建VPN实例，并进入VPN实例视图
 ipv4-family    //使能VPN实例的IPv4地址族，并进入VPN实例IPv4地址族视图
  route-distinguisher 4:4    //为VPN实例地址族配置路由标识RD
#
interface GigabitEthernet0/0/1
 ip binding vpn-instance 1    //将CE上的接口与VPN实例绑定，配置接口与VPN实例绑定后，或取消接口与VPN实例的绑定，都会清除该接口的IP地址、三层特性和IP相关的路由协议，如果需要应重新配置。接口不能与未使能任何地址族的VPN实例绑定。
 ip address 10.2.41.1 255.255.255.252 
#
~~~

##### 验证：

CE4：

![image-20220421203310989](T1_option-C_1.assets/image-20220421203310989.png)

![image-20220421203322680](T1_option-C_1.assets/image-20220421203322680.png)

#### ③ 合理设置VPN 1参数，使得Spoke站点互访的流量必须经过Hub-CE设备。当CE1-PE1链路断开的情况下，PE1仍然可以学习到CE1的业务路由。（PE3上的VPN 1的RD为100:13，EXPORT RT为100:1，import RT为200:1）（2分）

##### 解法：

PE3：

~~~
#
ip vpn-instance 1
 ipv4-family
  route-distinguisher 100:13
  vpn-target 100:1 export-extcommunity    //配置VPN实例地址族出方向的VPN-Target扩展团体属性
  vpn-target 200:1 import-extcommunity    //配置VPN实例地址族入方向的VPN-Target扩展团体属性
#
~~~

PE4：

~~~
#
ip vpn-instance 1
 ipv4-family
  route-distinguisher 100:14
  vpn-target 100:1 export-extcommunity
  vpn-target 200:1 import-extcommunity
#
~~~

PE1：

~~~
#
ip vpn-instance 1-in
 ipv4-family
  route-distinguisher 13:14
  vpn-target 100:1 200:1 import-extcommunity
#
ip vpn-instance 1-out
 ipv4-family
  route-distinguisher 11:11
  vpn-target 200:1 export-extcommunity
#
~~~

PE2：

~~~
#
ip vpn-instance 1-in
 ipv4-family
  route-distinguisher 13:14
  vpn-target 100:1 200:1 import-extcommunity
#
ip vpn-instance 1-out
 ipv4-family
  route-distinguisher 22:22
  vpn-target 200:1 export-extcommunity
#
~~~

##### 验证：

PE3：

![image-20220421204815245](T1_option-C_1.assets/image-20220421204815245.png)

PE4：

![image-20220421204823761](T1_option-C_1.assets/image-20220421204823761.png)

PE1：

![image-20220421204831547](T1_option-C_1.assets/image-20220421204831547.png)

PE2：

![image-20220421204840342](T1_option-C_1.assets/image-20220421204840342.png)

#### ④ 如图，CE1通过Ge0/0/1.1和Ge0/0/1.2建立直接EBGP邻居，接入PE1。CE1通过Ge0/0/1.2，向PE1通告BGP update中，某些路由信息的AS-path中有200。在CE1上，将OSPF路由导入BGP（2分）

![image-20220421204921167](T1_option-C_1.assets/image-20220421204921167.png)

##### 解法：

CE1：

~~~
#
bgp 65000    //启动BGP，进入BGP视图
 peer 10.2.11.2 as-number 100    //配置IPv4对等体10.2.11.2的对端AS号为100
 peer 10.2.11.6 as-number 100 
 #
 ipv4-family unicast    //进入BGP-IPv4单播地址族视图，BGP缺省使能的地址族是IPv4单播地址族
  undo synchronization    //用来关闭BGP与IGP的同步功能，缺省情况下，同步功能是关闭的。
  import-route ospf 1    //引入OSPF进程1的路由
  peer 10.2.11.2 enable    //缺省情况下，只有BGP-IPv4单播地址族的对等体是自动使能的
  peer 10.2.11.6 enable
#
interface GigabitEthernet0/0/1.1
 dot1q termination vid 1
 ip address 10.2.11.1 255.255.255.252 
 arp broadcast enable
#
interface GigabitEthernet0/0/1.2
 dot1q termination vid 2
 ip address 10.2.11.5 255.255.255.252 
 arp broadcast enable
#
~~~

PE1：

~~~
#
bgp 100
 peer 172.16.1.3 as-number 100
 peer 172.16.1.3 connect-interface LoopBack0
 peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DA03 as-number 100
 peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DA03 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 172.16.1.3 enable
 #
 ipv6-family unicast
  undo synchronization
  peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DA03 enable
 #
 ipv4-family vpn-instance 1-in    //将指定的VPN实例与IPv4地址族进行关联，并进入BGP-VPN实例IPv4地址族视图
  peer 10.2.11.1 as-number 65000
 #
 ipv4-family vpn-instance 1-out
  peer 10.2.11.5 as-number 65000
#
interface GigabitEthernet0/0/1.1
 dot1q termination vid 1
 ip binding vpn-instance 1-in
 ip address 10.2.11.2 255.255.255.252
 arp broadcast enable
#
interface GigabitEthernet0/0/1.2
 dot1q termination vid 2
 ip binding vpn-instance 1-out
 ip address 10.2.11.6 255.255.255.252
 arp broadcast enable
#
~~~

##### 验证：

CE1：

![img](T1_option-C_1.assets/d639c3ff-f4b9-4029-ab97-e8aa4d993f83-5395106.jpg)

PE1：

![img](T1_option-C_1.assets/cd9c7d5e-67de-4073-a577-33b4583c196a-5395106.jpg)

#### ⑤ CE2通过Ge0/0/1.1和Ge0/0/1.2建立直接EBGP邻居，接入PE2。CE2通过Ge0/0/1.2，向PE2通告BGP update中，某些路由信息的AS-path中有200。在CE1上，将OSPF路由导入BGP（2分）

##### 解法：

CE2：

~~~
#
bgp 65000
 peer 10.2.22.2 as-number 100 
 peer 10.2.22.6 as-number 100 
 #
 ipv4-family unicast
  undo synchronization
  import-route ospf 1
  peer 10.2.22.2 enable
  peer 10.2.22.6 enable
#
interface GigabitEthernet0/0/1.1
 dot1q termination vid 1
 ip address 10.2.22.1 255.255.255.252 
 arp broadcast enable
#
interface GigabitEthernet0/0/1.2
 dot1q termination vid 2
 ip address 10.2.22.5 255.255.255.252 
 arp broadcast enable
#
~~~

PE2：

~~~
#
bgp 100
 peer 172.16.1.3 as-number 100
 peer 172.16.1.3 connect-interface LoopBack0
 peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DA03 as-number 100
 peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DA03 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 172.16.1.3 enable
 #
 ipv6-family unicast
  undo synchronization
  peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DA03 enable
 #
 ipv4-family vpn-instance 1-in
  peer 10.2.22.1 as-number 65000
 #
 ipv4-family vpn-instance 1-out
  peer 10.2.22.5 as-number 65000
#
interface GigabitEthernet0/0/1.1
 dot1q termination vid 1
 ip binding vpn-instance 1-in
 ip address 10.2.22.2 255.255.255.252
 arp broadcast enable
#
interface GigabitEthernet0/0/1.2
 dot1q termination vid 2
 ip binding vpn-instance 1-out
 ip address 10.2.22.6 255.255.255.252
 arp broadcast enable
#
~~~

##### 验证：

CE2：

![image-20220421211512921](T1_option-C_1.assets/image-20220421211512921.png)

PE2：

![image-20220421211530544](T1_option-C_1.assets/image-20220421211530544.png)

#### ⑥ CE3通过OSPF区域1接入PE3，通过PE3-CE3的逻辑接口互通，通告CE3的各环回口；CE4通过OSPF区域0接入PE4，通过PE4-CE4的Ge0/0/1接口互通，通告CE4的各环回口（2分）

##### 解法：

CE3：

~~~
#
ospf 1 router-id 172.17.1.3 
 area 0.0.0.1 
  network 10.2.33.1 0.0.0.0 
  network 10.3.3.3 0.0.0.0 
  network 172.17.1.3 0.0.0.0 
#
~~~

PE3：

~~~
#
ospf 65001 router-id 172.16.1.11 vpn-instance 1    //指定VPN实例名称，如果指定了VPN实例，那么OSPF进程属于此实例，否则属于全局实例
 import-route bgp
 area 0.0.0.1 
  network 10.2.33.2 0.0.0.0 
#
interface Mp-group0/0/1
 ip binding vpn-instance 1
 ip address 10.2.33.2 255.255.255.252 
#
bgp 200
 peer 172.16.1.9 as-number 200 
 peer 172.16.1.9 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 172.16.1.9 enable
 #
 ipv4-family vpn-instance 1 
  import-route ospf 65001
#
~~~

CE4：

~~~
#
ospf 1 router-id 172.17.1.4 vpn-instance 1
 vpn-instance-capability simple    //用来禁止路由环路检测，直接进行路由计算（防止因为DN位的关系不收路由）
 area 0.0.0.0 
  network 10.2.41.1 0.0.0.0 
  network 10.3.3.4 0.0.0.0 
  network 172.17.1.4 0.0.0.0 
#
interface LoopBack0
 ip binding vpn-instance 1
 ip address 172.17.1.4 255.255.255.255 
#
interface LoopBack1
 ip binding vpn-instance 1
 ip address 10.3.3.4 255.255.255.255 
#
interface GigabitEthernet0/0/1
 ip binding vpn-instance 1
 ip address 10.2.41.1 255.255.255.252 
#
~~~

PE4：

~~~
#
ospf 65001 router-id 172.16.1.2 vpn-instance 1
 import-route bgp
 area 0.0.0.0 
  network 10.2.41.2 0.0.0.0 
#
bgp 200
 peer 172.16.1.9 as-number 200 
 peer 172.16.1.9 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 172.16.1.9 enable
 #
 ipv4-family vpn-instance 1 
  import-route ospf 65001
#
interface GigabitEthernet0/0/1
 ip binding vpn-instance 1
 ip address 10.2.41.2 255.255.255.252 
#
~~~

##### 验证：

CE3：

![image-20220421213510335](T1_option-C_1.assets/image-20220421213510335.png)

PE3：

![image-20220421213530059](T1_option-C_1.assets/image-20220421213530059.png)

CE4：

![image-20220421213643850](T1_option-C_1.assets/image-20220421213643850.png)

PE4：

![image-20220421213702505](T1_option-C_1.assets/image-20220421213702505.png)

#### ⑦ 如图，在AS 100、AS 200内建立IBGP IPv4邻居关系，RR1是PE1、PE2、P1、ASBR1、ASBR2的反射器，RR2是PE3、PE4、P2、ASBR4、ASBR3的反射器。ASBR1-ASBR3，ASBR2-ASBR4建立EBGP IPv4邻居关系。（已预配）

![image-20220421213828057](T1_option-C_1.assets/image-20220421213828057.png)

#### ⑧ 在ASBR上，将IS-IS的loopback 0路由引入BGP（2分）

##### 解法：

ASBR1、ASBR2：

~~~
#
ip ip-prefix as100 index 10 permit 172.16.1.1 32
ip ip-prefix as100 index 20 permit 172.16.1.20 32
ip ip-prefix as100 index 30 permit 172.16.1.3 32
ip ip-prefix as100 index 40 permit 172.16.1.4 32
ip ip-prefix as100 index 50 permit 172.16.1.5 32
ip ip-prefix as100 index 60 permit 172.16.1.6 32
#
bgp 100
 #
 ipv4-family unicast
  import-route isis 1 med 0 route-policy as100
#
~~~

ASBR3、ASBR4：

~~~
#
ip ip-prefix as200 index 10 permit 172.16.1.7 32
ip ip-prefix as200 index 20 permit 172.16.1.8 32
ip ip-prefix as200 index 30 permit 172.16.1.9 32
ip ip-prefix as200 index 40 permit 172.16.1.10 32
ip ip-prefix as200 index 50 permit 172.16.1.11 32
ip ip-prefix as200 index 60 permit 172.16.1.2 32
#
bgp 200
 #
 ipv4-family unicast
  import-route isis 1 med 0 route-policy as200
#
~~~

##### 验证：

ASBR1：

![image-20220422152808456](T1_option-C_1.assets/image-20220422152808456.png)

#### ⑨ 如图，在AS 100、AS 200内各网元配置MPLS LSR-IP，全局使能MPLS，MPLS LDP（已配）AS 100、AS 200内各有直连链路建立LDP（除PE1-RR1之间，其余已配）注：ASBR之间MPLS没有预配（1分）

![image-20220422160704420](T1_option-C_1.assets/image-20220422160704420.png)

##### 解法：

PE1：

~~~
#
interface Ip-Trunk1
 ipv6 enable
 ip address 10.1.13.1 255.255.255.252
 ipv6 address 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC10/127
 isis enable 1
 isis cost 3000
 mpls    //使能所在接口的MPLS能力
 mpls ldp    //使能接口上的MPLS LDP功能
#
~~~

RR1：

~~~
#
interface Ip-Trunk1
 ipv6 enable
 ip address 10.1.13.2 255.255.255.252
 ipv6 address 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC11/127
 isis enable 1
 isis cost 3000
 mpls
 mpls ldp
#
~~~

##### 验证：

PE1：

![image-20220422160347219](T1_option-C_1.assets/image-20220422160347219.png)

RR1：

![image-20220422160354033](T1_option-C_1.assets/image-20220422160354033.png)

#### ⑩ 如图，各站点通过MPLS BGP VPN跨域Option C方案一能够相互学习路由，MPLS域不能出现次优路径（15分）

##### 解法：

##### 验证：

#### ⑪ CE1-PE1之间链路开，CE1设备仍可以学习到spoke业务网段。配置保障有最好的扩展性（6分）

##### 解法：

##### 验证：

#### ⑫ 在拓扑正常情况下，要求CE1、CE2的EBGP优先级修改为120.访问spoke网段时，不从本AS内绕行（1分）

##### 解法：

##### 验证：

#### ⑬ 在PE3、PE4上修改BGP local-preference属性，实现CE3、CE4访问非直接的10.3.X.0/24网段时，若X为奇数，PE3、PE4优选的下一跳为PE1；若X为偶数，PE3、PE4优选的下一跳为PE2，不用考虑来回路径是否一致（3分）

##### 解法：

##### 验证：

## 四、future（17分）

### 4.1 HA
