# 综合实验
## 目录
- 实验背景
- 实验代码

---

## 实验背景
你是一个公司的网络管理员，公司的技术部(15台)、财务部门(4台)分属不同的VLAN（分别用一台pc作为部门主机代表），都接入到一个两层交换机上，两层交换机通过冗余方式汇聚到一个三层交换机上，三层交换机连接到一个对外的路由器上，这个路由器使用串行口连接到中国电信的一台路由器上且ip地址为202.120.120.1/29，在企业内部所有计算机都能互相访问，除财务部门以外且都能访问外部主机PC3（中国电信的一台主机，ip地址为200.20.20.20/24）。（拓扑图中企业内部地址都来源于192.168.x.0（x=(批号+批号)*10+组号如第2批第五小组x就等于45）网段且以最节约地址的方式且子网地址连续做ip地址规划，企业内部的pc都自动获得ip地址，企业内部pc都通过合法地址202.120.120.2/29接入到外网）。

---

## 实验代码
1. 
- 配置双层交换机
    switch#configure terminal
    switch(config)#vlan 10
    switch(config-vlan)#exit
    switch(config)#vlan 20
    switch(config-vlan)#exit
    switch(config)#interface fastethernet0/3
    switch(config-if)#switchport access vlan 10
    switch(config-if)#interface fastethernet0/4
    switch(config-if)#switchport access vlan 20
    switch(config-if)#exit
    switch(config)#interface aggregateport 1//配置聚合端口
    switch(config-if)#switchport mode trunk
    switch(config-if)#exit
    switch(config)#interface range fastethernet 0/1-2
    switch(config-if-range)#port-group 1
- 测试双层交换机配置
    switch(config-if-range)#end
    switch#show running

2. 
- 配置三层交换机
    switch#configure terminal
    switch(config)#vlan 10
    switch(config-vlan)#exit
    switch(config)#vlan 20
    switch(config-vlan)#exit
    switch(config)#vlan 50
    switch(config-vlan)#exit
    switch(config)#interface fastethernet0/3
    switch(config-if)#switchport access vlan 50
    switch(config-if)#exit

    switch(config)#interface aggregateport 1//配置聚合端口
    switch(config-if)#switchport mode trunk
    switch(config-if)#exit
    switch(config)#interface range fastethernet 0/1-2
    switch(config-if-range)#port-group 1
    switch(config-if-range)#exit
    switch(config)#interface vlan50//创建VLAN虚接口，并配置IP
    switch(config-if)#ip address 192.168.82.41 255.255.255.252
    switch(config-if)#no shutdown
    switch(config-if)#exit
    switch(config)#interface vlan20
    switch(config-if)#ip address 192.168.82.33 255.255.255.248
    switch(config-if)#no shutdown
    switch(config-if)#exit
    switch(config)#interface vlan10
    switch(config-if)#ip address 192.168.82.2 255.255.255.224
    switch(config-if)#no shutdown
    switch(config-if)#exit

    switch(config)#router ospf//配置ospf
    switch(config-router)#network 192.168.82.0 255.255.255.224 area 0
    switch(config-router)#network 192.168.82.32 255.255.255.248 area 0
    switch(config-router)#network 192.168.82.40 255.255.255.252 area 0
    switch(config-router)#end

    switch(config)#service dhcp//配置dhcp
    switch(config)#ip helper-address 192.168.82.42//指定DHCP服务器地址

    ip route 0.0.0.0 0.0.0.0 192.168.82.42
- 测试三层交换机配置
    switch(config)#end
    switch#show running

3. 
- 配置内网路由器 (ospf、dhcp server、访问控制列表)
    Router1#configure terminal
    Router1(config)#interface fastethernet 1/0
    Router1(config-if)#ip address 192.168.82.42 255.255.255.252
    Router1(config)#no shutdown
    Router1(config)#interface serial 1/2
    Router1(config-if)#ip address 202.120.120.1 255.255.255.248
    Router1(config-if)#clock rate 64000 
    Router1(config)#no shutdown

    Router1(config)#access-list 2 deny 192.168.82.32 0.0.0.7//配置访问控制列表
    Router1(config)#access-list 2 permit any//配置访问控制列表
    Router1(config)#interface fastethernet 1/0
    Router1(config-if)#ip access-group 2 in
    Router1(config-if)#end

    ip route 0.0.0.0 0.0.0.0 202.120.120.3

    Router1(config)#router osp//配置ospf
    Router1(config-router)#network 192.168.82.42 0.0.0.0 area 0
    Router1(config-router)#network 202.120.120.0  0.0.0.7 area 0
    Router1(config-router)#end

    Router1(config)#service dhcp//配置dhcp vlan10
    Router1(config)#ip dhcp pool vlan10
    Router1(dhcp-config)#network 192.168.82.0 255.255.255.224
    Router1(dhcp-config)#default-router 192.168.82.2
    Router1(dhcp-config)##dns-server 8.8.8.8 8.8.2.2
    Router1(dhcp-config)##lease 0 1
    Router1(dhcp-config)#exit

    Router1(config)#service dhcp//配置dhcp vlan 20
    Router1(config)#ip dhcp pool vlan20
    Router1(dhcp-config)#network 192.168.82.32 255.255.255.248
    Router1(dhcp-config)#default-router 192.168.82.33
    Router1(dhcp-config)##dns-server 8.8.8.8 8.8.2.2
    Router1(dhcp-config)##lease 0 1
    Router1(dhcp-config)#exit

    Router1(config)#interface serial 1/2//配置NAT
    Router1(config-if)#ip nat outside
    Router1(config)#interface fastethernet 1/0//配置NAT
    Router1(config-if)#ip nat inside
    Router1(config-if)#exit

    Router1(config)#ip nat pool onlyone 202.120.120.2 202.120.120.2 netmask 255.255.255.248 //配置NAT
    Router1(config)#access-list 1 permit 192.168.82.0 0.0.0.255
    Router1(config)#ip nat inside source list 1 pool onlyone overload

- 测试内网路由器 
    Router1(config)#end
    Router1#show running

4. 
- 配置外网路由器 (ip和ospf)
    Router2#configure terminal
    Router2(config)#interface serial 1/2
    Router2(config-if)#ip address 202.120.120.3 255.255.255.248
    Router2(config)#no shutdown
    Router2(config)#interface fastethernet 1/0
    Router2(config-if)#ip address1 255.255.255.0
    Router2(config)#no shutdown

    Router2(config)#ip route 192.168.90.1 255.255.255.224 202.120.120.1 //配置静态路由 
    Router2(config)#ip route 192.168.90.34 255.255.255.248 202.120.120.1 //配置静态路由

- 测试外网路由器 
    Router2(config)#end
    Router2#show running

---