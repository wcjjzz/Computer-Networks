- **插槽**：主板上**物理可见的连接器**
- **接口**：**数据传输的协议或规范**，定义了电压、时序、信号含义等逻辑层面的规则
- **一种插槽可以兼容多种接口**：
	- 通过多协议兼容
	- 不同协议的接口，课可能有统一的物理形态


## 设备类型库
- 不同型号的路由器，性能、接口类型、数量，支持的协议，不一样
  <img src="https://raw.githubusercontent.com/wcjjzz/Computer-Networks/main/attachments/Pasted%20image%2020250313194207.png">


## 命令
```bash
清空所有启动配置，重启交换机
erase startup-config
Switch# reload

清空fa0/1端口的所有配置
Switch(config)# default interface fa0/1
```

