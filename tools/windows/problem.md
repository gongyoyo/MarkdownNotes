# Windows

## 将快捷方式固定到开始菜单

将自己的快捷方式拖到``C:\ProgramData\Microsoft\Windows\Start Menu\Programs``


## 配置无线网卡和有线网卡同时上内外网
[参考CSDN博客](https://blog.csdn.net/qq_33819574/article/details/81026539)

使用CMD:

- route print
    - 输出当前的路由配置
    - 网络目标中的0.0.0.0表示所有的访问都使用这个网卡
- route delete 0.0.0.0 mask 0.0.0.0
    + 删除所有访问0.0.0.0的路由配置
- route add <font color="red">0.0.0.0</font> mask 0.0.0.0 <font color="red">192.168.43.1</font>
    + 添加无线路由访问外网
    + 第一个0.0.0.0是网络目标，第二个0.0.0.0是网络掩码，第三个192.168.43.1是wifi网关
- route add 192.168.101.1 mask 255.255.255.0 192.168.101.1 
    + 添加有线路由访问内网
    + 第一个192.168.101.1是网络目标，也就是你要访问的内网的网关，第二个255.255.255.0是网络掩码，第三个192.168.101.1是你的有线网络网关