## 一、实验目的

- 知道VLAN有什么作用；

-  能在单台交换机、多台交换机上配置VLAN；

- 知道VTP协议的作用，服务器模式和客户模式的区别；

- 了解如何利用VTP协议实现VLAN的管理。


---
## 二、预备问题
### 交换机的3种端口
- **Access端口**：
	- 用于连接终端设备（如电脑、打印机等）。
	- ==只能属于一个VLAN==，并且会将流量标记为该VLAN的标识。

- **Trunk端口**：
	- 用于连接交换机与交换机之间，或者连接交换机与路由器、无线控制器等设备。
	- ==可以承载多个VLAN的流量==，并且会对不同VLAN的数据帧进行标记（VLAN Tagging）。
	    
- **Dynamic端口**==【初始默认】==
	- 指交换机端口能够根据连接的设备，自动决定是作为Access端口还是Trunk端口。

---
### VTP 协议的作用
- **简化 VLAN 管理**：通过 Trunk 链路自动传播 VLAN 信息（如 VLAN 的创建、删除、名称修改等），避免在每台交换机上手动重复配置，也保证了同一 VTP 域内的交换机 VLAN 配置统一

- **实现方式**：将交换机划分为服务器模式交换机、客户模式交换机

| **特性**        | **服务器模式（Server）**                        | **客户模式（Client）**                           |
| ------------- | ---------------------------------------- | ------------------------------------------ |
| **VLAN 配置修改** | 允许创建、修改、删除 VLAN，<br>==其配置会转发到整个 VTP 域==。 | 不允许本地创建、修改或删除 VLAN，<br>==只能被动接收服务器的配置更新==。 |
| **配置存储**      | 将 VLAN 配置存储到 NVRAM（非易失性存储器），重启后配置保留。     | 不将 VLAN 配置存储到 NVRAM，重启后配置丢失（依赖服务器同步）。      |
| **默认模式**      | 交换机的默认 VTP 模式为服务器模式。                     | 需手动配置为客户模式。                                |

---
### VTP实现VLAN管理
#### 2种模式的交换机
- **作用**：通过 Trunk 链路自动传播 VLAN 信息（如 VLAN 的创建、删除、名称修改等），避免在每台交换机上手动重复配置，也保证了同一 VTP 域内的交换机 VLAN 配置统一

- **实现方式**：将交换机划分为服务器模式交换机、客户模式交换机

| **特性**        | **服务器模式（Server）**                        | **客户模式（Client）**                           |
| ------------- | ---------------------------------------- | ------------------------------------------ |
| **VLAN 配置修改** | 允许创建、修改、删除 VLAN，<br>==其配置会转发到整个 VTP 域==。 | 不允许本地创建、修改或删除 VLAN，<br>==只能被动接收服务器的配置更新==。 |
| **配置存储**      | 将 VLAN 配置存储到 NVRAM（非易失性存储器），重启后配置保留。     | 不将 VLAN 配置存储到 NVRAM，重启后配置丢失（依赖服务器同步）。      |
| **默认模式**      | 交换机的默认 VTP 模式为服务器模式。                     | 需手动配置为客户模式。                                |

---
#### VLAN数据库
- **VLAN数据库文件**：针对一台交换机，VLAN／VTP 相关信息都会即时写进文件 `vlan.dat`
- **整个VTP的信息**：当前的 VTP 版本、域名、运行模式、Configuration Revision（修订号）、最大/已用 VLAN 数、MD5 摘要
	- **VTP 通告**：服务器没做一次VLAN改动，就会向同一个VTP域内所有交换机发出VTP通告，同步更新
	- **修订号**：每次修订都有记录，修订一次，修订号加一
<img src="https://raw.githubusercontent.com/wcjjzz/Computer-Networks/main/attachments/Pasted%20image%2020250410212320.png">
- **每个VLAN信息**：1‑1005 号 VLAN 的 ID、名称、状态、下属的端口号
<img src="https://raw.githubusercontent.com/wcjjzz/Computer-Networks/main/attachments/Pasted%20image%2020250410212301.png">

---
#### VTP域名、域密码
- 将一台交换机加入一个VTP域，并参与 VLAN 数据库同步，需要满足2个条件
	- 声明当前交换机所属的域
	- 如果域内的任何一台交换机配置了密码，则需要通过密码验证
		- 以服务器最新设定的密码为正确密码（修订号最大那次）
		- 如果客户交换机设置的密码不是正确密码，会报错

---
#### Native VLAN
- **未打标签帧**：当交换机从未配置所属VLAN的端口获得一个发往其他交换机下主机的帧时，这个帧是未打标签（Untagged）
- **默认所属VLAN**：此时，Trunk端口组帧时，会默认802.1Q帧的VLAN = 当前交换机的Native VLAN
- **Native VLAN 的默认 VID 是 1**
- 两个相连的交换机，必须通过两个Trunk端口，最好两台交换机的Native VLAN也一样
	- 否则会警告：native VLAN mismatc


---
## 三、实验步骤

### 网络拓扑图
- 3台交换机：000（服务器）、111（客户）、222（客户）
- 000、222的`f0/1`，111的`f0/1`、`f0/2`设置为Trunk接口
- 000、222的`f0/2`、`f0/3`的VLAN设置为2、3
- 4台PC机：
  PC0（192.168.1.1）、PC1（192.168.2.1）连000的`f0/2`、`f0/3`
  PC2（192.168.1.2）、PC3（192.168.2.2）连222的`f0/2`、`f0/3`
<img src="https://raw.githubusercontent.com/wcjjzz/Computer-Networks/main/attachments/Pasted%20image%2020250410214900.png">

### 配置主机名、Trunk端口
- 修改主机名
```
Switch(config)# hostname 000 
```
- 服务器交换机
```
111(config)# interface range Fa0/1 - 2    

111(config-if-range)# description TRUNK_to_SW111/SW222

111(config-if-range)# switchport mode trunk             

111(config-if-range)# switchport trunk native vlan 1    

111(config-if-range)# no shutdown
```
- 客户交换机
```
000(config)# interface range Fa0/1 - 2    

000(config-if-range)# description TRUNK_to_SW111/SW222

000(config-if-range)# switchport mode trunk             

000(config-if-range)# switchport trunk native vlan 1    

000(config-if-range)# no shutdown
```

### 配置VTP域、基于接口创建VLAN
- 在每台交换机配置所属VTP 域名、域密码
```
000(config)#vtp domain aa

Changing VTP domain name from NULL to aa

000(config)#vtp mode client

Setting device to VTP CLIENT mode.

000(config)#vtp password 123

Setting device VLAN database password to 123
```


### 配置端口的VLAN
- 在VTP Server（000）上创建 VLAN
```
111(config-if-range)#vlan 2

111(config-vlan)#name Sales

111(config-vlan)#vlan 3

111(config-vlan)# name Tech
```

- 在客户交换机设置端口所属的VLAN
```
000(config-vlan)#interface f0/2

000(config-if)#switchport mode access

000(config-if)#switchport access vlan 2

000(config-if)#no shutdown

000(config-if)#interface f0/3

000(config-if)#switchport mode access

000(config-if)#switchport access vlan 3

000(config-if)#no shutdown
```

### ping同、不同VLAN跨PC机
<img src="https://raw.githubusercontent.com/wcjjzz/Computer-Networks/main/attachments/Pasted%20image%2020250410225357.png">


---
## 四、实验总结与心得体会

- **协议原理与动手实践的统一**  
    通过亲手配置 VTP 域并观察修订号变化，切实体会到 VTP「一次创建，多处同步」的优势。尤其是在 Server 端执行 `vlan 2`、`vlan 3` 后，Client 侧 `show vlan brief` 立即出现对应 VLAN，生动地把教材中的“配置自动下发”变成了肉眼可见的结果，也加深了对 VTP 修订号、配置摘要校验（MD5）等概念的理解。
    
- **参数一致性的重要性**
    - **域名 / 密码 / 版本三要素缺一不可**：实验初期曾故意把 SW222 的 VTP 域名拼错，结果导致其 VLAN 列表始终停留在默认状态，印证了 VTP 只在同域之间交换信息的机制。
    - **Native VLAN 对齐**：若两端 trunk 的 Native VLAN 不同，CDP 会立即抛出 `%CDP-4-NATIVE_VLAN_MISMATCH` 告警；这一“红灯”提醒我，细节配置往往决定网络能否稳定运行。
        
- **拓扑设计思维的提升**  
    本实验采用星形拓扑，让 SW000 同时承担 VTP Server 与汇聚交换机角色，节约了链路数量，却要求其拥有至少两条 trunk。对比链形拓扑可知：**物理连线方案会直接影响 trunk 口数量与冗余策略**，网络设计需要在成本、可扩展性与可靠性之间权衡。
    
- **故障定位与验证方法**
    - 当 VTP 同步失败时，先用 `show vtp status` 查看修订号，再用 `show interfaces trunk` 确认链路类型，最后检查域名/密码——形成了“自上而下”的排错流程。
    - 对 trunk 端口做 `switchport trunk allowed vlan` 后，立即用 `show int trunk` 验证允许列表，确保数据面与控制面配置一致。**及时验证** 能在实验室里省去大量“为什么不通”的时间。
        